---
title: Mastering charts in SwiftUI. Customizations.
layout: post
image: /public/chart14.png
category: Mastering SwiftUI views
---

The Swift Charts framework became a huge topic on my blog. But I decided to continue this subject to cover everything I've experienced with the Charts framework. This week we will learn how to customize the Chart view using a bunch of chart view modifiers provided by the framework.

{% include friends.html %}

#### Plot area
The first thing you might need is the tuning plot area of the Chart view. The Charts framework provides the *chartPlotStyle* view modifier allowing us to style the view representing the chart's plot area. Let's take a look at how we can use it.

```swift
struct ContentView: View {
    var body: some View {
        Chart {
            // ...
        }
        .chartPlotStyle { chartContent in
            chartContent
                .background(Color.secondary.opacity(0.2))
                .frame(height: 32)
            
        }
    }
}
```

As you can see in the example above, we use the *chartPlotStyle* view modifier to access the chart's content view. It is a simple SwiftUI view, meaning we can apply any view modifiers we need. In our case, we use the *frame* view modifier to set a particular height and the *background* view modifier to fill the background with a gray color. 

![stacked-bar-chart](/public/chart14.png)

As you know, the *Chart* view is also a SwiftUI view, and we can apply the *frame* view modifier to it, but in this case, it will affect the whole chart view with its legend view, which is not a good idea.

#### Axis
The Charts framework provides us *chartXAxis* and *chartYAxis* view modifiers allowing us to control the look and feel of the particular axis completely. These view modifiers accept *AxisContentBuilder* closure, where we can define how we want to build the axis.

```swift
struct ContentView: View {
    var body: some View {
        Chart {
            // ...
        }
        .chartXAxis {
            AxisMarks(
                preset: .aligned,
                position: .bottom,
                values: .stride(by: 500),
                stroke: StrokeStyle(
                    lineWidth: 4,
                    lineCap: .butt,
                    lineJoin: .bevel,
                    miterLimit: 1,
                    dash: [],
                    dashPhase: 1
                )
            )
        }
    }
}
```

The Charts framework includes *AxisMarks* type representing the chart's axes. We can use it to customize the axis in different ways. In the example above, we use the example of the *AxisMark* type, allowing us to configure the axis's preset, position, values, and stroke.

The *preset* parameter provides *automatic, aligned, inset, and extended* options. They all affect how the framework draws the value on the axis. The *position* parameter allows us to choose where to draw axis values. We can use the *values* parameter to generate values to draw on the axis. In our example, we use the *stride* function to generate values. The last parameter is the *stroke*. The framework uses it to draw axis lines.

We can also use another variant of the *AxisMarks* to fully customize the look and feel by manually adjusting every axis component per axis value.

```swift
struct ContentView: View {
    var body: some View {
        Chart {
            // ...
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: 500)) { value in
                AxisGridLine(
                    centered: true,
                    stroke: StrokeStyle(
                        lineWidth: 4,
                        lineCap: .butt,
                        lineJoin: .bevel,
                        miterLimit: 1,
                        dash: [],
                        dashPhase: 1
                    )
                )
                
                AxisValueLabel(anchor: .topTrailing)
                
                if let value = value.as(Int.self), value % 2 == 0 {
                    AxisTick(centered: true, length: 10)
                }
            }
        }
    }
}
```

As you can see in the example above, we use another variant of the *AxisMarks* initializer, allowing us to configure the axis look and feel per value. We use the concrete value to decide whether to show the tick. The last example demonstrates how we can use *AxisGridLine*, *AxisValueLabel*, and *AxisTick* types to compose the axis. They provide many customization points, allowing you to reach the desired look and feel.

