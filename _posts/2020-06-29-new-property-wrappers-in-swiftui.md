---
title: New property wrappers in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: Data Flow
---

WWDC20 brought a lot of new features into SwiftUI that I will discuss on my blog during the next weeks. Today I would like to start with the main additions to SwiftUI data flow with the brand new @*StateObject*, @*AppStorage*, @*SceneStorage*, and @*ScaledMetric* property wrappers.

{% include friends.html %}

#### StateObject
As you remember, SwiftUI provides us the @*ObservedObject* property wrapper that allows us to observe the changes in the data model that lives outside of the SwiftUI framework. For example, it might be the data that you fetch from web service or the local database. The main concern about @*ObservedObject* was the lifecycle. You have to store it somewhere outside of SwiftUI to save it during view updates, for example, in *SceneDelegate* or *AppDelegate*. In other cases, you can lose the data backed by @*ObservedObject* in certain circumstances.

> If you are not familiar with the legacy property wrappers that SwiftUI provides, I suggest to start with my ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/) post.

Here is the brand new *StateObject* property wrapper that fills the most significant gap in SwiftUI data flow management. SwiftUI creates only one instance of the *StateObject* for each container instance that you declare and holds it in the internal framework memory that saves it during view updates. *StateObject* works in a very similar way to *State* property wrapper, but instead of value types, it is designed to work with reference types.

```swift
struct CalendarContainerView: View {
    @StateObject var viewModel = ViewModel()

    var body: some View {
        CalendarView(viewModel.dates)
            .onAppear(perform: viewModel.fetch)
    }
}
```

#### AppStorage
*AppStorage* is another new property wrapper that we have this year. It is a perfect way to store a small amount of key-value data backed by *UserDefaults*. *AppStorage* is *DynamicProperty*, which means SwiftUI will update your views as soon as the value of a particular key in *UserDefaults* will update. *AppStorage* perfectly fits to store app settings. Let's take a look at how we can use it.

```swift
enum Settings {
    static let notifications = "notifications"
    static let sleepGoal = "sleepGoal"
}

struct SettingsView: View {
    @AppStorage(Settings.notifications) var notifications: Bool = false
    @AppStorage(Settings.sleepGoal) var sleepGoal: Double = 8.0

    var body: some View {
        Form {
            Section {
                Toggle("Notifications", isOn: $notifications)
            }

            Section {
                Stepper(value: $sleepGoal, in: 6...12) {
                    Text("Sleep goal is \(sleepGoal, specifier: "%.f") hr")
                }
            }
        }
    }
}
```

In the example above, we build a settings screen using the *AppStorage* property wrapper and *Form* view. Now we can access our settings anywhere across the app using *AppStorage*, and as soon as we change the values, SwiftUI will update the views.

```swift
struct ContentView: View {
    @AppStorage(Settings.sleepGoal) var sleepGoal = 8
    @StateObject var store = SleepStore()

    var body: some View {
        WeeklySleepChart(store.sleeps, goal: sleepGoal)
            .onAppear(perform: store.fetch)
    }
}
```

#### SceneStorage
This year we got all the abilities to control scenes in SwiftUI without UIKit. As a result, we have a new *SceneStorage* property wrapper that allows us to implement proper state restoration for our scenes. *SceneStorage* works similarly to *AppStorage*, but instead of *UserDefaults*, it uses per-scene storage. It means every scene has its private storage that can't be accessed by other scenes. The system entirely responsible for managing per-scene storage, and you don't have access to the data without *SceneStorage* property wrapper.

```swift
struct ContentView: View {
    @SceneStorage("selectedTab") var selection = 0

    var body: some View {
        TabView(selection: $selection) {
            Text("Tab 1").tag(0)
            Text("Tab 2").tag(1)
        }
    }
}
```

I recommend using *SceneStorage* to store scene-specific data like tab selection or selected book index in a reader app. Nowadays, iOS is very aggressive in terms of killing apps in the background, and state restoration is the key solution to provide a great user experience.

#### ScaledMetric
Another brand new property wrapper that we have is *ScaledMetric*. *ScaledMetric* allows us to scale any binary floating value relative to the Dynamic Type size category. For example, it is so easy to change the spacing in your app according to the Dynamic Type size category. Let's take a look at the small example.

```swift
struct ContentView: View {
    @ScaledMetric(relativeTo: .body) var spacing: CGFloat = 8

    var body: some View {
        VStack(spacing: spacing) {
            ForEach(0...10, id: \.self) { number in
                Text(String(number))
            }
        }
    }
}
```

As soon as user changes the Dynamic Type settings, SwiftUI scales the value of spacing and update the view.

#### Conclusion
Today we learned about new property wrappers in SwiftUI. I believe there are enough data flow property wrappers that can cover any logic we need while implementing our apps. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and have a nice week!