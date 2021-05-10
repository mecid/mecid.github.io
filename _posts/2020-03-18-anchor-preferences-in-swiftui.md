---
title: Anchor preferences in SwiftUI
layout: post
image: /public/anchor.png
category: Layout
---

Today we will continue mastering view preferences in SwiftUI that we touched a few weeks ago. Anchor preferences are another type of view preferences provided by SwiftUI. The main goal of anchor preferences is to pass layout data like bounds, center coordinates, etc. to its parent view.

{% include friends.html %}

#### Basics
First of all, I want to ask you to check the post about view preferences if you are not familiar with these API. Anchor preferences use a very similar API. The only difference is that it is tuned to pass layout-specific data.

> To learn more about the benefits of preferences in SwiftUI, take a look at my ["The magic of view preferences in SwiftUI"](/2020/01/15/the-magic-of-view-preferences-in-swiftui/) post.

Let's build a simple view that shows text and passes its bounds to the ancestor. The parent view will draw an overlaying rectangle in that position.

```swift
struct BoundsPreferenceKey: PreferenceKey {
    typealias Value = Anchor<CGRect>?

    static var defaultValue: Value = nil

    static func reduce(
        value: inout Value,
        nextValue: () -> Value
    ) {
        value = nextValue()
    }
}

struct ExampleView: View {
    var body: some View {
        ZStack {
            Color.yellow
            Text("Hello World !!!")
                .anchorPreference(
                    key: BoundsPreferenceKey.self,
                    value: .bounds
                ) { $0 }
        }
        .overlayPreferenceValue(BoundsPreferenceKey.self) { preferences in
            GeometryReader { geometry in
                preferences.map {
                    Rectangle()
                        .stroke()
                        .frame(
                            width: geometry[$0].width,
                            height: geometry[$0].height
                        )
                        .offset(
                            x: geometry[$0].minX,
                            y: geometry[$0].minY
                        )
                }
            }
        }
    }
}
```

As you can see in the example above, we still use the *PreferenceKey* protocol to create an anchor preference key. It has two requirements: default value and reduce function. Reduce function allows us to merge multiple values that appear from different views. We can replace the current value with the new one for now. We will see more advanced usage of reduce function later in the post.

Anchor preferences use opaque *Anchor* type. You can't merely use Anchor type anywhere in the app. You have to use it in pair with *GeometryProxy* provided by *GeometryReader*. You can use the subscript of *GeometryProxy* to resolve anchor and access wrapped *CGRect* value. As a bonus, SwiftUI will convert a coordinate space between views while solving anchor, and you don't need to do it manually.

We use the *anchorPreference* modifier to define the type of *PreferenceKey* and the value we want to gather. It can be bounds, center, leading, trailing, top, bottom. We also pass a closure that transforms provided anchor value. In this case, we don't need any transformation and return the *CGRect* value as is.

In the end, we use *overlayPreferenceValue* on ancestor view to access gathered preference values and return overlay view. As I mentioned before, we need a *GeometryProxy* to resolve an anchor. That's why we use here *GeometryReader*.

![anchor-basics](/public/anchor-basics.png)

We can easily use border modifier on the text view to achieve the same result, but I've done it to show you the basics of anchor preferences.

#### Advanced usage
Now we can move to more advanced usage of anchor preferences. As an example, we will build a grid view. We will need to gather the size of every view inside the grid to calculate its positions. Let's start by defining the *PreferenceKey* for our grid view.

```swift
struct SizePreferences<Item: Hashable>: PreferenceKey {
    typealias Value = [Item: CGSize]

    static var defaultValue: Value { [:] }

    static func reduce(
        value: inout Value,
        nextValue: () -> Value
    ) {
        value.merge(nextValue()) { $1 }
    }
}
```

As you can see here, we will store the dictionary that represents an item and its size. In the reduce function, we merge old and new dictionaries by overriding new values. Now we can define our grid view.

```swift
struct Grid<Data: RandomAccessCollection, ElementView: View>: View where Data.Element: Hashable {
    private let data: Data
    private let itemView: (Data.Element) -> ElementView

    @State private var preferences: [Data.Element: CGRect] = [:]

    init(_ data: Data, @ViewBuilder itemView: @escaping (Data.Element) -> ElementView) {
        self.data = data
        self.itemView = itemView
    }

    var body: some View {
        GeometryReader { geometry in
            ZStack(alignment: .topLeading) {
                ForEach(self.data, id: \.self) { item in
                    self.itemView(item)
                        .alignmentGuide(.leading) { _ in
                            -self.preferences[item, default: .zero].origin.x
                        }
                        .alignmentGuide(.top) { _ in
                            -self.preferences[item, default: .zero].origin.y
                        }
                        .anchorPreference(
                            key: SizePreferences<Data.Element>.self,
                            value: .bounds
                        ) {
                            [item: geometry[$0].size]
                        }
                }
            }
        }
    }
}
```

We use *ZStack* with top leading alignment. It allows us to position items inside in an effortless way. Instead of using the offset modifier which doesn't affect the layout, we use overridden alignment guides to position our child views. We also resolve our anchors here, because we already have access to the instance of *GeometryProxy*.

> To learn more about the benefits of alignment guides in SwiftUI, take a look at my ["Alignment guides in SwiftUI"]() post.

```swift
var body: some View {
    GeometryReader { geometry in
        ZStack(alignment: .topLeading) {
            ...
        }
        .onPreferenceChange(SizePreferences<Data.Element>.self) { sizes in
            var newPreferences: [Data.Element: CGRect] = [:]
            var bounds: [CGRect] = []
            for item in self.data {
                let size = sizes[item, default: .zero]
                let rect: CGRect
                if let lastBounds = bounds.last {
                    if lastBounds.maxX + size.width > geometry.size.width {
                        let origin = CGPoint(x: 0, y: lastBounds.maxY)
                        rect = CGRect(origin: origin, size: size)
                    } else {
                        let origin = CGPoint(x: lastBounds.maxX, y: lastBounds.minY)
                        rect = CGRect(origin: origin, size: size)
                    }
                } else {
                    rect = CGRect(origin: .zero, size: size)
                }
                bounds.append(rect)
                newPreferences[item] = rect
            }
            self.preferences = newPreferences
        }
    }
}
```

As the last step, we calculate bounds for every item using *onPreferenceChange* modifier which provides us the access to gathered sizes. Let's take a look at the final result.

```swift
struct RootView: View {
    @State private var cards: [String] = [
        "Lorem", "ipsum", "is", "placeholder", "text", "!!!"
    ]

    var body: some View {
        Grid(cards) { card in
            Text(card)
                .frame(width: 120, height: 120)
                .background(Color.orange)
                .cornerRadius(8)
                .padding(4)
        }
    }
}
```

![anchor](/public/anchor.png)

#### Conclusion
SwiftUI provides us so many great tools that we can use to build impressive views. Anchor preferences feature is one of the powerful hidden gems of SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!