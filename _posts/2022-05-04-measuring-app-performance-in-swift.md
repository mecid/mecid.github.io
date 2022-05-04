---
title: Measuring app performance in Swift
layout: post
image: /public/signpost.png
category: Logging
---

The Unified Logging System is a great way to build a proper logging system allowing you to understand different exceptional cases happening in your app. But it is not limited only to logging. It also provides a way to measure various events in your app. This week, we will learn how to use the Unified Logging System to measure app performance.

#### Measuring app events
The Unified Logging System provides us with the Signpost API, which is a way to measure various time intervals in your app. Let's take a look at how we can use it in a small example.

```swift
@MainActor final class TimerStore: ObservableObject {
    private static let logger = Logger(
        subsystem: "com.aaplab.fastbot",
        category: "TimerStore"
    )
    private static let signposter = OSSignposter(logger: logger)
    
    @Published private(set) var accountStatus: CKAccountStatus = .couldNotDetermine
    @Published private(set) var recentFasting: Fasting? = nil
    
    private let service = CloudKitService()
    
    func fetchRecent() async {
        let signpostID = Self.signposter.makeSignpostID()

        let interval = Self.signposter.beginInterval("Fetching recent", id: signpostID)
        do {
            accountStatus = try await service.checkAccountStatus()
            recentFasting = try await service.fetchAllFastingRecords().last
            Self.signposter.endInterval("Fetching recent", interval)
        } catch {
            Self.signposter.endInterval("Fetching recent", interval)
        }
    }
}
```

First, we need to import the *OSLog* module that contains the Unified Logging System API. Then we create an instance of the *OSSignposter* type that we will use to track event intervals. We use the *makeSignpostID* function on the *OSSignposter* type to create a unique event identifier.

Now we can use the identifier to start monitoring an event with a particular message using the *beginInterval* function. This function returns a state of the interval that we will use to associate starting and finishing points of an event. As the last step, we call the *endInterval* function by passing a message and the interval state.

> Remember that the message you use while beginning and ending intervals should be the same.

Another thing we might need is attaching metadata to a signpost interval. For example, we can bind a localized error description whenever interval ends with an error.

```swift
func fetchRecent() async {
    let signpostID = Self.signposter.makeSignpostID()

    let interval = Self.signposter.beginInterval("Fetching recent", id: signpostID)
    do {
        accountStatus = try await service.checkAccountStatus()
        recentFasting = try await service.fetchAllFastingRecords().last
        Self.signposter.endInterval("Fetching recent", interval)
    } catch {
        Self.signposter.endInterval("Fetching recent", interval, "\(error.localizedDescription, privacy: .public)")
    }
}
```

We can also emit intermediate events inside a particular interval. It might be helpful to divide a multi-step event into timepieces to understand which part of the event is slowing the app.

```swift
func fetchRecent() async {
    let signpostID = Self.signposter.makeSignpostID()

    let interval = Self.signposter.beginInterval("Fetching recent", id: signpostID)
    do {
        accountStatus = try await service.checkAccountStatus()
        Self.signposter.emitEvent("Account status fetched", id: signpostID)
        recentFasting = try await service.fetchAllFastingRecords().last
        Self.signposter.endInterval("Fetching recent", interval)
    } catch {
        Self.signposter.endInterval("Fetching recent", interval, "\(error.localizedDescription, privacy: .public)")
    }
}
```

In the example above, we use the *emitEvent* function on the *OSSignposter* type to emit additional events connected to a particular signpost identifier.

#### Collecting performance data
OK, we learned how to measure app events using the *Signposter* type. Now it is time to learn how to read that data to analyze our app performance. There are two ways of reading the data written by the *Signposter* type. 

First, we can use the Instruments app to visualize all the performance data nicely. The second is a programmatic way of exporting performance data from the user devices using the *OSLogStore* type.

Let's start with the simplest one. While debugging your app via Xcode, you can build the app for profiling by pressing CMD + I. In the opened Instruments app, choose the Logging template. It contains both logs and signposts. Now run the app by pressing the record button and start interacting and producing events in your app.

![instruments-logging-template](/public/signpost.png)

Instruments app is a great way to profile your app locally, but sometimes we need data from the user devices. In this case, we can export the signpost data using the *OSLogStore* type.

```swift
import OSLog
import Foundation

@MainActor final class LogStore: ObservableObject {
    private static let logger = Logger(
        subsystem: Bundle.main.bundleIdentifier!,
        category: String(describing: LogStore.self)
    )

    @Published private(set) var entries: [String] = []

    func exportPerformanceData() {
        do {
            let store = try OSLogStore(scope: .currentProcessIdentifier)
            let position = store.position(timeIntervalSinceLatestBoot: 1)
            entries = try store
                .getEntries(at: position)
                .compactMap { $0 as? OSLogEntrySignpost }
                .filter { $0.subsystem == Bundle.main.bundleIdentifier! }
                .map { "[\($0.date.formatted())] [\($0.category)] [\($0.signpostType)] \($0.signpostName)" }
        } catch {
            Self.logger.warning("\(error.localizedDescription, privacy: .public)")
        }
    }
}
```

#### Conclusion
Today we learned how to use the Unified Logging System to measure and collect performance data of our apps. Understanding the performance of particular events is crucial for building a great user experience. Fortunately, we can quickly achieve that by using the Unified Logging System.
