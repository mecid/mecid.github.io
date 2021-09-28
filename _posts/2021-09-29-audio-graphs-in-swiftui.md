---
title: Audio graphs in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

Charts and graphs are one of the complicated things in terms of accessibility. Fortunately, iOS 15 has a new feature called Audio Graphs. This week we will learn how to build an audio representation for any SwiftUI view presenting a chart like a custom bar chart view or an image by using the *accessibilityChartDescriptor* view modifier.

Let's start by building a simple bar chart view in SwiftUI that displays a set of data points using vertical bars.

```swift
struct DataPoint: Identifiable {
    let id = UUID()
    let label: String
    let value: Double
    let color: Color
}
```

Here we have the *DataPoint* struct that describes the bar in the bar chart view. It has id, label, numeric value, and color to fill. Next, we can define a bar chart view that accepts an array of *DataPoint* instances and displays them.

```swift
struct BarChartView: View {
    let dataPoints: [DataPoint]

    var body: some View {
        HStack(alignment: .bottom) {
            ForEach(dataPoints) { point in
                VStack {
                    RoundedRectangle(cornerRadius: 8, style: .continuous)
                        .fill(point.color)
                        .frame(height: point.value * 50)
                    Text(point.label)
                }
            }
        }
    }
}
```

As you can see in the example above, we have the *BarChartView* that receives an array of *DataPoint* instances and display them as rounded rectangles with different heights in the horizontal stack. I want to appreciate how easily we were able to build a bar chart view in SwiftUI. Let's try to use our new *BarChartView* with sample data.

```swift
struct ContentView: View {
    @State private var dataPoints = [
        DataPoint(label: "1", value: 3, color: .red),
        DataPoint(label: "2", value: 5, color: .blue),
        DataPoint(label: "3", value: 2, color: .red),
        DataPoint(label: "4", value: 4, color: .blue),
    ]

    var body: some View {
        BarChartView(dataPoints: dataPoints)
            .accessibilityElement()
            .accessibilityLabel("Chart representing some data")
    }
}
```

Here we create a sample array of *DataPoint* instances and pass it to the *BarChartView*. We also make an accessibility element for the chart and disable its children's accessibility information. To improve the accessibility experience for our chart view, we also added the accessibility label.

> To learn about the basics of accessibility in SwiftUI, take a look at my ["Accessibility in SwiftUI"](/2019/09/10/accessibility-in-swiftui/) post.

Finally, we can start implementing the audio graph feature for our bar chart view. Audio graphs are available via the rotors menu. To use the rotor, rotate two fingers on your iOS device's screen as if you're turning a dial. VoiceOver will say the first rotor option. Keep rotating your fingers to hear more options. Lift your fingers to choose audio graphs. Then flick your finger up or down on the screen to navigate through it.

Audio graphs allow users to understand and interpret the chart data using audio components. VoiceOver plays sound with different pitches while moving through bars in your chart view. VoiceOver uses high pitches for more significant values and low pitches for small values. These pitches represent the data in your array.

> To learn about the custom rotor navigation in SwiftUI, take a look at my ["Accessibility rotors in SwiftUI"](/2021/09/14/accessibility-rotors-in-swiftui/) post.

Now we can talk about implementing this feature in our *BarChartView*. First of all, we have to create a type conforming to the *AXChartDescriptorRepresentable* protocol. *AXChartDescriptorRepresentable* protocol has only one requirement that creates the instance of *AXChartDescriptor* type. The instance of the *AXChartDescriptor* type represents the data in our chart in the format that VoiceOver can understand and interact with. 

```swift
extension ContentView: AXChartDescriptorRepresentable {
    func makeChartDescriptor() -> AXChartDescriptor {
        let xAxis = AXCategoricalDataAxisDescriptor(
            title: "Labels",
            categoryOrder: dataPoints.map(\.label)
        )

        let min = dataPoints.map(\.value).min() ?? 0.0
        let max = dataPoints.map(\.value).max() ?? 0.0

        let yAxis = AXNumericDataAxisDescriptor(
            title: "Values",
            range: min...max,
            gridlinePositions: []
        ) { value in "\(value) points" }

        let series = AXDataSeriesDescriptor(
            name: "",
            isContinuous: false,
            dataPoints: dataPoints.map {
                .init(x: $0.label, y: $0.value)
            }
        )

        return AXChartDescriptor(
            title: "Chart representing some data",
            summary: nil,
            xAxis: xAxis,
            yAxis: yAxis,
            additionalAxes: [],
            series: [series]
        )
    }
}

```

All we need to do is conforming to the *AXChartDescriptorRepresentable* protocol and add the *makeChartDescriptor* function that returns an instance of *AXChartDescriptor*.

First, we define the X and Y axes by using *AXCategoricalDataAxisDescriptor* and *AXNumericDataAxisDescriptor* types. We want to use string labels on the X-axis. That's why we use the *AXCategoricalDataAxisDescriptor* type. In the case of a line chart, we will use the *AXNumericDataAxisDescriptor* for both axes.

Next, we use the *AXDataSeriesDescriptor* type to define points in our chart. There is the *isContinuous* parameter that allows us to define different chart styles. For example, it should be false for bar charts but true for line charts.

```swift
struct ContentView: View {
    @State private var dataPoints = [
        DataPoint(label: "1", value: 3, color: .red),
        DataPoint(label: "2", value: 5, color: .blue),
        DataPoint(label: "3", value: 2, color: .red),
        DataPoint(label: "4", value: 4, color: .blue),
    ]

    var body: some View {
        BarChartView(dataPoints: dataPoints)
            .accessibilityElement()
            .accessibilityLabel("Chart representing some data")
            .accessibilityChartDescriptor(self)
    }
}
```

As the last step, we use the *accessibilityChartDescriptor* view modifier to set the instance of *AXChartDescriptorRepresentable* protocol to describe our chart.

The audio graphs feature is a significant improvement for visually impaired users. The great thing about the audio graphs feature is that you can use it with any view you want, even with the image view. All you need is to create the instance of AXChartDescriptor type. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
