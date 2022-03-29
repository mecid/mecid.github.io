---
title: Getting started with CloudKit
layout: post
image: /public/cloudkit.png
category: CloudKit
---

CloudKit is an easy way to store data in the cloud, sync between multiple devices, and share it between the app's users. This week we will learn how to start using CloudKit in the app to save and fetch data from the cloud and sync between multiple user devices.

#### Basics
First, to start using CloudKit in the app, we need to enable it in the project Signing and Capabilities section. Here we can create a default container for the app. A container is a space in the cloud that stores all of your saved data. You can use a container per application or a single container to share data between multiple apps.

Every container has a public, private and shared database. The public database is accessible for any user of the app. Every user of the app has a private database that lives in the personal iCloud and counts towards iCloud storage. You can use the shared database to fetch data shared with the user.

```swift
import CloudKit
import os

final class CloudKitService {
    private static let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: String(describing: CloudKitService.self)
    )

    func checkAccountStatus() async throws -> CKAccountStatus {
        try await CKContainer.default().accountStatus()
    }
}

@MainActor final class OnboardingViewModel: ObservableObject {
    private static let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: String(describing: OnboardingViewModel.self)
    )

    @Published private(set) var accountStatus: CKAccountStatus = .couldNotDetermine

    private let cloudKitService = CloudKitService()

    func fetchAccountStatus() async {
        do {
            accountStatus = try await cloudKitService.checkAccountStatus()
        } catch {
            Self.logger.error("\(error.localizedDescription, privacy: .public)")
        }
    }
}
```

Before saving or fetching data from the CloudKit, we should check if the user has logged in an Apple ID and enabled iCloud Drive in the settings.

```swift
import SwiftUI

struct OnboardingView: View {
    @StateObject private var viewModel = OnboardingViewModel()
    @State private var accountStatusAlertShown = false
    @Environment(\.dismiss) var dismiss

    var body: some View {
        Button("startUsingApp") {
            if viewModel.accountStatus != .available {
                accountStatusAlertShown = true
            } else {
                dismiss()
            }
        }
        .alert("iCloudAccountDisabled", isPresented: $accountStatusAlertShown) {
            Button("cancel", role: .cancel, action: {})
        }
        .task {
            await viewModel.fetchAccountStatus()
        }
    }
}
```

#### Saving data
We need to define a schema for record types we want to store on CloudKit. Go to the Signing and Capabilities tab on the project settings page and press the CloudKit Console button. It should open the browser with CloudKit dashboard, where you can find schema setup in the navigation menu. Press the record types button, and create a new one that we want to store and fetch.

```swift
struct Fasting: Hashable {
    var start: Date
    var end: Date
    var goal: TimeInterval
}

enum FastingRecordKeys: String {
    case type = "Fasting"
    case start
    case end
    case goal
}

extension Fasting {
    var record: CKRecord {
        let record = CKRecord(recordType: FastingRecordKeys.type.rawValue)
        record[FastingRecordKeys.goal.rawValue] = goal
        record[FastingRecordKeys.start.rawValue] = start
        record[FastingRecordKeys.end.rawValue] = end
        return record
    }
}
```

In the example above, you see the simple *Fasting* value type that I want to store on CloudKit. CloudKit provides us *CKRecord* type representing items in the CloudKit database. Usually, we need to implement a converter from/to *CKRecord* for our custom types.

```swift
extension CloudKitService {
    func save(_ record: CKRecord) async throws {
        do {
            try await CKContainer.default().privateCloudDatabase.save(record)
        } catch {
            Self.logger.error("\(error.localizedDescription, privacy: .public)")
        }
    }
}
```

And now, we can finally create a form to populate fasting record data and save it to the private database of the current user on CloudKit.

