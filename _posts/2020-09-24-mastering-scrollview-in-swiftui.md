---
title: Mastering ScrollView in SwiftUI
layout: post
image: /public/wwdc20.jpg
category: Mastering SwiftUI views
---

We had a scroll view from the very first version of SwiftUI. It was quite limited. But this year changed everything when Apple released ScrollViewReader during WWDC 20. This week we will learn all about scroll views in SwiftUI. We will learn how to scroll to the particular position and read the current offset of scroll view content.

#### Basics
The usage of a scroll view is pretty simple. You create a scroll view and pass the content inside a ViewBuilder closure. Let's take a look at the first example.

=====================================================

We have a few parameters to control the scroll view behavior. We can set the axis of the scroll. It can be horizontal, vertical, or both. Another parameter allows us to show or hide the scrolling indicators.

#### ScrollViewReader
During WWDC20, Apple released ScrollViewReader that allows us to scroll to a particular position. Let's take a look at how we can use it.

=====================================================

As you can see in the example above, we define a ScrollViewReader that passes the scroll view parameter to its ViewBuilder closure. ScrollViewReader traverses its child view, find the first scroll view and pass it into its ViewBuilder closure. The parameter of ViewBuilder closure is an instance of ScrollViewProxy. ScrollViewProxy is a simple struct that provides us scrollTo function. We can use this function to scroll to any view that defines its id.

I have to mention that the scrollTo function is animatable, and you can wrap it using withAnimation function to animate scrolling.

=====================================================

** Tip: You can use ScrollViewReader with List also. **

#### ScrollView content offset
Now we can move scroll view content offset to a particular position, but what about content offset. How can we keep the view updated while the user scrolling the content? We don't have this behavior out of the box, but we can easily implement it using preferences.

> If you are not familiar with preferences in SwiftUI, I suggest reading my "The magic of view preferences in SwiftUI" post.

Let's start with defining a preference key type that will store the current content offset using CGPoint.

=====================================================

Now we can create a view that will replace the scroll view from SwiftUI and reuse it under the hood by providing the ability to read the content offset.

=====================================================

As you can see, we have a pretty similar to SwiftUI's scroll view definition. It uses the same parameters but also adds offsetChanged closure that we will call whenever content offset changes. Let's move to the body property implementation of our scroll view.

=====================================================

As I said before, we use SwiftUI's scroll view under the hood and pass all the parameters to configure its behavior. Before adding the content, we place a GeometryReader view that allows us to track the content offset changes. We use preferences to pass the origin point of our content to the parent view.

> To learn more about GeometryReader, look at my "Building BarChart with Shape API in SwiftUI" post.

#### Conclusion
Today we learned all about using a scroll view in SwiftUI. It looks like now we can use SwiftUI's scroll view and forget about wrapping UIScrollView with UIViewRepresentable. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!