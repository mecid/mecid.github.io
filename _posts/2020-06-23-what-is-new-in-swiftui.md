---
title: What's new in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: Meta
---

I have been waiting for this day for the last nine months, and it has finally arrived. We saw the next iteration of the SwiftUI framework. Apple did a great job during the last year by improving SwiftUI and moving it towards by making it a standalone way for building apps for the Apple ecosystem. Today we will take a peek at all-new SwiftUI features.

{% include friends.html %}

#### App structure
Apple provides a brand new way of defining the app's entry point by using the *App* protocol. *App* protocol allows us to easily replace the *AppDelegate* and *SceneDelegate* with a single struct that will manage our scenes and app lifecycle. Let's take a look at a very quick example.

```swift
@main
struct CardioBotApp: App {
    @SceneBuilder var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

In the example above, we have a struct that conforms to the *App* protocol. The only requirement is the *body* property that should return any *Scene*. The *Scene* is another new protocol introduced by SwiftUI that allows us to declare an app scene. SwiftUI comes with a few ready to use implementations, and one of them is *WindowGroup*. *WindowGroup* is a scene type that we will mostly use for the main interface of our app that isn't document-based.

```swift
@main
struct MyApp: App {
    @SceneBuilder var body: some Scene {
        DocumentGroup(newDocument: TextFile()) { textFile in
            TextEditor(textFile.$document.text)
        }

        #if os(macOS)
        Settings {
            SettingsView()
        }
        #endif
    }
}
```

For document-based apps, SwiftUI provides a *DocumentGroup* scene that automatically handles the navigation through files.

#### Lazy stacks
One thing that I don't like about SwiftUI stacks is the eager initialization. Whenever you have ten or thousand views in a stack, SwiftUI tries to create them immediately. Fortunately, it is changed today. The new version of the SwiftUI framework provides us *LazyHStack* and *LazyVStack* which create its children only when needed.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0...1000, id: \.self) { index in
                    Text(String(index))
                        .onAppear { print(index) }
                }
            }
        }
    }
}
```

#### Grids
It was so tough to create a photo gallery or calendar layout in SwiftUI without *UICollectionView*. Nowadays, most of the apps have a screen with a grid layout. It was nearly impossible to create a grid layout efficiently using SwiftUI. Fortunately, now we can do it using the new *LazyVGrid* and *LazyHGrid* views. Let's take a look at a quick example.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVGrid(
                columns: Array(
                    repeating: GridItem(.flexible()),
                    count: 3
                )
            ) {
                ForEach(0...1000, id: \.self) { _ in
                    Color.red
                }
            }
        }
    }
}
```

#### ScrollView
If you read my post about [SwiftUI wishes](/2020/06/10/swiftui-wishlist-for-wwdc20/), you might know that I have been waiting for an ability to scroll to a particular offset using a *ScrollView*. That part of functionally stopped me from using SwiftUI's *ScrollView*. It is also changed today when Apple released the *ScrollViewReader*. *ScrollViewReader* works very similarly to *GeometryReader* and provides a way to scroll to a specific view using its *ID*.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        ScrollView {
            ScrollViewReader { scrollView in
                LazyVStack {
                    ForEach(0...1000, id: \.self) { index in
                        Text(String(index))
                            .id(index)
                    }
                    Button("Scroll to the middle") {
                        scrollView.scrollTo(500)
                    }
                }
            }
        }
    }
}
```

#### TextEditor
One of the most missing components in SwiftUI was *TextView*. There was no way to edit or type multiline text. We end up with wrapping *TextView* with *UIViewRepresentable*, but it doesn't work well in different circumstances. Finally, this year SwiftUI provides us a *TextEditor* view that allows us to edit multiline text. Let's take a look at how we can use it.

```swift
import SwiftUI

struct ContentView: View {
    @State private var text = "Hello World!"

    var body: some View {
        TextEditor(text: $text)
    }
}
```

The usage is pretty similar to *TextField*, but in this case, we allowed to type very long text examples. There is still no way to use attributed text, but I hope it will arrive during the next betas.

#### New data flow property wrappers
Besides all the new views that Apple released today, we also have brand new ways to handle data flow in SwiftUI. SwiftUI now includes the *AppStorage* property wrapper that accesses *UserDefauls* and invalidates view as soon as corresponding key changes. Let's take a look at the example.

```swift
import SwiftUI

struct ContentView: View {
    @AppStorage("isNotificationsEnabled") var enabled = false

    var body: some View {
        Toggle("Notifications", isOn: $enabled)
    }
}
```

There is also *SceneStorage* property wrapper that we can use for the automatic state restoration of the value. It works similarly to *AppStorage*, but instead of *UserDefaults*, it uses per-scene storage managed by the system.

Another new property wrapper is *StateObject*. *StateObject* works similarly to *State* property wrapper. It allocates memory inside the SwiftUI framework and stores your *ObservableObject* there. It allows to *ObservableObject* to survive during view updates.

```swift
@main
struct CardioBotApp: App {
    @StateObject var store = Store()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(store)
        }
    }
}
```

#### New styling opportunities
One of the best things about SwiftUI is the way that the framework uses to apply styling. Most of the views in SwiftUI provides protocols that we can conform to share the styling across the app. This year the list of styling protocols increased. SwiftUI allows us to transform *TabView* into a paging view by applying *PageTabViewStyle*. There is also a new collection of list styles like *SidebarListStyle*, *InsetGroupedListStyle*, and *InsetListStyle*.

#### New views
This year SwiftUI integrates more deeply with all the frameworks across the Apple ecosystem. For example, MapKit provides *Map* and *MapAnnotations* SwiftUI views. ClockKit provides us a *Gauge* view that we can use to show value within a range. AVKit provides the *VideoPlayer* view that we can use to integrate with *AVPlayer*.

There is also a bunch of new views that SwiftUI provides us today. We finally have a system-wide color picker, native *SignInWithAppleButton*, *ProgressView* that supports both linear and circular progress indicators, *OutlineGroup* that allows us to display tree-structured collections of data, and much more.

#### Conclusion
I'm delighted to see that the next iteration of SwiftUI is so strong. Of course, I will share with you detailed posts about all new features of SwiftUI as soon as I play with them enough to share something. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and have a nice week!