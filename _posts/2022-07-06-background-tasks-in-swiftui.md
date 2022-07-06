---
title: Background tasks in SwiftUI
layout: post
---
One of the exciting frameworks released along with iOS 13 was the BackgroundTasks framework. It allows you to schedule work intelligently in the background. Finally, we can handle background tasks using the SwiftUI app lifecycle. This week we will learn how to schedule and handle background tasks in SwiftUI.

#### Basics
First, you must add background mode to your Xcode project on the capabilities page. You should choose background fetch in the list of available background modes to enable your app to make network requests in the background. Let's see how we can schedule a background task.

=====================================================

As you can see in the example above, we have the scheduleAppRefresh function using the shared instance of BGTaskScheduler to schedule an app refresh task. Every app refresh task should have a unique identifier. We must also define the list of all the identifiers in the Info.plist file within the "Permitted background task scheduler identifiers" key. Now we can call the scheduleAppRefresh function within the SwiftUI app lifecycle.

=====================================================

Here we schedule a background task as soon as the user leaves our app. We can tune the timing of the task by using the earliestBeginDate property on the instance of the BGAppRefreshTaskRequest type. We tell the system to run the task only after the deadline by providing the earliest beginning date.

=====================================================

#### App refresh tasks
To handle app refresh tasks in SwiftUI, we have the brand new backgroundTask modifier that we can attach to any scene.

=====================================================

As you can see in the example above, we the backgroundTask modifier to register an app refresh handler. SwiftUI relies on the new Swift Concurrency feature and allows us to build complex async jobs using the async/await syntax. It also fully supports cooperative cancelation, and you can quickly check if your task is out of background runtime using the static isCancelled property on the Task type.

#### URLSession tasks
Whenever you need a long-running task like downloading a file, you should use an instance of URLSession with background configuration. In this case, you can handle task suspension and continue your work later.

=====================================================

In the example above, we create an instance of URLSession with the background configuration. We also use the withTaskCancellationHandler function allowing us to handle task cancellation. Whenever our task is canceled, we create a download task that will be saved by the system and run later.

=====================================================

We can handle events from the URLSession with background configuration by using the backgroundTask modifier with the particular identifier.

=====================================================

#### Conclusion
Today we learned how to use the BackgroundTasks framework in SwiftUI by leveraging the power of the new Swift Concurrency feature.
