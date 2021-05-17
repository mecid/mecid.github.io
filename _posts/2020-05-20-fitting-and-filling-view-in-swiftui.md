---
title: Fitting and filling views in SwiftUI
layout: post
image: /public/fill.png
category: Layout
---

This week I want to continue the topic of layout system in SwiftUI. The SwiftUI layout engine works predictably, and usually, an outcoming result looks like we expect. Today, to make this process even more apparent, we will talk about fitting and filling views in SwiftUI.

{% include friends.html %}

A few weeks ago, we will talk about layout priorities in SwiftUI. Let me refresh your memory by describing how the layout system works in SwiftUI. Usually, a parent view proposes the available space to its child and asks to calculate its size. Then the parent view places the child in the center of available space. Pretty easy, right?

> To learn more about the SwiftUI layout engine, take a look at my ["Layout priorities in SwiftUI"](/2020/04/15/layout-priorities-in-swiftui/) post.

But how exactly view calculates its size? There are two types of views: fitting and filling.

#### Fitting views
A fitting view calculates its size based on its content. It tries to fit its content into available space and return the size. Most of the views we use in SwiftUI are fitting. For example, buttons, stacks, texts, toggles, and pickers. All of them use the content that should be displayed to calculate its size. Let's take a look at the small example.

```swift
struct RootView: View {
    var body: some View {
        HStack {
            Text("Hello World!")
        }.border(Color.red)
    }
}
```

I think the border modifier is the best way to highlight the view's frame. As you can see in the example above, the stack has the size of its content. Stack always uses the space that it needs to place its children.

#### Filling views
A filling view tries to fill all available space provided by its parent view. Usually, this view doesn't have a proper way to understand its content. That's why it fills all the free space. SwiftUI provides us a bunch of filling views. For example, shapes, colors, spacers, dividers, and *GeometryReader*. 

Yes, yes. *GeometryReader* is also a filling view. *GeometryReader* always consumes all the available space provided by its parent and allows you to place its child using a manual calculation based on the given instance of *GeometryProxy* that holds all the needed information about available space and safe area. Let's take a look at another example.

```swift
struct RootView: View {
    var body: some View {
        HStack {
            Circle().fill(Color.green)
        }.border(Color.blue)
    }
}
```

> To learn more about the benefits of GeometryReader view, take a look at my ["Building BarChart with Shape API in SwiftUI"](/2019/08/14/building-barchart-with-shape-api-in-swiftui/) post.

The best way to manage the size of a filling view is by using the *frame* modifier. 

```swift
struct RootView: View {
    var body: some View {
        Color.red.frame(width: 100, height: 100)
    }
}
```

It might be strange, but the color is also a view. It just fills the available space with the color you choose.

#### Both fitting and filling views
There is one exception that I clearly see, and it is Image view. By default, the image component has the size of the image that it should to display. We can call it fitting, but we also can add the *resizable* modifier to the image component, which resizes the image to fill the entire available space. As we do with other filling views, we can control the size of the resizable image by using the *frame* modifier in pair with *scale to fit* modifier to save the aspect ratio of the original image.

#### Conclusion
Today we divide views in SwiftUI into two groups. I believe this post dispels myths about the work of the SwiftUI layout engine. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!