---
title: Exporting data from Unified Logging System in Swift 
layout: post
image: /public/logging.png
category: Architecture
---

We discussed building a proper logging system instead of using the print function in the previous post. Apple provides us a framework to utilize its logging system backed by on-disk persistence. This week we will talk about exporting logs from the user devices by leveraging the power of the Unified Logging System.

> To learn about basics of the Unified Logging System, take a look at my ["Logging in Swift"](/2022/04/06/logging-in-swift/) post.

The Unified Logging System comes with the *OSLogStore* type letting us fetch and filter logs saved in our app. Let's build the *LogStore* type that we can use in our settings screen to create a feature allowing our users to export and share logs with the app maintainer.

```swift
import OSLog
import Foundation

@MainActor final class LogStore: ObservableObject {
    private static let logger = Logger(
        subsystem: Bundle.main.bundleIdentifier!,
        category: String(describing: LogStore.self)
    )

    @Published private(set) var entries: [String] = []

    func export() {
        do {
            let store = try OSLogStore(scope: .currentProcessIdentifier)
            let position = store.position(timeIntervalSinceLatestBoot: 1)
            entries = try store
                .getEntries(at: position)
                .compactMap { $0 as? OSLogEntryLog }
                .filter { $0.subsystem == Bundle.main.bundleIdentifier! }
                .map { "[\($0.date.formatted())] [\($0.category)] \($0.composedMessage)" }
        } catch {
            Self.logger.warning("\(error.localizedDescription, privacy: .public)")
        }
    }
}
```

As you can see in the example above, we created an instance of the *OSLogStore* type scoped to the current process. We use the *position* function to build an object representing a date from which we want to export logs. In the recent example, we use the *timeIntervalSinceLatestBoot* parameter to fetch entries since the last boot. 

Then we use the *getEntries* function to fetch the log entries from the defined position. In the example above, we filter entries to include only the needed *subsystem*. We also use different fields of log entries to build formatted strings. Not let's see how we can use our *LogStore* type.

```swift
struct SettingsView: View {
    @ObservedObject var logs: LogStore
    @State private var exportShown = false
    
    var body: some View {
        Form {
            Section(header: Text("debug")) {
                Button("exportLogs") {
                    logs.export()
                    exportShown = true
                }
                .sheet(isPresented: $exportShown) {
                    ShareView(items: [logs.entries.joined(separator: "\n")])
                }
            }
        }
    }    
}

struct ShareView: UIViewControllerRepresentable {
    typealias UIViewControllerType = UIActivityViewController

    let items: [Any]

    func makeUIViewController(context: Context) -> UIActivityViewController {
        UIActivityViewController(activityItems: items, applicationActivities: nil)
    }

    func updateUIViewController(_ uiViewController: UIActivityViewController, context: Context) {
    }
}
```

I have the debug section in the settings screen that provides a button to export logged data. I also wrap the *UIActivityViewController* with the *UIViewControllerRepresentable* to give a nice way to share logs with the app maintainer.

The *OSLogStore* type provides us another overload of the *position* function, allowing us to specify a particular date to fetch all the entries after it. For example, you might need to fetch the log entries for the last 24 hours.

```swift
import OSLog
import Foundation

@MainActor final class LogStore: ObservableObject {
    private static let logger = Logger(
        subsystem: Bundle.main.bundleIdentifier!,
        category: String(describing: LogStore.self)
    )

    @Published private(set) var entries: [String] = []

    func export() {
        do {
            let store = try OSLogStore(scope: .currentProcessIdentifier)
            let date = Date.now.addingTimeInterval(-24 * 3600)
            let position = store.position(date: date)
            
            entries = try store
                .getEntries(at: position)
                .compactMap { $0 as? OSLogEntryLog }
                .filter { $0.subsystem == Bundle.main.bundleIdentifier! }
                .map { "[\($0.date.formatted())] [\($0.category)] \($0.composedMessage)" }
        } catch {
            Self.logger.warning("\(error.localizedDescription, privacy: .public)")
        }
    }
}
```

Today we learned how to use the Unified Logging System to export the valuable information logged by our apps. We can't always catch all the things on our testing devices. That's why a proper logging system is essential for any app. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
