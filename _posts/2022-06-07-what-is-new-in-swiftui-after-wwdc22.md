---
title: What is new in SwiftUI after WWDC22
layout: post
category: Meta
image: /public/wwdc22.jpg
---

WWDC22 brings tons of new features to SwiftUI and makes it a full-featured UI framework that we can use daily. Unfortunately, most of the new features are available on iOS 16 and macOS 13. This post will cover the most significant additions and changes to the SwiftUI framework.

#### Navigation
SwiftUI 4 provides a new way to build programmatic navigation with deep linking quickly. The old *NavigationView* is deprecated now, and we should use the new *NavigationStack* instead. Here is a quick example of how we can use it.

```swift
struct ContentView: View {
    @State private var path: [User] = []
    @State private var users: [User] = [
        .init(name: "Majid")
    ]

    var body: some View {
        NavigationStack(path: $path) {
            List {
                ForEach(users) { user in
                    NavigationLink(user.name, value: user)
                }

            }
            .navigationTitle("Users")
            .navigationDestination(for: User.self) { user in
                UserView(user: user)
                    .navigationTitle(user.name)
            }
        }
    }
}
```

We use the new version of *NavigationLink* type, allowing us to bind a link to a value. Then we can use the *navigationDestination* view modifier to provide a destination view for a particular value.

I love the new Navigation API and the ability to manipulate the navigation stack completely programmatically using the type-safe API. We can implement deep linking easily now.

```swift
struct ContentView: View {
    @State private var path: [User] = []
    @State private var users: [User] = [
        .init(name: "Majid")
    ]

    var body: some View {
        NavigationStack(path: $path) {
            List {
                ForEach(users) { user in
                    NavigationLink(user.name, value: user)
                }

            }
            .navigationTitle("Users")
            .navigationDestination(for: User.self) { user in
                UserView(user: user)
                    .navigationTitle(user.name)
            }
        }
        .onAppear {
            path.append(
                contentsOf: [
                    .init(name: "John"),
                    .init(name: "Majid")
                ]
            )
        }
    }
}
```

Here we use another version of the *NavigationStack* view's initializer to bind its navigation stack to an array of values. It allows us to add and remove views in the navigation stack programmatically.

#### Layout
This year another great addition to SwiftUI is the *Layout* protocol that allows us to build super-custom container types. You can create a flow layout or a container size that fits the children equally. All you need is to create a type conforming to the *Layout* protocol and make your calculations there.

```swift
struct EqualWidthHStack: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        let maxSize = maxSize(subviews: subviews)
        let spacings = spacings(subviews: subviews)
        let totalSpacing = spacings.reduce(0.0, +)
        
        return CGSize(
            width: maxSize.width * CGFloat(subviews.count) + totalSpacing,
            height: maxSize.height
        )
    }
    
    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        let maxSize = maxSize(subviews: subviews)
        let spacing = spacings(subviews: subviews)
        
        let size = ProposedViewSize(
            width: maxSize.width,
            height: maxSize.height
        )
        var x = bounds.minX + maxSize.width / 2
        
        for index in subviews.indices {
            subviews[index].place(
                at: CGPoint(x: x, y: bounds.midY),
                anchor: .center,
                proposal: size
            )
            x += maxSize.width + spacing[index]
        }
    }
    
    private func spacings(subviews: Subviews) -> [CGFloat] {
        let spacing: [CGFloat] = subviews.indices.map { index in
            guard index < subviews.count - 1 else {
                return 0.0
            }
            
            return subviews[index].spacing.distance(
                to: subviews[index + 1].spacing,
                along: .horizontal
            )
        }
        return spacing
    }
    
    private func maxSize(subviews: Subviews) -> CGSize {
        let subviewSizes = subviews.map { $0.sizeThatFits(.unspecified) }
        let maxSize: CGSize = subviewSizes.reduce(.zero) { current, size in
            CGSize(
                width: max(current.width, size.width),
                height: max(current.height, size.height)
            )
        }
        return maxSize
    }
}
```

Here we have a layout type that finds the biggest child and places the items equally to the biggest one with the defined spacing.

```swift
struct ContentView: View {
    var body: some View {
        EqualWidthHStack {
            Text("Hello...")
                .foregroundColor(.red)
            Text("Hello.........")
                .foregroundColor(.green)
            Text("Hello..............")
                .foregroundColor(.blue)
        }
    }
}
```

#### Swift Charts
SwiftUI provides us with a robust charting framework supporting accessibility and localization out of the box. It supports different types of charts that you can mix and combine using declarative syntax.

```swift
import Charts

struct Item: Identifiable {
    let id = UUID()
    let index: Int
    let value: Double
}

struct ContentView: View {
    @State private var items: [Item] = [
        .init(index: 0, value: 1),
        .init(index: 1, value: 5),
        .init(index: 2, value: 0),
        .init(index: 3, value: 7),
        .init(index: 4, value: 4)
    ]
    
    var body: some View {
        Chart(items) { item in
            LineMark(
                x: .value("Index", item.index),
                y: .value("Value", item.value)
            )

            BarMark(
                x: .value("Index", item.index),
                yStart: .value("Start", 0),
                yEnd: .value("End", item.value)
            )
            .foregroundStyle(by: .value("Value", item.value))
        }
    }
}
```

#### Bottom Sheet
The new *presentationDetents* view modifier allows us to present sheet as a bottom sheet and control its size. 

```swift
struct ContentView: View {
    @State private var showSettings = false

    var body: some View {
        Button("View Settings") {
            showSettings = true
        }
        .sheet(isPresented: $showSettings) {
            Text("Settings")
                .presentationDetents([.medium, .large])
        }
    }
}
```

By default, it uses the large appearance, but we allow the user to drag the bottom sheet by adding the medium one.

#### ViewThatFits
There is a new type of view called *ViewThatFits*. It allows you to place the first or the second provided view depending on the size that fits.

```swift
ViewThatFits(in: .horizontal) {
    HStack {
        Image(systemName: "star")
        Text("Long text here")
    }
            
    VStack {
        Image(systemName: "star")
        Text("Long text here")
    }
}
```
#### Many small additions
Today we had a massive update for SwiftUI. There are also many small things like *Transferable* protocol allowing you to share or drag and drop items in your apps. The new *Grid* view allows you to place views in a simple grid. We also have the new single window and menu bar extra scenes with environment values for manipulating windows.

#### Conclusion
I will try to cover all the new features in the dedicated posts over the following weeks. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
