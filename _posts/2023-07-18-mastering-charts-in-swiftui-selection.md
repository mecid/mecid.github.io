---
title: Mastering charts in SwiftUI. Selection
layout: post
---

Swift Charts provides a lovely API allowing you to customize it and add custom interactions within a few lines of code. The following framework iteration goes further and allows us to track chart selection in a single line of code. This week we will learn about new APIs allowing us to handle selection in Swift Charts.

The pre-iOS17 version of Swift Charts provides us with the chartOverlay view modifier allowing us to build custom overlays, including gestures. It also provides APIs for converting geometry position into a chart value. Let's look at how we can use the chartOverlay view modifier to build selection tracking in Swift Charts.

=====================================================

As you can see in the example above, we use the chartOverlay view modifier to add a transparent overlay with a drag gesture. We use an instance of the ChartProxy type to convert the drag position to the chart data. We also draw a RuleMark in the place where the chart is selected.

The code above is simple, but writing for every chart that needs a selection feature is repetitive. Fortunately, the next iteration of the Swift Charts framework includes the chartXSelection and chartYSelection view modifiers allowing us to implement the chart selection feature in a single line.

=====================================================

As you can see, we replaced the whole chart overlay logic with a single line of code using the chartXSelection view modifier. It does the same thing in a single line and looks great. 

The new version of the Swift Charts framework allows us to select a single value and a range of values. We can use chartXSelection and chartYSelection with the binding of ClosedRange type to allow range selection.

=====================================================

In the example above, we use the chartXSelection view modifier with a binding of ClosedRange type. It allows us to draw a rectangle in the selected values range.

The Swift Charts framework also allows us to customize the gesture while selecting by applying the chartGesture view modifier. We can use it to create our own instance of the DragGesture type and set the minimum distance to the particular value.

=====================================================

As you can see, we tuned an instance of the DragGesture type to handle selection in our chart. The chartGesture view modifier provides us access to an instance of the ChartProxy type that we can use to select values or ranges in the chart.
