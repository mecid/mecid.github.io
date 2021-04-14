---
title: Managing scenes in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: View Composition
---

This week we will continue the series of posts about the app and scene lifecycle in SwiftUI. Today we will concentrate on scene management and the features that the new *Scene* protocol provides us to replace the old *SceneDelegate*.

{% include friends.html %}

#### Basics
The scene is a part of an app's user interface with a lifecycle managed by the system. The system will decide how to present it to a user depending on the running platform. For example, it might be a full-screen window on iOS and watchOS, or it can use the part of the screen to render user interface on macOS and iPadOS. Let's start with a small example.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            Text("Hello World!")
        }
    }
}
```

As you can see in the example above, we use the new app lifecycle API to build our app. The only requirement of *App* protocol is the body property that should return the instance of *Scene* protocol.

> If you are not familiar with the new App lifecycle API in SwiftUI, I highly recommend you to take a look at my ["Managing app in SwiftUI"](/2020/08/19/managing-app-in-swiftui/) post.

Here we use *WindowGroup*. It is one of the primitive scene types that SwiftUI gives us out of the box. *WindowGroup* is a scene that presents a group of identically structured windows. It allows the user to create as many windows with the same user interface as needed and provide functionality for grouping them into tabs on macOS. On iOS and watchOS, *WindowGroup* consumes the entire screen to render its content. 

*WindowGroup* is the scene type that you will mostly use. But there are other scene types like *DocumentGroup* and *Settings* that you can use to build document-based apps or to provide settings window on macOS.

> To learn more about other types of scenes in SwiftUI, take a look at my ["What's new in SwiftUI"](/2020/06/23/what-is-new-in-swiftui/) post.

#### Scene phase
There are several scene callbacks like *sceneDidBecomeActive* and *sceneWillEnterForeground* that we used to handle in *SceneDelegate*. SwiftUI gives us a new solution to achieve the same result in a new way. Scenes provide us the *onChange* modifier and scene phase environment value that we can use to handle scene state changes. Let's take a look at the example.

```swift
import SwiftUI

@main
struct CardioBotApp: App {
    @Environment(\.scenePhase) var scenePhase
    @StateObject var store = Store(
        initialState: AppState(),
        reducer: appReducer,
        environment: .live
    )

    var body: some Scene {
        WindowGroup {
            RootView().environmentObject(store)
        }
        .onChange(of: scenePhase) { phase in    
            switch phase {
            case .background:
                store.send(.notifications(.startObservers))
            case .active:
                store.send(.fetch)
            default: break
            }
        }
    }
}
```

SwiftUI will run the closure that you pass to *onChange* modifier whenever the value of scene phase changes.

#### Custom scenes
Composition is my favorite thing about SwiftUI. You can create many small views and compose them together into a complex picture. The same rule applies to scenes. You can create your custom scene that uses primitive scenes provided by SwiftUI. 

I work on the app that supports both watchOS and iOS. It shares the codebase between two platforms, but it has different feature sets on iOS and watchOS. Let's start with iOS.

```swift
struct MainScene: Scene {
    @Environment(\.scenePhase) var scenePhase
    @ObservedObject var store: Store<AppState, AppAction>

    var body: some Scene {
        WindowGroup {
            RootView().environmentObject(store)
        }.onChange(of: scenePhase) { scenePhase in
            switch scenePhase {
            case .active:
                store.send(.fetch)
            case .background:
                store.send(.startObservers)
            default:
                break
            }
        }
    }
}
```

We create a custom scene type that conforms to *Scene* protocol. The only requirement is the body property that should also return an instance of scene protocol. My app is derived by a global store that I ask to fetch data when the scene becomes active. I even start observers when the scene enters the background. 

On the other hand, my watchOS app doesn't have background observers. It also provides an additional scenes for handling dynamic notifications in SwiftUI. That's why I decided to create another scene for the watchOS app.

```swift
struct WatchAppScene: Scene {
    @Environment(\.scenePhase) var scenePhase
    @ObservedObject var store: Store<AppState, AppAction>

    var body: some Scene {
        WindowGroup {
            RootView().environmentObject(store)
        }.onChange(of: scenePhase) { scenePhase in
            if scenePhase == .active {
                store.send(.fetch)
            }
        }

        WKNotificationScene(controller: DailyReportController.self, category: "dailyReport")
        WKNotificationScene(controller: WorkoutController.self, category: "workoutReport")
    }
}
```

As you can see, my watchOS app is slightly different. Now let's take a look at how we can compose our custom scene types in one app.

```swift
@main
struct CardioApp: App {
    @StateObject var store = Store(
        initialState: AppState(),
        reducer: appReducer,
        environment: AppEnvironment(
            healthService: HealthService(store: .init())
        )
    )

    var body: some Scene {
        #if os(watchOS)
        WatchAppScene(store: store)
        #else
        MainScene(store: store)
        #endif
    }
}
```

#### Conclusion
Today we learned another new SwiftUI API that allows us to handle scenes in a composable way. We understood how to use scenes API to build custom scenes for different platforms. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!