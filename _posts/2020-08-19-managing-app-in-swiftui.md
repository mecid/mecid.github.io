---
title: Managing app in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: Data Flow
---

One of the most important things that Apple did release this year was the native app and scene management for SwiftUI apps. This week we will understand how to manage our apps using *App* protocol without old *AppDelegate*. We will learn how to achieve the same set of features with *App* protocol.

{% include friends.html %}

#### Entry point
[SE-0281](https://github.com/apple/swift-evolution/blob/master/proposals/0281-main-attribute.md) introduced type-based program entry points for Swift language. This proposal provides us *@main* attribute that allows us to mark the app's entry point. Swift compiler will recognize this entry point and launch it by running the static main function. Let's take a look at the smallest "Hello World" app example.

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

As you can see in the example above, we create an app by conforming to *App* protocol and implementing the single requirement, *body* property. *Body* property should return an instance of scene protocol, and we use the *WindowGroup* scene here. *WindowGroup* is a standard scene type provided by SwiftUI that handles multiple windows on macOS or various scenes on iPadOS. We will deep dive into scene management in [another post](https://swiftwithmajid.com/2020/08/26/managing-scenes-in-swiftui/).

You may notice that there is no static main function in the code example that Swift compiler uses as the entry point. *App* protocol has the extension that defines the static main function; that's why we don't need to care about it.

#### didBecomeActive and didEnterBackground
We have learned the basics of *App* protocol, and it is time to move forward to understand how we can replace *AppDelegate* callbacks like *didBecomeActive* or *didEnterBackground*. 

```swift
import SwiftUI

@main
struct CardioBotApp: App {
    @Environment(\.scenePhase) var scenePhase

    @StateObject var store = Store(
        initialState: AppState(),
        reducer: appReducer,
        environment: AppEnvironment()
    )

    var body: some Scene {
        WindowGroup {
            RootView()
                .environmentObject(store)
                .onChange(of: scenePhase) { phase in
                    if phase == .active {
                        store.send(.fetch)
                    }
                }
        }
    }
}
```

SwiftUI uses the environment to store the phase of the current scene. You can access it by using the scene phase environment value. The excellent addition to the new app and scene management API is the *onChange* modifier. It runs the provided closure on every change of the value that you pass. In the example above, we fetch data whenever the scene becomes active.

#### Deep linking
Another thing that we used to do in *AppDelegate* is deep linking. SwiftUI provides us a special *onOpenURL* modifier that allows us to handle links.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            Text("Hello World!")
                .onOpenURL { url in
                    if url.absoluteString.contains("today") {
                        // toggle state to navigate to today screen
                    }
                }
        }
    }
}
```

As you can see in the example above, we register a closure that handles URLs. SwiftUI will run it whenever the app should open the URL. We can extract the data from a URL and assign it to the state property that enables the navigation link or switch tabs.

#### Handoff and user activity
Similarly to deep linking, SwiftUI provides us a modifier to handle Handoff and user activity. We can use *onContinueUserActivity* modifier in the very same way as we use *onOpenURL* modifier. Let's take a look at another example.

```swift
import SwiftUI

@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            Text("Hello World!")
                .onContinueUserActivity("com.myapp.today") { userActivity in
                    // toggle state to navigate to today screen
                }
        }
    }
}
```

We use *onContinueUserActivity* modifier to set a closure that will parse a user activity. In the same way, as we did before, we can parse a user activity and toggle state property that routes the navigation. SwiftUI will run this closure as soon as the user continues activity.

#### UIApplicationDelegateAdaptor
As you can see, we can implement many *AppDelegate* callbacks with *App* protocol and the new set of modifiers that SwiftUI provides us. But there are still some gaps.

For example, there is no way to register for remote notifications with *App* protocol. That's why SwiftUI gives us another type called *UIApplicationDelegateAdaptor*. We can merge the functionality of old *AppDelegate* with the new App protocol using *UIApplicationDelegateAdaptor* property wrapper.

```swift
final class AppDelegate: NSObject, UIApplicationDelegate {
    func applicationDidBecomeActive(_ application: UIApplication) {
        application.registerForRemoteNotifications()
    }
}

@main
struct MyApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate

    var body: some Scene {
        WindowGroup {
            Text("Hello World!")
        }
    }
}
```

#### Conclusion
*App* protocol is another step towards the declarative approach that Apple gives us to build our apps. It allows us to replace old *AppDelegate* with a simple struct that conforms to *App* protocol. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!