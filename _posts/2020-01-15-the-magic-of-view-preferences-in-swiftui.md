---
title: The magic of view preferences in SwiftUI
layout: post
image: /public/swiftui.png
category: Data Flow
---

I took a one week break from SwiftUI topic when we were talking about [building networking in Swift using functions](/2020/01/08/building-networking-layer-using-functions/). It's time to go back to SwiftUI. This week we will talk about view preferences, which is another powerful concept of SwiftUI views that allows us to pass data through view hierarchy.

{% include friends.html %}

#### Preferences
SwiftUI has the environment concept which we can use to pass data down into a view hierarchy. Parent views share its environment with child views and subscribe to the changes. But sometimes we need to pass data up from child view to the parent view, and this is where preferences shine. Let's take a look at the small example.

> To learn more about the benefits of the environment feature take a look at ["The power of Environment in SwiftUI" post](/2019/08/21/the-power-of-environment-in-swiftui/).

```swift
import SwiftUI

struct ContentView: View {
    let messages: [String]

    var body: some View {
        NavigationView {
            List(messages, id: \.self) { message in
                Text(message)
            }.navigationBarTitle("Messages")
        }
    }
}
```

Here is an excellent example of preference usage. *navigationBarTitle* modifier uses the preference feature to pass the data up to the *NavigationView*, which renders it in the navigation bar. Let's take a look at the possible internal implementation of the *navigationBarTitle* modifier.

```swift
import SwiftUI

struct NavigationBarTitleKey: PreferenceKey {
    static var defaultValue: String = ""

    static func reduce(value: inout String, nextValue: () -> String) {
        value = nextValue()
    }
}

extension View {
    func navigationBarTitle(_ title: String) -> some View {
        self.preference(key: NavigationBarTitleKey.self, value: title)
    }
}
```

In order to use preferences, we need to declare a struct conforming *PreferenceKey* protocol. *PreferenceKey* has two requirements, default value for preference and reduce method. Reduce method maintains the logic that combines multiple values into a single one. You might need to replace or append values. In our case, we need to replace the old title with the current one. As you can see, we use a preference modifier to set a value.

```swift
struct ContentView: View {
    let messages: [String]

    var body: some View {
        NavigationView {
            List(messages, id: \.self) { message in
                Text(message)
            }.navigationBarTitle("Messages")
        }.onPreferenceChange(NavigationBarTitleKey.self) { title in
            // you have to set title value in the navigation bar here
            print(title)
        }
    }
}
```

In the example above, we use the *onPreferenceChange* modifier to observe *NavigationBarTitleKey*. SwiftUI will call this closure whenever view sets a new value for the preference.

#### Understanding the size of child view
Sometimes we need to get the size of the child view to make some offset, and it is another excellent example of preference usage in SwiftUI. Let's take a look at how we can use preferences to fetch the size of the child view.

```swift
struct SizePreferenceKey: PreferenceKey {
    static var defaultValue: CGSize = .zero

    static func reduce(value: inout CGSize, nextValue: () -> CGSize) {
        value = nextValue()
    }
}

struct SizeModifier: ViewModifier {
    private var sizeView: some View {
        GeometryReader { geometry in
            Color.clear.preference(key: SizePreferenceKey.self, value: geometry.size)
        }
    }

    func body(content: Content) -> some View {
        content.background(sizeView)
    }
}
```

Here we have a *SizeModifier* struct, which attaches a geometry reader to a view as a background to read its size. It is a pretty useful technique that allows us to calculate the size of the view. Now we can understand the size of the view using *onPreferenceChange* modifier.

> To learn more about the benefits of the view modifiers take a look at ["ViewModifiers in SwiftUI" post](/2019/08/07/viewmodifiers-in-swiftui/).

```swift
struct ScrollView<Content: View>: View {
    let content: Content

    @GestureState private var translation: CGSize = .zero
    @State private var contentSize: CGSize = .zero
    @State private var offset: CGSize = .zero

    private var dragGesture: some Gesture {
        DragGesture(minimumDistance: 0)
            .updating($translation) { value, state, _ in
                state = value.translation
        }.onEnded { value in
            self.offset = value.translation
        }
    }

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        GeometryReader { geometry in
            self.content
                .fixedSize()
                .offset(self.offset)
                .offset(self.translation)
                .modifier(SizeModifier())
                .onPreferenceChange(SizePreferenceKey.self) { self.contentSize = $0 }
                .gesture(self.isScrollable(geometry.size) ? self.dragGesture : nil)
        }.clipped()
    }

    private func isScrollable(_ size: CGSize) -> Bool {
        contentSize.width > size.width || contentSize.height > size.height
    }
}
```

Here is the possible implementation of *ScrollView* that uses preferences to understand the size of its content and enable/disable scrolling based on that value. I use this implementation only for the demo, please don't use it in production.

#### Conclusion
Today we talked about another very great feature of SwiftUI. Preferences feature has the same power as the environment, but instead, it uses reversed direction to pass the data. I'm sure you won't need it very often, but you should know about it. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!