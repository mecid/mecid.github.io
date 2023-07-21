---
title: Mastering charts in SwiftUI. Scrolling
layout: post
category: Mastering SwiftUI views
image: /public/chart15.png
---

Another feature we have been waiting for is scrolling charts. The latest version of the Swift Charts framework provides the functionality, allowing us to make any chart scroll in a few different ways. This week we will learn how to make our charts scroll, and the customization points the Swift Charts framework provides.

We can make any chart scrollable by applying to it the *chartScrollableAxes* modifier. You can make it scroll in the horizontal or vertical direction or both. Let's take a look at a quick example.

```swift
struct ContentView: View {
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
    }
}
```

As you can see in the example above, we make our chart scrolling by attaching the *chartScrollableAxes* modifier with horizontal axes. We can also tune the visible amount of data points in the chart using the *chartXVisibleDomain* modifier.

```swift
struct ContentView: View {
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartXVisibleDomain(length: 10)
    }
}
```

You can also use the *chartYVisibleDomain* modifier if you make your chart scroll in the vertical direction. Another customization point in the latest version of the Swift Charts framework is the ability to set an initial scroll position of a particular chart instance using the *chartScrollPosition* modifier.

```swift
struct ContentView: View {
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartScrollPosition(initialX: 50)
        .chartXVisibleDomain(length: 10)
    }
}
```

You must use the *chartScrollPosition* modifier with a value on the particular axes. In our example, we use the *chartScrollPosition* modifier with the number 50 indicating an index of the 50th element in the array.

The Swift Charts framework provides another overload of the *chartScrollPosition* modifier accepting a binding to a plottable value. This version of the *chartScrollPosition* modifier allows us to read and change the visible region of the chart.

```swift
struct ContentView: View {
    @State private var scrollPosition = 50
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartScrollPosition(x: $scrollPosition)
        .onChange(of: scrollPosition) {
            print(scrollPosition)
        }
    }
}
```

Remember that you can both read and write to the current position of the chart using the *chartScrollPosition* modifier accepting a binding. It updates the value of the binding whenever the user changes the visible region of the chart. It also updates the visible region of the chart whenever you change the value of the binding programmatically.

The Swift Charts framework allows us to control the scrolling behavior using the *chartScrollTargetBehavior* modifier. For example, we can use it to enable paging in the chart.

```swift
struct ContentView: View {
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartScrollTargetBehavior(
            .valueAligned(
                unit: 5, 
                majorAlignment: .page
            )
        )
    }
}
```

As you can see in the example above, we use the *chartScrollTargetBehavior* modifier with the value-aligned behavior. The Swift Charts uses the unit parameter to calculate the next target while the user snaps the chart. 

In contrast, the *majorAlignment* parameter estimates the next target while scrolling. In our example, we use the page option as *majorAlignment*, which means the chart will use the visible region's size to calculate the next target. We can also use units to define the major alignment.

```swift
struct ContentView: View {
    @State private var numbers = (0..<100)
        .map { _ in Double.random(in: 0...100) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartScrollTargetBehavior(.valueAligned(unit: 5, majorAlignment: .unit(10)))
    }
}
```

We use dates on X axes very often while building charts. For this particular case, the Swift Charts framework allows to align charts with dates.

```swift
struct ContentView: View {
    let data: [Stats]
    
    var body: some View {
        Chart(data) { item in
            LineMark(
                x: .value("Date", item.date, unit: .day),
                y: .value("Value", item.value)
            )
        }
        .chartScrollableAxes(.horizontal)
        .chartScrollTargetBehavior(
            .valueAligned(
                matching: DateComponents(minute: 0),
                majorAlignment: .matching(DateComponents(hour: 0))
            )
        )
    }
}
```

The value-aligned behavior is an instance of the *ValueAlignedChartScrollTargetBehavior* type conforming to the *ChartScrollTargetBehavior* protocol. You can build your own behaviors by defining a type conforming to the *ChartScrollTargetBehavior* protocol.

> To learn more about *ChartScrollTargetBehavior* protocol and building custom target behaviors, take a look at my ["Mastering ScrollView in SwiftUI. Target Behavior"](/2023/06/20/mastering-scrollview-in-swiftui-target-behavior/) post.

Today we learned how to make our charts scrolling using new APIs available in the Swift Charts framework. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
