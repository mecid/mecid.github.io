---
title: Mastering charts in SwiftUI. Pie and Donut charts.
layout: post
image: /public/donut1.png
category: Mastering SwiftUI views
---

One of the additions to the Swift Chars framework after WWDC 23 was a brand new *SectorMark* type. The *SectorMark* allows us to build pie and donut charts in SwiftUI easily. This week, we will learn how to plot the data using *SectorMark*.

{% include friends.html %}

The Swift Charts framework provides us with mark types like *BarMark*, *LineMark*, etc. Mark types allow us to define a piece of data on the chart declaratively. *SectorMark* is another mark type we can use to plot a pie or donut chart.

```swift
import SwiftUI
import Charts

struct Product: Identifiable {
    let id = UUID()
    let title: String
    let revenue: Double
}

struct SectorChartExample: View {
    @State private var products: [Product] = [
        .init(title: "Annual", revenue: 0.7),
        .init(title: "Monthly", revenue: 0.2),
        .init(title: "Lifetime", revenue: 0.1)
    ]
    
    var body: some View {
        Chart(products) { product in
            SectorMark(
                angle: .value(
                    Text(verbatim: product.title),
                    product.revenue
                )
            )
            .foregroundStyle(
                by: .value(
                    Text(verbatim: product.title),
                    product.title
                )
            )
        }
    }
}
```

![pie-chart](/public/pie.png)

As you can see in the example above, we use the *SectorMark* type to plot a pie chart. We use the *foregroundStyle* modifier to fill sections with different colors as we do with other mark types.

Whenever we create an instance of the *SectorMark* type, we must pass the *angle* parameter. It might be a plottable value defining an angle portion of the section, or we can pass a range with exact start and end values of the angle for a particular area.

The second parameter of the *SectorMark* initializer is the *innerRadius* value. We can use it to transform our pie chart into a donut chart.

```swift
struct SectorChartExample: View {
    @State private var products: [Product] = [
        .init(title: "Annual", revenue: 0.7),
        .init(title: "Monthly", revenue: 0.2),
        .init(title: "Lifetime", revenue: 0.1)
    ]
    
    var body: some View {
        Chart(products) { product in
            SectorMark(
                angle: .value(
                    Text(verbatim: product.title),
                    product.revenue
                ),
                innerRadius: .ratio(0.6)
            )
            .foregroundStyle(
                by: .value(
                    Text(verbatim: product.title),
                    product.title
                )
            )
        }
    }
}
```

![donut-chart](/public/donut.png)

As you can see in the example above, we use the *innerRadius* parameter to plot a donut chart. The *innerRadius* parameter accepts *ratio*, *inset*, and *fixed* sizes. We can use the *angularInset* parameter to set the spacing between sectors of the pie or donut charts.

```swift
struct SectorChartExample: View {
    @State private var products: [Product] = [
        .init(title: "Annual", revenue: 0.7),
        .init(title: "Monthly", revenue: 0.2),
        .init(title: "Lifetime", revenue: 0.1)
    ]
    
    var body: some View {
        Chart(products) { product in
            SectorMark(
                angle: .value(
                    Text(verbatim: product.title),
                    product.revenue
                ),
                innerRadius: .ratio(0.6),
                angularInset: 8
            )
            .foregroundStyle(
                by: .value(
                    Text(verbatim: product.title),
                    product.title
                )
            )
        }
    }
}
```

![donut-chart-with-spacing](/public/donut1.png)

As you can see, the *SectorMark* type is a simple chart mark that supports all the modifiers we use for other mark types to customize them. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
