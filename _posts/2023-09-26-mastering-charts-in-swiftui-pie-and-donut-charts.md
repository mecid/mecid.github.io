---
title: Mastering charts in SwiftUI. Pie and Donut charts.
layout: post
---

One of the additions to the Swift Chars framework after WWDC 23 was a brand new SectorMark type. The SectorMark allows us to build pie and donut charts in SwiftUI easily. This week, we will learn how to plot the data using SectorMark.

The Swift Charts framework provides us with mark types like BarMark, LineMark, etc. Mark types allow us to define a piece of data on the chart declaratively. SectorMark is another mark type we can use to plot a pie or donut chart.

=====================================================

As you can see in the example above, we use the SectorMark type to plot a pie chart. We use the foregroundStyle modifier to fill sections with different colors as we do with other mark types.

Whenever we create an instance of the SectorMark type, we must pass the angle parameter. It might be a plottable value defining an angle portion of the section, or we can pass a range with exact start and end values of the angle for a particular area.

The second parameter of the SectorMark initializer is the innerRadius value. We can use it to transform our pie chart into a donut chart.

=====================================================

As you can see in the example above, we use the innerRadius parameter to plot a donut chart. The innerRadius parameter accepts ration, inset, and fixed sizes. We can use the angularInset parameter to set the spacing between sectors of the pie or donut charts.

=====================================================

As you can see, the SectorMark type is a simple chart mark that supports all the modifiers we use for other mark types to customize them.
