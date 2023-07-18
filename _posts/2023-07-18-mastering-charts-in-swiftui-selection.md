---
title: Mastering charts in SwiftUI. Selection
layout: post
category: Mastering SwiftUI views
---

Swift Charts provides a lovely API allowing you to customize it and add custom interactions within a few lines of code. The following framework iteration goes further and allows us to track chart selection in a single line of code. This week we will learn about new APIs allowing us to handle selection in Swift Charts.

{% include friends.html %}

The pre-iOS17 version of Swift Charts provides us with the *chartOverlay* view modifier allowing us to build custom overlays, including gestures. It also provides APIs for converting geometry position into a chart value. Let's look at how we can use the *chartOverlay* view modifier to build selection tracking in Swift Charts.

```swift
struct SelectionExample: View {
    @State private var selectedIndex: Int?
    @State private var numbers = (0..<10)
        .map { _ in Double.random(in: 0...10) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
            
            if let selectedIndex {
                RuleMark(x: .value("Index", selectedIndex))
                    .annotation(position: .bottom) {
                        Text(verbatim: selectedIndex.formatted())
                            .padding()
                            .background(.regularMaterial)
                    }
            }
        }
        .chartOverlay { chart in
            GeometryReader { geometry in
                Rectangle().fill(.primary.opacity(0.01)).containerShape(.rect)
                    .gesture(
                        DragGesture()
                            .onEnded { _ in
                                selectedIndex = nil
                            }
                            .onChanged { value in
                                guard let plotFrame = chart.plotFrame else {
                                    return
                                }
                                
                                let currentX = value.location.x - geometry[plotFrame].origin.x
                                
                                if let index: Int = chart.value(atX: currentX) {
                                    selectedIndex = index
                                }
                            }
                    )
            }
        }
    }
}
```

As you can see in the example above, we use the *chartOverlay* view modifier to add a transparent overlay with a drag gesture. We use an instance of the *ChartProxy* type to convert the drag position to the chart data. We also draw a *RuleMark* in the place where the chart is selected.

The code above is simple, but writing for every chart that needs a selection feature is repetitive. Fortunately, the next iteration of the Swift Charts framework includes the *chartXSelection* and *chartYSelection* view modifiers allowing us to implement the chart selection feature in a single line.

```swift
struct SelectionExample: View {
    @State private var selectedIndex: Int?
    @State private var numbers = (0..<10)
        .map { _ in Double.random(in: 0...10) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
            
            if let selectedIndex {
                RuleMark(x: .value("Index", selectedIndex))
                    .annotation(position: .bottom) {
                        Text(verbatim: selectedIndex.formatted())
                            .padding()
                            .background(.regularMaterial)
                    }
            }
        }
        .chartXSelection(value: $selectedIndex)
    }
}
```

As you can see, we replaced the whole chart overlay logic with a single line of code using the *chartXSelection* view modifier. It does the same thing in a single line and looks great. 

The new version of the Swift Charts framework allows us to select a single value and a range of values. We can use *chartXSelection* and *chartYSelection* with the binding of *ClosedRange* type to allow range selection.

```swift
struct RangeSelectionExample: View {
    @State private var selectedRange: ClosedRange<Int>?
    @State private var numbers = (0..<10)
        .map { _ in Double.random(in: 0...10) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
            
            if let selectedRange {
                RectangleMark(
                    xStart: .value("Index", selectedRange.lowerBound),
                    xEnd: .value("Index", selectedRange.upperBound),
                    yStart: .value("Number", 0),
                    yEnd: .value("Number", 10)
                )
                .foregroundStyle(.blue.opacity(0.03 ))
            }
        }
        .chartXSelection(range: $selectedRange)
    }
}
```

In the example above, we use the *chartXSelection* view modifier with a binding of *ClosedRange* type. It allows us to draw a rectangle in the selected values range.

The Swift Charts framework also allows us to customize the gesture while selecting by applying the *chartGesture* view modifier. We can use it to create our own instance of the *DragGesture* type and set the minimum distance to the particular value.

```swift
struct RangeSelectionExample: View {
    @State private var selectedRange: ClosedRange<Int>?
    @State private var numbers = (0..<10)
        .map { _ in Double.random(in: 0...10) }
    
    var body: some View {
        Chart(Array(zip(numbers.indices, numbers)), id: \.1) { index, number in
            LineMark(
                x: .value("Index", index),
                y: .value("Number", number)
            )
            
            if let selectedRange {
                RectangleMark(
                    xStart: .value("Index", selectedRange.lowerBound),
                    xEnd: .value("Index", selectedRange.upperBound),
                    yStart: .value("Number", 0),
                    yEnd: .value("Number", 10)
                )
                .foregroundStyle(.blue.opacity(0.03 ))
            }
        }
        .chartXSelection(range: $selectedRange)
        .chartGesture { chart in
            DragGesture(minimumDistance: 16)
                .onChanged {
                    chart.selectXRange(
                        from: $0.startLocation.x,
                        to: $0.location.x
                    )
                }
                .onEnded { _ in selectedRange = nil }
        }
    }
}
```

As you can see, we tuned an instance of the *DragGesture* type to handle selection in our chart. The *chartGesture* view modifier provides us access to an instance of the *ChartProxy* type that we can use to select values or ranges in the chart.
