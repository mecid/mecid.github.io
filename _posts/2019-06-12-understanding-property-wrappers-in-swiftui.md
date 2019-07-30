---
title: Understanding Property Wrappers in SwiftUI
layout: post
---

Last week we started a new series of [posts](/2019/06/05/swiftui-making-real-world-app) about *SwiftUI* framework. Today I want to continue this topic by covering *Property Wrappers* provided by *SwiftUI*. *SwiftUI* gives us *@State*, *@Binding*, *@ObservedObject*, *@EnvironmentObject*, and *@Environment* *Property Wrappers*. So let's try to understand the differences between them and when and why which one we have to use.

#### Property Wrappers
*Property Wrappers* feature described in [SE-0258](https://github.com/DougGregor/swift-evolution/blob/property-wrappers/proposals/0258-property-wrappers.md) proposal. The main goal here is wrapping properties with logic which can be extracted into the separated struct to reuse it across the codebase. This proposal is not accepted yet, but Apple provides *Xcode beta with Swift 5.1 snapshot*, which has it.

#### @State
*@State* is a *Property Wrapper* which we can use to mark *View*'s state. *SwiftUI* will store it in special internal framework memory outside of *View* struct. Only related *View* and its children can access it. As soon as the value of @*State* property change *SwiftUI* rebuilds *View* to respect state changes. Here is a simple example.

```swift
struct ProductsView: View {
    let products: [Product]

    @State private var showFavorited: Bool = false

    var body: some View {
        List {
            Button(
                action: { self.showFavorited.toggle() },
                label: { Text("Change filter") }
            )

            ForEach(products) { product in
                if !self.showFavorited || product.isFavorited {
                    Text(product.title)
                }
            }
        }
    }
}
```

In the example above we have a straightforward screen with *Button* and *List* of products. As soon as we press the button, it changes the value of state property, and *SwiftUI* recreates *View*.

#### @Binding
@*Binding* provides reference like access for a value type. Sometimes we need to make the state of our *View* accessible for its children. But we can't simply pass that value because it is a value type and *Swift* will pass the copy of that value. And this is where we can use *@Binding Property Wrapper*.

```swift
struct FilterView: View {
    @Binding var showFavorited: Bool

    var body: some View {
        Toggle(isOn: $showFavorited) {
            return Text("Change filter")
        }
    }
}

struct ProductsView: View {
    let products: [Product]

    @State private var showFavorited: Bool = false

    var body: some View {
        List {
            FilterView(showFavorited: $showFavorited)

            ForEach(products) { product in
                if !self.showFavorited || product.isFavorited {
                    Text(product.title)
                }
            }
        }
    }
}
```

We use @*Binding* to mark *showFavorited* property inside the *FilterView*. We also use *$* to pass a binding reference, because without *$* *Swift* will pass a copy of the property's value instead of passing bindable reference. *FilterView* can read and write the value of *ProductsView*'s *showFavorited* property, but it can't observe the changes using this binding. As soon as *FilterView* changes value of *showFavorited* property, *SwiftUI* will recreate the *ProductsView* and *FilterView* as its child.

#### @ObservedObject
@*ObservedObject* work very similarly to @*State* *Property Wrapper*, but the main difference is that we can share it between multiple independent *Views* which can subscribe and observe changes on that object, and as soon as changes appear *SwiftUI* rebuild all *Views* bound to this object. Let's take a look at an example.

```swift
import Combine

final class PodcastPlayer: ObservableObject {
    @Published private(set) var isPlaying: Bool = false

    func play() {
        isPlaying = true
    }

    func pause() {
        isPlaying = false
    }
}
```

Here we have *PodcastPlayer* class which we share between the screens of our app. Every screen has to show floating pause button in the case when the app is playing a podcast episode. *SwiftUI* tracks changes on *ObservableObject* with the help of *@Published* property wrapper, when property marked as *@Published* changes *SwiftUI* rebuilds the view.

```swift
struct EpisodesView: View {
    @ObservedObject var player: PodcastPlayer
    let episodes: [Episode]

    var body: some View {
        List {
            Button(
                action: {
                    if self.player.isPlaying {
                        self.player.pause()
                    } else {
                        self.player.play()
                    }
            }, label: {
                    Text(player.isPlaying ? "Pause": "Play")
                }
            )
            ForEach(episodes) { episode in
                Text(episode.title)
            }
        }
    }
}
```

Here we use *@ObservedObject Property Wrapper* to bind our *EpisodesView* to *PodcastPlayer* class, and as soon as current *View* or any other *View* attached to *PodcastPlayer* object change it, *SwiftUI* will rebuild all *Views* bound to that *PodcastPlayer* object.

#### @EnvironmentObject
Instead of passing *ObservedObject* via init method of our View we can implicitly inject it into *Environment* of our *View* hierarchy. By doing this, we create the opportunity for all child *View* of current *Environment* access to this *ObservableObject*.

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        let window = UIWindow(frame: UIScreen.main.bounds)
        let episodes = [
            Episode(id: 1, title: "First episode"),
            Episode(id: 2, title: "Second episode")
        ]

        let player = PodcastPlayer()
        window.rootViewController = UIHostingController(
            rootView: EpisodesView(episodes: episodes)
                .environmentObject(player)
        )
        self.window = window
        window.makeKeyAndVisible()
    }
}

struct EpisodesView: View {
    @EnvironmentObject var player: PodcastPlayer
    let episodes: [Episode]

    var body: some View {
        List {
            Button(
                action: {
                    if self.player.isPlaying {
                        self.player.pause()
                    } else {
                        self.player.play()
                    }
            }, label: {
                    Text(player.isPlaying ? "Pause": "Play")
                }
            )
            ForEach(episodes) { episode in
                Text(episode.title)
            }
        }
    }
}
```

As you can see, we have to pass *PodcastPlayer* object via *environmentObject* method on our *View*. By doing this, we can easily access to *PodcastPlayer* by defining it with *@EnvironmentObject Property Wrapper*. @*EnvironmentObject* uses dynamic member lookup feature to find *PodcastPlayer* class instance in the *Environment*, that's why you even don't need to pass it via init method of *EpisodesView*. The *Environment* is the right way of *Dependency Injection in SwiftUI*. It works like magic.

#### @Environment
As we discussed in the previous chapter, we can pass custom objects into the *Environment* of a *View* hierarchy inside *SwiftUI*. But *SwiftUI* already has an *Environment* populated with system-wide settings. We can easily access them with *@Environment Property Wrapper*.

```swift
struct CalendarView: View {
    @Environment(\.calendar) var calendar: Calendar
    @Environment(\.locale) var locale: Locale
    @Environment(\.colorScheme) var colorScheme: ColorScheme

    var body: some View {
        return Text(locale.identifier)
    }
}
```

By marking our properties with *@Environment Property Wrapper*, we access and subscribe to changes of system-wide settings. As soon as *Locale*, *Calendar* or *ColorScheme* of the system change, *SwiftUI* recreates our *CalendarView*.

#### Conclusion
Today we talked about *Property Wrappers* provided by *SwiftUI*. *@State, @Binding, @EnvironmentObject, and @ObservedObject* play huge role in *SwiftUI* development. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  