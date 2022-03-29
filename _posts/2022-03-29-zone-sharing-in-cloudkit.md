---
title: Zone sharing in CloudKit
layout: post
category: CloudKit
image: /public/cloudkit.png
---

Last week we talked about the basics of CloudKit. We learned how to save and fetch data from the storage in the cloud and how to sync the data between devices. This week I want to cover the only reason why I have chosen CloudKit instead of Firebase, and it is data sharing between users.

> If you are not familiar with CloudKit, please take a look at my ["Getting started with CloudKit"](/2022/03/22/getting-started-with-cloudkit/) post.

CloudKit provides you ready to use data sharing API that allows you to implement collaborative features of your app without much effort. There are two ways to share data via CloudKit: record sharing and zone sharing. In this post, we will talk about zone sharing.

A zone is a defined bucket inside the private database of the current user. You can create and assign a zone to any record in the database. You share all the records in the zone by sharing the zone itself. For example, in the todo app, you can create a zone with the name of the todo list and share it with your family.

First of all, we need to create and save a zone in the private database of the current user. Then we have to assign it to a particular record and save or update it in the private database.

```swift
private enum FastingRecordKeys {
    static let type = "Fasting"
    static let startDate = "startDate"
    static let endDate = "endDate"
    static let goal = "goal"
}

private enum SharedZone {
    static let name = "SharedZone"
    static let ID = CKRecordZone.ID(
        zoneName: name,
        ownerName: CKCurrentUserDefaultName
    )
}

extension Fasting {
    var asRecord: CKRecord {
        let record = CKRecord(
            recordType: FastingRecordKeys.type,
            recordID: .init(zoneID: SharedZone.ID)
        )
        record[FastingRecordKeys.goal] = goal.rawValue
        record[FastingRecordKeys.startDate] = startDate
        record[FastingRecordKeys.endDate] = endDate
        return record
    }
    
    init?(from record: CKRecord) {
        guard
            let startDate = record[FastingRecordKeys.startDate] as? Date,
            let endDate = record[FastingRecordKeys.endDate] as? Date,
            let goalRawValue = record[FastingRecordKeys.goal] as? String,
            let goal = Fasting.Goal(rawValue: goalRawValue)
        else { return nil }
        
        self = .init(
            startDate: startDate,
            endDate: endDate,
            goal: goal,
            name: record.recordID.recordName
        )
    }
}

final class CloudKitService {
    static let container = CKContainer(
        identifier: "iCloud.com.aaplab.fastbot"
    )
    
    func save(_ fasting: Fasting) async throws {
        _ = try await Self.container.privateCloudDatabase.modifyRecordZones(
            saving: [CKRecordZone(zoneName: SharedZone.name)],
            deleting: []
        )
        _ = try await Self.container.privateCloudDatabase.modifyRecords(
            saving: [fasting.asRecord],
            deleting: []
        )
    }
}
```

Now we have a zone saved in the private database and associated records. But to start sharing, we should create an instance of *CKShare* type and save it into the private database.

```swift
extension CloudKitService {
    func shareFastingRecords() async throws -> CKShare {
        _ = try await Self.container.privateCloudDatabase.modifyRecordZones(
            saving: [CKRecordZone(zoneName: SharedZone.name)],
            deleting: []
        )

        let share = CKShare(recordZoneID: SharedZone.ID)
        share.publicPermission = .readOnly
        let result = try await Self.container.privateCloudDatabase.save(share)
        return result as! CKShare
    }
}
```

As you can see in the example above, we create a new *CKShare* object and set the public permission to *readOnly*. The default value is *none* which blocks anyone from accessing your data without your approval.

Remember that you can have only one instance of *CKShare* per zone in the database. You need to check if you already have one and fetch if it is available. The next step is to present an instance of *UICloudSharingController* with the created *CKShare* object.

```swift
struct CloudKitShareView: UIViewControllerRepresentable {
    let share: CKShare

    func makeUIViewController(context: Context) -> UICloudSharingController {
        let sharingController = UICloudSharingController(
            share: share,
            container: CloudKitService.container
        )
        
        sharingController.availablePermissions = [.allowReadOnly, .allowPrivate]
        sharingController.modalPresentationStyle = .formSheet
        return sharingController
    }

    func updateUIViewController(
        _ uiViewController: UIViewControllerType,
        context: Context
    ) { }
}
```

*UICloudSharingController* provides you with all the needed functionality to add and manage participants of the share. You can also configure available options of the *UICloudSharingController* instance by setting the value of the *availablePermissions* property to *allowReadOnly, allowPrivate, allowReadWrite, allowPublic*. Now let's talk about how we should implement share accepting functionality.

Before writing any code, we should add the *CKSharingSupported* boolean key with the value *YES* to the Info.plist. Whenever a user receives and opens a link shared via *UICloudSharingController*, the *userDidAcceptCloudKitShareWith* delegate method will be called by the system. Here we can call the accept method to approve the sharing.

```swift
extension CloudKitService {
    func accept(_ metadata: CKShare.Metadata) async throws {
        try await Self.container.accept(metadata)
    }
}

private final class SceneDelegate: NSObject, UIWindowSceneDelegate {
    private let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: "SceneDelegate"
    )
    
    private let cloudKitService = CloudKitService()

    func windowScene(
        _ windowScene: UIWindowScene,
        userDidAcceptCloudKitShareWith cloudKitShareMetadata: CKShare.Metadata
    ) {
        Task {
            do {
                try await cloudKitService.accept(cloudKitShareMetadata)
            } catch {
                logger.error("\(error.localizedDescription, privacy: .public)")
            }
        }
    }
}
```

Finally, we can use the shared database to fetch the content of shared zones.

```swift
extension CloudKitService {
    func fetchSharedFastingRecords(
        in interval: DateInterval
    ) async throws -> [Fasting] {
        let sharedZones = try await Self.container.sharedCloudDatabase.allRecordZones()
        
        return try await withThrowingTaskGroup(
            of: [Fasting].self,
            returning: [Fasting].self
        ) { group in
            for zone in sharedZones {
                group.addTask {
                    let predicate = NSPredicate(
                        format: "\(FastingRecordKeys.endDate) > %@ AND \(FastingRecordKeys.endDate) <= %@",
                        interval.start as NSDate,
                        interval.end as NSDate
                    )

                    return try await self.fetchFastingRecords(
                        with: predicate,
                        in: zone.zoneID,
                        from: Self.container.sharedCloudDatabase
                    )
                }
            }
            
            var results: [Fasting] = []
            for try await history in group {
                results.append(contentsOf: history)
            }
            
            return results
        }
    }
                    
    private func fetchFastingRecords(
        with predicate: NSPredicate,
        in zone: CKRecordZone.ID? = nil,
        from database: CKDatabase
    ) async throws -> [Fasting] {
        let query = CKQuery(recordType: FastingRecordKeys.type, predicate: predicate)
        query.sortDescriptors = [.init(key: FastingRecordKeys.endDate, ascending: true)]

        let response = try await database.records(
            matching: query,
            inZoneWith: zone,
            desiredKeys: nil,
            resultsLimit: CKQueryOperation.maximumResults
        )

        return response.matchResults
            .compactMap { try? $0.1.get() }
            .compactMap { $0.compactMap(Fasting.init) }
    }
}
```

Remember that you have to fetch all the zones from the shared database. Even when they have the same name, they have different owners. We can enumerate all the shared zones and fetch shared records in every zone.

 I think CloudKit data sharing API is one of the best things about this technology. And today we learned how to use it. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
