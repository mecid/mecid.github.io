---
title: What is new in SwiftUI after WWDC22
layout: post
---

WWDC22 brings tons of new features to SwiftUI and makes it a full-featured UI framework that we can use daily. Unfortunately, most of the new features are available on iOS 16 and macOS 13. This post will cover the most significant addition and changes to the SwiftUI framework.

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

We use the new version of *NavigationLink* type, allowing us to bind a link to a value. Then we can use the *navigationDestination* view modifier to provide a destination view for a particular value. Remember that we should add the *navigationDestination* view modifier to the view inside the instance of *NavigationStack*.

I love the new Navigation API and the ability to manipulate the navigation stack completely programmatically using the type-safe API.

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
This year another great addition to SwiftUI is the *Layout* protocol that allows us to build super-custom container types. You can create a flow layout or a container size that fits the children equally. You need to create a type conforming to the *Layout* protocol.

```swift
struct EqualWidthStackView: Layout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        let totalSpacing = subviews
            .map(\.spacing)
            .reduce(0.0) {
                $0 + $1.distance(to: .zero, along: .horizontal)
            }

        let maxSize = subviews
            .map { $0.sizeThatFits(proposal) }
            .max { ($0.width * $0.height) > ($1.width * $1.height) } ?? .zero

        let width = maxSize.width * CGFloat(subviews.count) + totalSpacing
        return CGSize(width: width, height: maxSize.height)
    }

    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        let maxSize = subviews
            .map { $0.sizeThatFits(proposal) }
            .max { ($0.width * $0.height) > ($1.width * $1.height) } ?? .zero

        var x: CGFloat = subviews.first?.spacing.distance(to: .zero, along: .horizontal) ?? 0

        for subview in subviews {
            x += subview.spacing.distance(to: .zero, along: .horizontal)
            subview.place(
                at: .init(x: x, y: maxSize.height / 2),
                proposal: proposal
            )
            x += maxSize.width
        }
    }
}
```

Here we have a layout type that finds the biggest child and places the items equally to the biggest one with the defined spacing.

```swift
struct ContentView: View {
    var body: some View {
        EqualWidthStackView {
            Text("Hello, my name is Majid")
            Text("Bye!")
        }
    }
}
```

#### Swift Charts
SwiftUI provides us with a robust charting framework supporting accessibility and localization out of the box.

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

It supports different types of charts that you can mix and combine using declarative syntax.

#### Bottom Sheet
A new *presentationDetents* view modifier allows us to present sheets as bottom sheets. 

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

#### Conclusion
Today we have a massive update for SwiftUI. There are also many new small things like *Transferable* protocol allowing you to share or drag and drop items in your apps and *Grid* view, allowing you to place views in a simple grid. I will try to cover all the new features in the dedicated posts over the following weeks.

