---
title: Tracking geometry changes in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/layout.png
---

The SwiftUI framework became a mature tool for building apps on all Apple platforms. The recent WWDC introduced missing APIs, adding more value to the framework. One of them is even backward compatible with previous versions of Apple platforms. This week, we will discuss tracking geometry changes of any view in SwiftUI.

{% include friends.html %}

The SwiftUI framework introduced the *onGeometryChange* view modifier, and I am happy to say that it is backward compatible with iOS 16, macOS 13, tvOS 16, watchOS 9, and visionOS 1. The *onGeometryChange* allows us to track geometry changes of any view in SwiftUI.

```swift
struct ContentView: View {
    @State private var size: CGSize = .zero
    
    var body: some View {
        ScrollView {
            Color.red.onGeometryChange(for: CGSize.self) { geometry in
                return geometry.size
            } action: { newValue in
                size = newValue
            }
        }
        .onChange(of: size) {
            print(size)
        }
    }
}
```

As you can see in the example above, we use the *onGeometryChange* view modifier on the instance of the *Color* view. The *onGeometryChange* view modifier takes three parameters. 

The first one is an equatable type of transformation result you will observe. A view's geometry can change very often, especially when it is placed in a scrolling view like *ScrollView* or *List*. That's why you should avoid updating large parts of your app with every geometry update.

The *onGeometryChange* view modifier helps us avoid performance issues by requiring the first parameter, which we have to define the derived type of *GeometryProxy*.

> To learn more about using *GeometryProxy* type in SwiftUI, take a look at my ["How to use GeometryReader without breaking SwiftUI layout"](/2020/11/04/how-to-use-geometryreader-without-breaking-swiftui-layout/) post.

The second parameter is the *transformation* closure, where we take the actual instance of the *GeometryProxy* type and make our transformations to derive the result of the type that we define as the first parameter of the *onGeometryChange* view modifier.

The third parameter is the *action* closure, where we take the result of the transformation closure and can do whatever we want. In our example, we assign the new value to the state property.

Remember that the result type of the *onGeometryChange* view modifier must conform to the *Equatable* protocol. This allows SwiftUI to manage performance and run the *action* closure only when it changes.

```swift
struct ContentView: View {
    @State private var offset: CGFloat = 0
    
    var body: some View {
        ScrollView {
            Color.clear
                .frame(height: 0)
                .onGeometryChange(for: CGFloat.self) { geometry in
                    return geometry.frame(in: .scrollView).minY
                } action: { newValue in
                    offset = newValue
                }
            
                // Scroll content here...
        }
        .onChange(of: offset) {
            print(offset)
        }
    }
}
```

Here is an example of building backward-compatible scroll offset tracking in SwiftUI. You can use this code even on iOS 16. As you can see, we use the *frame* function of the *GeometryProxy* type to calculate the frame of the particular view in different coordinate spaces. In our example, we use the scrollView coordinate space to calculate the frame inside the scroll view. You can also define custom coordinate spaces using the *coordinateSpace* view modifier.

```swift
struct ContentView: View {
    @State private var offset: CGFloat = 0
    
    var body: some View {
        
        ScrollView {
            Color.clear
                .frame(height: 0)
                .onGeometryChange(for: CGFloat.self) { geometry in
                    return geometry.frame(in: .scrollView).minY
                } action: { old, new in
                    offset = min(old, new)
                }
            
                // Scroll content here...
        }
        .onChange(of: offset) {
            print(offset)
        }
    }
}
```

Another version of the *onGeometryChange* view modifier, with the *action* closure taking two parameters: old and new value, is only available on the latest Apple platforms and is not backward compatible.

Today, we learned how to use the new backward-compatible *onGeometryChange* view modifier in SwitUI. It should improve your codebase by reducing the direct usage of the *GeometryReader* type that can easily break your layout. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
