---
title: Container relative frames in SwiftUI
layout: post
---

Container relative frames in SwiftUI

The easiest way to size a view in SwiftUI is to place it in a container and allow it to fit its content size. You can also use the frame view modifier to specify a particular concrete size. Anything related to the size of its parent needs hard work using GeometryReader, which is not the easiest way to do things correctly in SwiftUI.

Fortunately, Apple introduced container-relative sizing APIs in SwiftUI, that we will learn how to use this week. The new containerRelativeFrame view modifier allows us to set the frame of the view relative to its parent. It provides many options allowing us to customize the size, but let’s start with the simplest one.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, World!")
            .containerRelativeFrame([.horizontal], alignment: .leading)
            .background(Color.pink)
    }
}
```

As you can see in the example above, we use the containerRelativeFrame view modifier and provide the axis we want to fill. It is the easiest way to fill an entire parent by providing an axis. You can provide both vertical and horizontal axes to fill the entire space. The second parameter allows us to set the alignment in the filled space.


Another overload of the containerRelativeFrame view modifier allows us to size the view inside the parent according to the number of items that should be visible at the same time inside a container. It becomes super useful when you use it inside a scroll view with a lazy container.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: 0) {
                ForEach(0..<10) { _ in
                    Color.random
                        .containerRelativeFrame(
                            .horizontal,
                            count: 3,
                            span: 1,
                            spacing: 0
                        )
                }
            }
        }
    }
}
```

Here we use the containerRelativeFrame view modifier with count and span parameters. The count and span parameters work in pair, SwiftUI divides the container’s provided space by the count and then multiplies it by the span to calculate the final size. You can still use the alignment parameter if needed.

The last but not least overload of the containerRelativeFrame view modifier allows you to apply the custom logic to calculate the final size of the view by providing you available space and the axis in which you need to provide a value.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: 0) {
                ForEach(0..<10) { _ in
                    Color.random
                        .containerRelativeFrame([.horizontal], alignment: .center) { len, axis in
                            switch axis {
                            case .horizontal:
                                return len / 3
                            case .vertical:
                                return len / 5
                            }
                        }
                }
            }
        }
    }
}
```

As you can see in the example above, we use the containerRelativeFrame view modifier and provide a special closure where we calculate the size of the view depending on available space and the axis. This overload provides you full control on the size calculation of the view.

