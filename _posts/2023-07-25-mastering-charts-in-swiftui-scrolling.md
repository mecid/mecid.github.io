---
title: Mastering charts in SwiftUI. Scrolling
layout: post
---

Another feature we have been waiting for is scrolling charts. The latest version of the Swift Charts framework provides the functionality, allowing us to make any chart scroll in a few different ways. This week we will learn how to make our charts scroll, and the customization points the Swift Charts framework provides.

We can make any chart scrollable by applying to it the chartScrollableAxes modifier. You can make it scroll in the horizontal or vertical direction or both. Let's take a look at a quick example.

=====================================================

As you can see in the example above, we make our chart scrolling by attaching the chartScrollableAxes modifier with horizontal axes. We can also tune the visible amount of data points in the chart using the chartXVisibleDomain modifier.

=====================================================

You can also use the chartYVisibleDomain modifier if you make your chart scroll in the vertical direction. Another customization point in the latest version of the Swift Charts framework is the ability to set an initial scroll position of a particular chart instance using the chartScrollPosition modifier.

=====================================================

You must use the chartScrollPosition modifier with a value on the particular axes. In our example, we use the chartScrollPosition modifier with the number 50 indicating an index of the 50th element in the array.

The Swift Charts framework provides another overload of the chartScrollPosition modifier accepting a binding to a plottable value. This version of the chartScrollPosition modifier allows us to read and change the visible region of the chart.

=====================================================

Remember that you can both read and write to the current position of the chart using the chartScrollPosition modifier accepting a binding. It updates the value of the binding whenever the user changes the visible region of the chart.

The Swift Charts framework allows us to control the scrolling behavior using the chartScrollTargetBehavior modifier. For example, we can use it to enable paging in the chart.

=====================================================

As you can see in the example above, we use the chartScrollTargetBehavior modifier with the value-aligned behavior. The Swift Charts uses the unit parameter to calculate the next target while the user snaps the chart. 

In contrast, the majorAlignment parameter estimates the next target while scrolling. In our example, we use the page option as majorAlignment, which means the chart will use the visible region's size to calculate the next target. We can also use units to define the major alignment.

=====================================================

We use dates on X axes very often while building charts. For this particular case, the Swift Charts framework allows to align charts with dates.

=====================================================

The value-aligned behavior is an instance of the ValueAlignedChartScrollTargetBehavior type conforming to the ChartScrollTargetBehavior protocol. You can build your own behaviors by defining a type conforming to the ChartScrollTargetBehavior protocol.
