---
title: Mastering ScrollView in SwiftUI. Scroll Offset
layout: post
---

WWDC 24 is over, and I decided to start writing posts about new features in the upcoming iteration of the SwiftUI framework. Apple continues filling gaps this year by introducing more granular control over the scroll position. This week, we will learn how to manipulate and read the scroll offset.

The SwiftUI framework already allows us to track and set the scroll view position by view identifiers. This approach works well but is not enough to track user interactions more accurately.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
        }
        .scrollPosition(initialAnchor: .center)
    }
}
```

Fortunately, the SwiftUI framework introduces the new ScrollPosition type, allowing us to combine the scroll position by offset, the edge of the scroll view, view identifier, etc.

```swift
struct ContentView: View {
    @State private var position = ScrollPosition(edge: .top)
    
    var body: some View {
        ScrollView {
            Button("Scroll to bottom") {
                position.scrollTo(edge: .bottom)
            }
            
            ForEach(1..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
            
            Button("Scroll to top") {
                position.scrollTo(edge: .top)
            }
        }
        .scrollPosition($position)
    }
}
```

As you can see in the example above, we define the position state property and use the scrollPosition view modifier to bind the scroll view with the state property. We also place two buttons allowing you to quickly scroll to the first or latest items in the scroll view. The ScrollPosition type provides the scrollTo function with many overloads, allowing us to handle different cases.

```swift
struct ContentView: View {
    @State private var position = ScrollPosition(edge: .top)
    
    var body: some View {
        ScrollView {
            Button("Scroll to bottom") {
                position.scrollTo(edge: .bottom)
            }
            
            ForEach(1..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
            
            Button("Scroll to top") {
                position.scrollTo(edge: .top)
            }
        }
        .scrollPosition($position)
        .animation(.default, value: position)
    }
}
```

We can easily animate programmatic scrolling by attaching the animation view modifier by passing an instance of the ScrollPositions type as the value parameter.

```swift
struct ContentView: View {
    @State private var position = ScrollPosition(edge: .top)
    
    var body: some View {
        ScrollView {
            Button("Scroll somewhere") {
                let id = (1..<100).randomElement() ?? 0
                position.scrollTo(id: id, anchor: .center)
            }
            
            Button("Scroll to bottom") {
                position.scrollTo(edge: .bottom)
            }
            
            ForEach(1..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
            
            Button("Scroll to top") {
                position.scrollTo(edge: .top)
            }
        }
        .scrollPosition($position)
        .animation(.default, value: position)
    }
}
```

Here, we have added another button to change the position of the scroll view to a random item. We still use the scrollTo function of the ScrollPosition type, but instead of an edge, we provide a hashable identifier. This option allows us to change the position to a particular item, and by using the anchor parameter, we can choose the point of the selected view that should be visible.

Last but not least is the overload of the scrollTo function with point parameter, allowing us to pass an instance of the CGPoint to scroll the view to the particular point of the content. 

```swift
struct ContentView: View {
    @State private var position = ScrollPosition(edge: .top)
    
    var body: some View {
        ScrollView {
            Button("Scroll to offset") {
                position.scrollTo(point: CGPoint(x: 0, y: 100))
            }
            
            ForEach(1..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
        }
        .scrollPosition($position)
        .animation(.default, value: position)
    }
}
```

As you can see in the example above, we use the scrollTo function with a CGPoint parameter. It also provides overloads, allowing us to scroll the view only by the X or Y axis.

```swift
struct ContentView: View {
    @State private var position = ScrollPosition(edge: .top)
    
    var body: some View {
        ScrollView {            
            Button("Scroll to offset") {
                position.scrollTo(y: 100)
                position.scrollTo(x: 200)
            }
            
            ForEach(1..<100) { index in
                Text(verbatim: index.formatted())
                    .id(index)
            }
        }
        .scrollPosition($position)
        .animation(.default, value: position)
    }
}
```

We learned how to manipulate the scroll position using the new ScrollPosition type, which also allows us to read the position of the scroll view. The ScrollPosition provides the optional edge, point, and viewID properties to read the value when you scroll programmatically. Whenever the user interacts with the scroll view, these properties become nil. The isPositionedByUser property on the ScrollPosition type allows us to understand whenever the user gesture moves the scroll view content.

Today, we learned how to programmatically set the offset of the content in a scroll view. However, we can't read the offset of the scroll view using the ScrollPosition type whenever a user interacts with a scroll view using a gesture. To make it possible, the SwiftUI framework provides us with the onScrollGeometryChange view modifier we will cover in the next post.
