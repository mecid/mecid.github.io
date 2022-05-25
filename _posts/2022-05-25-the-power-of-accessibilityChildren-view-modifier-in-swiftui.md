---
title: The power of accessibilityChildren view modifier in SwiftUI
layout: post
category: Accessibility
---

SwiftUI provides us with a rich set of view modifiers to manipulate the accessibility try of views. I covered many of them, and you can find them in the blog's dedicated "accessibility" category. This week we will talk about the *accessibilityChildren* view modifier and how we can benefit from it.

> To learn more about accessibility view modifiers available in SwiftUI, take a look at the "Accessibility" category on the blog.

The *accessibilityChildren* view modifier allows us to create an accessibility container for a view and populate it with the elements from a view you provide using a *ViewBuilder* closure. Let's take a look at a quick example.

```swift
struct BarChartShape: Shape {
    let dataPoints: [DataPoint]
    
    func path(in rect: CGRect) -> Path {
        Path { p in
            let width: CGFloat = rect.size.width / CGFloat(dataPoints.count)
            var x: CGFloat = 0
            
            for point in dataPoints {
                let pointRect = CGRect(
                    x: x,
                    y: rect.size.height - point.value,
                    width: width,
                    height: rect.size.height
                )
                let pointPath = RoundedRectangle(cornerRadius: 8).path(in: pointRect)
                p.addPath(pointPath)
                x += width
            }
        }
    }
}
```

As you can see in the example above, we have the shape-type drawing data points we pass. We can't provide accessibility values for every data point because the shape becomes a single view after stroking or filling it. 

Fortunately, SwiftUI gives us the *accessibilityChildren* view modifier, especially for this case.

```swift
struct ContentView: View {
    @State private var dataPoints: [DataPoint] = [
        .init(id: .init(), value: 20),
        .init(id: .init(), value: 30),
        .init(id: .init(), value: 5),
        .init(id: .init(), value: 100),
        .init(id: .init(), value: 80)
    ]
    
    var body: some View {
        BarChartShape(dataPoints: dataPoints)
            .accessibilityLabel("Chart")
            .accessibilityChildren {
                HStack(alignment: .bottom, spacing: 0) {
                    ForEach(dataPoints) { point in
                        RoundedRectangle(cornerRadius: 8)
                            .accessibilityValue(Text(point.value.formatted()))
                    }
                }
            }
    }
}
```

By applying the *accessibilityChildren* view modifier, we create an accessibility container and populate it with elements from the view provided in the *ViewBuilder* closure. SwiftUI doesn't render the view that we pass via *ViewBuilder* closure. SwiftUI uses it only for populating the accessibility tree with child elements.

The main difference between the *accessibilityChildren* and *accessibilityRepresentation* view modifiers is that the first one doesn't affect the view itself. It only creates an accessibility container for child elements where the *accessibilityRepresentation* view modifier completely replaces the accessibility tree of the current view.

> To learn more about the benefits of the *accessibilityRepresentation* view modifier, look at my dedicated post.

Today we learned about another powerful accessibility view modifier that SwiftUI provides us. SwiftUI is doing an excellent job by giving us so many friendly APIs, simplifying the work we have to do to make our apps accessible for everyone.
