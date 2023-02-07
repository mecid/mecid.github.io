---
title: Mastering charts in SwiftUI. Interactions.
layout: post
category: Mastering SwiftUI views
image: /public/chart12.png
---

The Swift Charts framework provides excellent functionality for implementing super custom charts. This week we will learn how to handle user input with gestures to build interactive charts. The *Chart* type is a simple SwiftUI view, which means you can attach any gestures and buttons to interact with the chart.

{% include friends.html %}

Let's start with a simple example where we attach a drag gesture and change the line's color while the user touches the chart.

```swift
struct ContentView: View {
    @State private var isDragging = false
    @State private var numbers = (0...10).map { _ in
        Int.random(in: 0...10)
    }
    
    var body: some View {
        Chart {
            ForEach(Array(zip(numbers, numbers.indices)), id: \.0) { number, index in
                LineMark(
                    x: .value("Index", index),
                    y: .value("Value", number)
                )
                .foregroundStyle(isDragging ? .red : .blue)
            }
        }
        .gesture(
            DragGesture()
                .onChanged { _ in isDragging = true }
                .onEnded { _ in isDragging = false }
        )
    }
}
```

As you can see in the example above, we reflect the user input by changing the foreground color of the line mark. Being a SwiftUI view allows us to update the chart's content as soon as the state of the view changes. It is a simple rule you must keep in mind and interact with the *Chart* view as you used to interact with other SwiftUI views.

We have looked at the elementary example, but usually, we need to understand which part of the chart user touches and interacts with the particular mark on the chart. The Charts framework provides us with the *ChartProxy* type for these special cases.

The *ChartProxy* type allows us to do a few essential calculations. First, we can get a *Plottable* value by providing a position on the chart. The second thing we can do with the instance of the *ChartProxy* type is to calculate a position for a particular *Plottable* value. Both of these operations might be very helpful for plotting additional content for the points the user interacts with at the very moment.

The *ChartProxy* type also provides us with two beneficial properties, *plotAreaSize*, and *plotAreaFrame*, which we can use to convert coordinates between view space and chart area.

There is no way to create an instance of the *ChartProxy* type, but we can access it through the *chartOverlay* and *chartBackground* modifiers. Let's see how we can use it.

```swift
struct ContentView1: View {
    @State private var selectedIndex: Int? = nil
    @State private var numbers = (0...10).map { _ in
        Int.random(in: 0...10)
    }
    
    var body: some View {
        Chart {
            // ...
        }
        .chartOverlay { chart in
            GeometryReader { geometry in
                Rectangle()
                    .fill(Color.clear)
                    .contentShape(Rectangle())
                    .gesture(
                        DragGesture()
                            .onChanged { value in
                                let currentX = value.location.x - geometry[chart.plotAreaFrame].origin.x
                                guard currentX >= 0, currentX < chart.plotAreaSize.width else {
                                    return
                                }
                                
                                guard let index = chart.value(atX: currentX, as: Int.self) else {
                                    return
                                }
                                selectedIndex = index
                            }
                            .onEnded { _ in
                                selectedIndex = nil
                            }
                    )
            }
        }
    }
}
```

As you can see in the example above, we use the *chartOverlay* modifier to put an overlay view over the chart view with the instance of the *GeometryReader* inside. We need the *GeometryReader* here because the *plotAreaFrame* property provides us an instance of the *Anchor* type, which we can use only in conjunction with the instance of *GeometryProxy* provided by the *GeometryReader*.

After converting the location of the drag gesture to the chart coordinate space, we can use the instance of the *ChartProxy* type to extract the value of the X-axis for the position of the drag gesture.

```swift
struct ContentView1: View {
    @State private var selectedIndex: Int? = nil
    @State private var numbers = (0...10).map { _ in
        Int.random(in: 0...10)
    }
    
    var body: some View {
        Chart {
            ForEach(Array(zip(numbers, numbers.indices)), id: \.0) { number, index in
                if let selectedIndex, selectedIndex == index {
                    RectangleMark(
                        x: .value("Index", index),
                        yStart: .value("Value", 0),
                        yEnd: .value("Value", number),
                        width: 16
                    )
                    .opacity(0.4)
                }

                LineMark(
                    x: .value("Index", index),
                    y: .value("Value", number)
                )
            }
        }
        .chartOverlay { chart in
            // ...
        }
    }
}
```

Finally, we store the index value of the recent drag gesture in the state variable and update our chart content to plot the rectangle to cover the selected area.

In the recent example, we use the *value(forX:)* function on the *ChartProxy* type to read the value by providing the point on the X-axis, but we can also use *value(forY:)* to read by Y-axis, or *value(for:)* to read both by X and Y.

A few functions also allow us to get the location point on the chart by providing a *Plottable* value. Look at the *position(for:)* function on the *ChartProxy* type.
