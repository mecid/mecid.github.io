---
title: ViewThatFits in SwiftUI
layout: post
image: /public/wwdc22.jpg
category: Mastering SwiftUI views
---

How often did you use *GeometryReader* to measure layout and place different views? *GeometryReader* was always a great tool in our toolbox, but It is elementary to break the layout while using the *GeometryReader*. Fortunately, the next generation of the SwiftUI framework introduces a new way to measure available space and place different views. This week we will learn how to use the brand new *ViewThatFits* view.

{% include friends.html %}

The introduced *ViewThatFits* view is elementary to use. You don't need to manually measure available space or calculate if the particular view fits into the available space. All you need to do is to create an instance of the *ViewThatFits* view and place in its *ViewBuilder* closure up to 10 views. *ViewThatFits* automatically measures available space and the size of its children and takes the first view that perfectly fits the available space. That's it.

```swift
struct ContentView: View {
    var body: some View {
        ViewThatFits {
            HugeView()
            MediumView()
            SmallView()
        }
    }
}
```

As you can see in the example above, it is straightforward to use the new *ViewThatFits* view. But you should be careful about the order of the passed views. *ViewThatFits* uses the first view that fits the available space. It means that you should usually place your views from the biggest to the smallest.

> To learn how to use the GeometryReader properly, look at my ["How to use GeometryReader without breaking SwiftUI layout"](/2020/11/04/how-to-use-geometryreader-without-breaking-swiftui-layout/) post.

By default, the *ViewThatFits* view considers all the available space, but you can set the particular axis you need to measure. For example, it might be a horizontal or vertical axis.

```swift
struct ContentView: View {
    var body: some View {
        ViewThatFits(in: .horizontal) {
            HugeView()
            MediumView()
            SmallView()
        }
    }
}
```

In the current example, we completely ignore the vertical axis and consider only horizontally available space. Let me describe the steps the *ViewThatFits* view applies while choosing a view.

1. *ViewThatFits* measures available space for a particular axis or both of them.
2. It measures the size of the first view and places it if it fits the available space.
3. It measures the size of the second view if the first one doesn't fit the available space and place the second one if it fits.
4. It places the last view in the *ViewBuilder* closure if no view fits the available space.

Today we learned the new and easy way of measuring available space and placing a view fitting that space. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
