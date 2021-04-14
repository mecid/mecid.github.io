---
title: How to use GeometryReader without breaking SwiftUI layout
layout: post
image: /public/swiftui.png
category: Layout
---

Usually, I try to avoid *GeometryReader* as much as I can. But sometimes, we need it to build our custom view. This week we will talk about *GeometryReader*. The view that allows us to read its geometry and layout child views manually.

{% include friends.html %}

#### Basics
Let's start with a quick example of *GeometryReader* usage.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        GeometryReader { geometry in
            Text("Hello!")
                .frame(
                    width: geometry.size.width,
                    height: geometry.size.height
                ).background(Color.red)
        }
    }
}
```

Here we have a *GeometryReader* in the root of our *ContentView*. By default, *GeometryReader* places its children in the top left corner, which is very unusual for SwiftUI views. Usually, SwiftUI views place content in the center of its coordinate space.

As you can see, *GeometryReader*'s @*ViewBuilder* closure has the parameter called geometry. This parameter is an instance of *GeometryProxy* struct. You can use an instance of *GeometryProxy* to access information about the coordinate space using the following properties.
1. *size* - The size of space available to *GeometryReader*.
2. *safeAreaInsets* - An instance of *EdgeInsets* struct that provides safe area insets of *GeometryReader*.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        GeometryReader { geometry in
            Text("Hello!")
                .frame(
                    width: geometry.frame(in: .global).width,
                    height: geometry.frame(in: .global).height
                ).background(Color.red)
        }
    }
}
```

*GeometryProxy* also has a *frame* function that allows you to access the frame of the *GeometryReader* by converting it into a correct coordinate space. You can also create your own coordinate spaces by using *coordinateSpace* modifier on any view.

> Take a look at ["Building Bottom sheet in SwiftUI"](/2019/12/11/building-bottom-sheet-in-swiftui/) post to learn more about advanced usages of *GeometryReader*.

#### Hints to follow while using GeometryReader
Now we know how to use *GeometryReader* and its benefits. It is a perfect time to talk about its disadvantages. *GeometryReader* fills all the available space, and usually, it is not something you want to achieve. 

If you use *GeometryReader* as a full-screen view, that is great, and you don't need to do anything. In other cases, try to limit the available space of *GeometryReader* by using *frame* or *aspectRatio* modifiers. These modifiers allow you to keep *GeometryReader* under control.

The second tip is to use *GeometryReader* inside an overlay or background of any view. SwiftUI keeps overlay and background views in the same size as the view that you apply them. It limits the size of *GeometryReader* and doesn't allow it to grow and fill all the available space.

> To learn more about this approach, take a look at ["The magic of view preferences in SwiftUI"](/2020/01/15/the-magic-of-view-preferences-in-swiftui/) post.

In case when you need to draw something, you can replace the *GeometryReader* by using Shape API. Shape API provides you all the needed information about available space using *Path* struct.

> To learn more about Shape API, look at ["Building BarChart with Shape API in SwiftUI"](/2019/08/14/building-barchart-with-shape-api-in-swiftui/) post.

#### Conclusion
I suggest you avoid *GeometryReader* where you can. In the case where you sure that you need it, try to follow my tips. These tips should save many hours of debugging the SwiftUI layout. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!