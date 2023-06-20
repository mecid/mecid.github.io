---
title: Mastering ScrollView in SwiftUI. Target Behavior
layout: post
category: Mastering SwiftUI views
image: /public/scroll-transition.png
---

This year we have massive additions for all APIs related to the ScrollView in SwiftUI. This week we will talk about snapping behavior in ScrollView and how we can customize the scroll target.

The SwiftUI framework provides the brand-new scrollTargetBehavior view modifier allowing us to specify a particular snapping behavior. Let's take a look at a quick example.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
            }
        }
        .scrollTargetBehavior(.paging)
    }
}
```

Here we use the scrollTargetBehavior view modifier with the paging option to enable scrolling by pages. In this case, the ScrollView uses containing size to calculate the next visible target.

The paging instance of the PagingScrollTargetBehavior type conforms to the ScrollTargetBehavior protocol. Another type conforming to the ScrollTargetBehavior protocol is ViewAlignedScrollTargetBehavior.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
            }
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

As you can see in the example above, we use the scrollTargetBehavior view modifier with the viewAligned option to enable view snapping. ScrollView automatically decelerates after scrolling to align with the first visible item in its viewport.

The ScrollView uses the views inside to find the next item to target. Usually, you define the ScrollView with the lazy container inside, like LazyVGrid or LazyVStack. In this case, you should use the scrollTargetLayout view modifier on an instance of the LazyVGrid or LazyVStack to allow the ScrollView to target lazy views outside of the view bounds that still need to be created.

```swift
struct ExampleScrollView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<100) { number in
                    Text(verbatim: String(number))
                        .font(.largeTitle)
                        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                        .background(Rectangle().fill(Color.green))
                }
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

You can also use the scrollTarget view modifier to mark individual views as scroll targets.

```swift
struct AnotherExampleScrollView: View {
    var body: some View {
        ScrollView {
            CustomHeaderView()
                .scrollTarget()
            
            LazyVStack {
                // some content here
            }
            .scrollTargetLayout()
            
            CustomFooterView()
                .scrollTarget()
        }
    }
}
```

Remember that you can use the scrollTargetBehavior view modifier to set the horizontal and vertical scrolling target. It depends on how you allow axes on the ScrollView.

```swift
struct ExampleScrollView: View {
    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack {
                ForEach(0..<100) { number in
                    Text(verbatim: String(number))
                        .font(.largeTitle)
                        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                        .background(Rectangle().fill(Color.green))
                }
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.viewAligned)
    }
}
```

The SwiftUI provides two options for scrolling target behavior: viewAligned and paging. But you can create your types by conforming to the ScrollTargetBehavior protocol.

```swift
struct CustomScrollTargetBehavior: ScrollTargetBehavior {
    func updateTarget(_ target: inout ScrollTarget, context: TargetContext) {
        if context.velocity.dy > 0 {
            target.rect.origin.y = context.originalTarget.rect.maxY
        } else if context.velocity.dy < 0 {
            target.rect.origin.y = context.originalTarget.rect.minY
        }
    }
}

extension ScrollTargetBehavior where Self == CustomScrollTargetBehavior {
    static var custom: CustomScrollTargetBehavior { .init() }
}
```

As you can see in the example above, we define the CustomScrollTargetBehavior type conforming to the ScrollTargetBehavior protocol. It needs to implement the only required function called updateTarget. The updateTarget function has two parameters target and context.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(0..<100) { number in
                Text(verbatim: String(number))
                    .font(.largeTitle)
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 300)
                    .background(Rectangle().fill(Color.green))
            }
        }
        .scrollTargetBehavior(.custom)
    }
}
```

The target parameter is an inout instance of the ScrollTarget type and allows us to set the rect and anchor point for our target inside the ScrollView.

The context parameter is an instance of the ScrollTargetBehaviorContext type and provides information about the context in which the scroll target updates. It provides access to enabled axes, velocity, container size, content size, original target, etc.

With these additions, the ScrollView type gains a lot of power and allows us to build a super-custom experience for our users on all Apple platforms.
