---
title: The difference between @StateObject, @EnvironmentObject, and @ObservedObject in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: Data Flow
---

This week I decided to share as much as I can about data flow in SwiftUI. In this post, we will discuss the difference between @*StateObject*, @*EnvironmentObject*, and @*ObservedObject* property wrappers. I know that this is the most confusing topic in SwiftUI for newcomers.

{% include friends.html %}

#### Why SwiftUI does need property wrappers?
SwiftUI uses immutable struct types to describe the view hierarchy. All the views that SwiftUI provides are immutable. That's why SwiftUI gives us a bunch of property wrappers to handle data mutations. Property wrappers allow us to declare them inside SwiftUI views but store the data outside of the view declaring it.

#### @StateObject
@*StateObject* is the new property wrapper that initializes the instance of a class that conforms to *ObservableObject* protocol and store it inside the SwiftUI framework's internal memory. SwiftUI creates @*StateObject* only once for every container that declares it and keeps it outside of view lifecycle. Let's look at some examples where we use @*StateObject* to keep the whole app's state.

```swift
import SwiftUI

@main
struct CardioBotApp: App {
    @StateObject var store = Store(
        initialState: AppState(),
        reducer: appReducer,
        environment: AppEnvironment(service: HealthService())
    )

    var body: some Scene {
        WindowGroup {
            RootView().environmentObject(store)
        }
    }
}
```

As you can see, @*StateObject* perfectly fits to store the whole app state and share it with different scenes or views of your app. SwiftUI will store it in special framework memory to keep your data in a safe place outside of scene or view lifecycle.

> To learn more about using a single state container, take a look at my ["Redux-like state container in SwiftUI. Basics."](/2019/09/18/redux-like-state-container-in-swiftui/) post.

#### @ObservedObject
@*ObservedObject* is another way to subscribe and keep track of changes in *ObservableObject*. SwiftUI doesn't control the lifecycle of @*ObservedObject*, and you have to manage it yourself. @*ObservedObject* perfectly fits a case where you have an *ObservableObject* stored by @*StateObject*, and you want to share it with any reusable view.

```swift
struct CalendarContainerView: View {
    @ObservedObject var store: Store<CalendarState, CalendarAction>
    let interval: DateInterval

    var body: some View {
        CalendarView(interval: interval) { date in
            DateView(date: date) {
                self.view(for: date)
            }
        }.navigationBarTitle("calendar", displayMode: .inline)
    }
}
```

I mention reusable views because I use *CalendarContainerView* in multiple places in my app, and I don't want to make it dependent on the environment. I use @*ObservedObject* to explicitly indicate the data used by the view in that particular case.

```swift
NavigationLink(
    destination: CalendarContainerView(
        store: transformedStore,
        interval: .twelveMonthsAgo
    )
) {
    Text("Calendar")
}
```

> To learn more about using container views, take a look at my ["Redux-like state container in SwiftUI. Container Views."](/2019/10/02/redux-like-state-container-in-swiftui-part3/) post.

#### @EnvironmentObject
@*EnvironmentObject* is an excellent way to implicitly inject an instance of a class that conforms to *ObservableObject* into a part of the view hierarchy. Assume that you have a module in your app that contains 3-4 screens, and all of them use the same view model. If you don't want to pass the same view model explicitly from one view to another, then @*EnvironmentObject* is all you need. Let's take a look at how we can use it.

```swift
@main
struct CardioBotApp: App {
    @StateObject var store = Store(
        initialState: AppState(),
        reducer: appReducer,
        environment: .production
    )

    var body: some Scene {
        WindowGroup {
            TabView {
                NavigationView {
                    SummaryContainerView()
                        .navigationBarTitle("today")
                        .environmentObject(
                            store.derived(
                                deriveState: \.summary,
                                embedAction: AppAction.summary
                            )
                        )
                }

                NavigationView {
                    TrendsContainerView()
                        .navigationBarTitle("trends")
                        .environmentObject(
                            store.derived(
                                deriveState: \.trends,
                                embedAction: AppAction.trends
                            )
                        )
                }
            }
        }
    }
}
```

In the example above, we inject the environment object into the view hierarchy of *SummaryContainerView*. SwiftUI will implicitly give access for inserted environment objects to all child views that live inside *SummaryContainerView*. We can quickly obtain and subscribe to injected environment objects using @*EnvironmentObject* property wrapper.

```swift
struct SummaryContainerView: View {
    @EnvironmentObject var store: Store<SummaryState, SummaryAction>

    var body: some View {
        //......
    }
}
```

I have to mention that @*EnvironmentObject* has the same lifecycle as @*ObservedObject*. It means that you can get a new environment object whenever you create it inside a view that can be recreated by SwiftUI.

> To learn more about advanced techniques while using a single state container, take a look at my ["Redux-like state container in SwiftUI. Best practices."](/2019/09/25/redux-like-state-container-in-swiftui-part2/) post.

#### Conclusion
Today we talked about the differences between @*StateObject*, @*EnvironmentObject*, and @*ObservedObject* property wrappers. I hope this post makes it easier to understand which property wrapper fits best your case. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!