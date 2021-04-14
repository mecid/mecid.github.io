---
title: You have to change mindset to use SwiftUI
layout: post
category: View Composition
---

Last week I saw that the community tries to move *UIKit* development patterns to SwiftUI. But I'm sure that the best way to write efficient SwiftUI is to forget everything about *UIKit* and entirely change your mindset in terms of User Interface development. This week we will learn the main differences between *UIKit* and SwiftUI development.

{% include friends.html %}

#### Diffing
*UIKit* is an imperative event-driven framework for building User Interfaces for iOS platform. It means you have to handle all the state changes during events like view loaded, button pressed, etc. The big downside of this approach is the complexity of keeping in sync User Interface with its state. As soon as state changes, you need to manually add/remove/show/hide the views and keep it in sync with the current state. 

SwiftUI is a declarative framework for building User Interfaces on Apple platforms. The keyword here is declarative. Declarative means that you need to declare what you want to achieve, and the framework takes care of it. Framework knows the best way to render the User Interface.

> `UI = f(state)`

The User Interface in SwiftUI is a function of its state. It means whenever the view's state changes, it recomputes its **body property** and generates a new view. Let's take a look at a quick sample.

```swift
struct ContentView: View {
    @ObservedObject var store: Store

    var body: some View {
        Group {
            if store.isLoading {
                Text("Loading...")
                    .font(.subheadline)
            } else {
                Image("photo")
                    .font(.largeTitle)
            }
        }
    }
}
```

As you can see in the example above, we have a view that shows loading text and image when the loading finishes. *ObserverObject* here is a state of this view, and as soon as it changes, SwiftUI recomputes the body property and assigns a new view. In typical *UIKit* development, we need manually to hide/show the elements of the view hierarchy, but in SwiftUI, we don't need to add/remove the loading indicator. We have a few ways of describing a state of the view in SwiftUI, to learn more about them take a look at ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/).

Let's take a more in-depth look at what happens when the view's state changes. SwiftUI has a snapshot of the current view hierarchy, and as soon as state changes, it computes a new view. Finally, SwiftUI applies diffing algorithms to understand differences and automatically add/remove/update needed views. By default, SwiftUI uses standard fade in/out transition to show/hide views, but you can manually change the transition to any animation you want. To learn more about transitions and animations in SwiftUI, take a look at my ["Animations in SwiftUI"](/2019/06/26/animations-in-swiftui/) post.

#### View hierarchy
Let's talk about view hierarchy now, and how actually SwiftUI renders your view struct. The very first thing which I want to mention that SwiftUI doesn't render your view struct by doing the one-to-one mapping. You can use as many view containers as you want, but in the end, SwiftUI renders only views that make sense. It means that you can extract you view logic into small views and then compose and reuse them across the app. Don't worry, performance, in this case, won't be affected. To learn more about view composition in SwiftUI take a look at [the dedicated post](/2019/10/30/view-composition-in-swiftui/).

The best way to understand the complex view hierarchies in SwiftUI is by printing its type. SwiftUI uses the static type system of Swift to make diffing so fast. First of all, it checks the type of the view, and then checks its values of the view components. I'm not a fan of using reflections in production code, but it is very helpful during the learning process.

```swift
print(Mirror(reflecting: ContentView(store: .init()).body))
// Group<_ConditionalContent<Text, ModifiedContent<Image, _EnvironmentKeyWritingModifier<Optional<Font>>>>>
```

By using *Mirror* struct, we can print the real type of the *ContentView*'s body and learn how SwiftUI works under the hood.

#### Conclusion
This week we learned the main difference between *UIKit* and SwiftUI and took an in-depth look at the diffing algorithm in SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 