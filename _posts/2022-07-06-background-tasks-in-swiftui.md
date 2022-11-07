---
title: Background tasks in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/wwdc22.jpg
---

One of the exciting frameworks released along with iOS 13 was the BackgroundTasks framework. It allows you to schedule work intelligently in the background. Finally, we can handle background tasks using the SwiftUI app lifecycle. This week we will learn how to schedule and handle background tasks in SwiftUI.

{% include friends.html %}

#### Basics
First, you must add background mode to your Xcode project on the capabilities page. You should choose background fetch in the list of available background modes to enable your app to make network requests in the background. Let's see how we can schedule a background task.

```swift
import BackgroundTasks
    
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "myapprefresh")
    try? BGTaskScheduler.shared.submit(request)
}
```

As you can see in the example above, we have the *scheduleAppRefresh* function using the shared instance of *BGTaskScheduler* to schedule an app refresh task. An app refresh task should have a unique identifier. We must also define the list of all the identifiers in the Info.plist file within the "Permitted background task scheduler identifiers" key. Now we can call the *scheduleAppRefresh* function within the SwiftUI app lifecycle.

```swift
@main
struct MyApp: App {
    @Environment(\.scenePhase) private var phase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: phase) { newPhase in
            switch newPhase {
            case .background: scheduleAppRefresh()
            default: break
            }
        }
    }
}    
```

Here we schedule a background task as soon as the user leaves our app. We can tune the timing of the task by using the *earliestBeginDate* property on the instance of the *BGAppRefreshTaskRequest* type. We tell the system to run the task only after the deadline by providing the earliest beginning date.

```swift
import BackgroundTasks
    
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "myapprefresh")
    request.earliestBeginDate = .now.addingTimeInterval(24 * 3600)
    try? BGTaskScheduler.shared.submit(request)
}
```

#### App refresh tasks
To handle app refresh tasks in SwiftUI, we have the brand new *backgroundTask* modifier that we can attach to any scene.

```swift
@main
struct MyApp: App {
    @Environment(\.scenePhase) private var phase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: phase) { newPhase in
            // ..
        }
        .backgroundTask(.appRefresh("myapprefresh")) {
            let request = URLRequest(url: URL(string: "your_backend")!)
            guard let data = try? await URLSession.shared.data(for: request).0 else {
                return
            }
            
            let decoder = JSONDecoder()
            guard let products = try? decoder.decode([Product].self, from: data) else {
                return
            }
         
            if !products.isEmpty && !Task.isCancelled {
                await notifyUser(for: products)
            }
        }
    }
}    
```

> To learn more about scenes in SwiftUI, take a look at my ["Managing scenes in SwiftUI"](https://swiftwithmajid.com/2020/08/26/managing-scenes-in-swiftui/) post. 

As you can see in the example above, we the *backgroundTask* modifier to register an app refresh handler for a particular identifier. SwiftUI relies on the new Swift Concurrency feature and allows us to build complex async jobs using the async/await syntax. It also fully supports cooperative cancelation, and you can quickly check if your task is out of background runtime using the static *isCancelled* property on the Task type.

#### URLSession tasks
Whenever you need a long-running task like downloading a file, you should use an instance of *URLSession* with background configuration. In this case, you can handle task suspension and continue your work later.

```swift
func handleFileDownload() async {
    let url = URL(string: "your_backend")!
    
    let config = URLSessionConfiguration.background(withIdentifier: "myurlsession")
    config.sessionSendsLaunchEvents = true
    let session = URLSession(configuration: config)
    
    let data = await withTaskCancellationHandler {
        try? await session.data(for: URLRequest(url: url))
    } onCancel: {
        let task = session.downloadTask(with: URLRequest(url: url))
        task.resume()
    }

    if let data {
        await notifyUser()
    }
}
```

In the example above, we create an instance of *URLSession* with the background configuration. We also use the *withTaskCancellationHandler* function allowing us to handle task cancellation. Whenever our task is canceled, we create a download task that will be saved by the system and run later.

```swift
@main
struct MyApp: App {
    @Environment(\.scenePhase) private var phase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: phase) { newPhase in
            switch newPhase {
            case .background: scheduleAppRefresh()
            default: break
            }
        }
        .backgroundTask(.appRefresh("myapprefresh")) {
            await handleFileDownload()
        }
        .backgroundTask(.urlSession("myurlsession")) {
            // Handle your background url session events here
        }
    }
}
```

We can handle events from the *URLSession* with background configuration by using the *backgroundTask* modifier with the particular identifier.

#### Debugging
The only way of debugging background tasks is to keep your phone connected to Xcode debugger, but we don’t know when iOS will decide to run our jobs because it uses some hidden logic for that. Luckily, Apple provides two private functions, which we can use in the debugger to start and expire background tasks. Please remember that you can use it only during development, don’t include them in release.

For starting background tasks, pause your app and run in the debugger this code:

```
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"TASK_IDENTIFIER"]
```

To force early termination use

```
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateExpirationForTaskWithIdentifier:@"TASK_IDENTIFIER"]
```

Don't forget to replace TASK_IDENTIFIER with the real identifier.

#### Conclusion
Today we learned how to use the BackgroundTasks framework in SwiftUI by leveraging the power of the new Swift Concurrency feature. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