```swift
@MainActor final class NewFastingViewModel: ObservableObject {
    private static let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: String(describing: NewFastingViewModel.self)
    )

    @Published var fasting: Fasting = .init(
        start: .now,
        end: .now,
        goal: 16 * 3600
    )
    @Published private(set) var isSaving = false

    private let cloudKitService = CloudKitService()

    func save() async {
        isSaving = true

        do {
            try await cloudKitService.save(fasting.record)
        } catch {
            Self.logger.error("\(error.localizedDescription, privacy: .public)")
        }

        isSaving = false
    }
}

struct NewFastingRecord: View {
    @StateObject private var viewModel = NewFastingViewModel()
    @Environment(\.dismiss) var dismiss

    var body: some View {
        Form {
            Section {
                DatePicker("start", selection: $viewModel.fasting.start)
                DatePicker("end", selection: $viewModel.fasting.end)
            }

            Section {
                Picker("goal", selection: $viewModel.fasting.goal) {
                    ForEach([16, 18, 23], id: \.self) { hours in
                        Text(String(hours))
                            .tag(hours * 3600)
                    }
                }
            }
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button("save") {
                    Task {
                        await viewModel.save()
                        dismiss()
                    }
                }.disabled(viewModel.isSaving)
            }
        }
        .navigationTitle("newFasting")
    }
}
```

#### Fetching data
Now we can learn how to fetch data from the CloudKit. First, we should update our schema by adding indexes marking fields in our records queryable and sortable. Let's open the CloudKit dashboard and go to Schema -> Indexes. Here we should create indexes for all the fields we want to fetch or sort. 

```swift
extension Fasting {
    init?(from record: CKRecord) {
        guard
            let start = record[FastingRecordKeys.start.rawValue] as? Date,
            let end = record[FastingRecordKeys.end.rawValue] as? Date,
            let goal = record[FastingRecordKeys.goal.rawValue] as? TimeInterval
        else { return nil }
        self = .init(start: start, end: end, goal: goal)
    }
}
```

In the example above, we implemented another converter from an instance of the *CKRecord* type. Let's move forward and implement a method on the *CloudKitService* type to fetch the records in the provided date interval.

```swift
extension CloudKitService {
    func fetchFastingRecords(in interval: DateInterval) async throws -> [Fasting] {
        let predicate = NSPredicate(
            format: "\(FastingRecordKeys.start.rawValue) >= %@ AND \(FastingRecordKeys.end.rawValue) <= %@",
            interval.start as NSDate,
            interval.end as NSDate
        )

        let query = CKQuery(
            recordType: FastingRecordKeys.type.rawValue,
            predicate: predicate
        )

        query.sortDescriptors = [.init(key: FastingRecordKeys.end.rawValue, ascending: true)]

        let result = try await CKContainer.default().privateCloudDatabase.records(matching: query)
        let records = result.matchResults.compactMap { try? $0.1.get() }
        return records.compactMap(Fasting.init)
    }
}
```

And now we are ready to implement a view showing the fasting history.

```swift
@MainActor final class FastingHistoryViewModel: ObservableObject {
    private static let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: String(describing: FastingHistoryViewModel.self)
    )

    @Published var interval: DateInterval = .init(
        start: .now.addingTimeInterval(-30 * 34 * 3600),
        end: .now
    )

    @Published private(set) var history: [Fasting] = []
    @Published private(set) var isLoading = false

    private let cloudKitService = CloudKitService()

    func fetch() async {
        isLoading = true

        do {
            history = try await cloudKitService.fetchFastingRecords(in: interval)
        } catch {
            Self.logger.error("\(error.localizedDescription, privacy: .public)")
        }

        isLoading = false
    }
}

struct FastingHistoryView: View {
    @StateObject private var viewModel = FastingHistoryViewModel()

    var body: some View {
        List(viewModel.history, id: \.self) { fasting in
            VStack(alignment: .leading) {
                Text(fasting.start, style: .time)
                Text(fasting.end, style: .time)
            }
        }
        .redacted(reason: viewModel.isLoading ? .placeholder : [])
        .refreshable {
            await viewModel.fetch()
        }
        .task {
            await viewModel.fetch()
        }
    }
}
```

#### Conclusion
CloudKit provides us with development and production environments. While developing an app and running it in the debug mode, you automatically use the development environment. Before publishing the app on TestFlight or App Store, you should deploy schema to the production environment in the CloudKit dashboard.

This week we learned the basics of storing and fetching data in the CloudKit. Now you know how to sync the data between the user devices. Next week we will learn how to implement data sharing between your app users via CloudKit. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
