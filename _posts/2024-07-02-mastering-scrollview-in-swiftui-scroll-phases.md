---
title: Mastering ScrollView in SwiftUI. Scroll Phases
layout: post
---

This year, the SwiftUI framework introduced several new scrolling APIs, allowing us to track and tune everything in a scroll view. This week, we will discuss monitoring scroll phases in SwiftUI.

The SwiftUI framework defines the ScrollPhase enum with a few cases. Let's examine the definition file to understand which functionality it provides.

=====================================================

As you can see, it is a frozen enum, which means it will not change in the future. We have five cases, each representing a particular scrolling phase.

Idle - means nothing is happening, and the scroll view is idle now.
Tracking - the user touches content inside the scroll view but doesn't drag the finger to scroll the content. 
Interacting - the user drags the content to initiate or continue scrolling.
Decelerating - the user finishes interacting with the scroll view, and it smoothly decelerates towards the final destination. 
Animating - the phase indicates that the scroll view programmatically scrolls to the target using ScrollPosition or ScrollViewReader types.

Now, we are familiar with the list of scroll phases we can monitor. The SwiftUI framework introduces the onScrollPhaseChange view modifier, which allows us to observe scroll phase changes. Let's see how we can use it.

=====================================================

As you can see in the example above, we attach the onScrollPhaseChange view modifier to the scroll view. The onScrollPhaseChange view modifier takes an action closure that the SwiftUI framework calls on every scroll phase change. The action closure provides us with two arguments: old and new phases. So you can inspect the recent and previous phases of the scroll view.

Another version of the onScrollPhaseChange view modifier takes an action closure with the additional argument of type ScrollGeometry. 

=====================================================

Having an instance of the ScrollGeometry type might be helpful in certain circumstances, as it provides many valuable properties, such as content size and visible rectangle.

> To learn more about the ScrollGeometry type, take a look at my dedicated "Mastering ScrollView in SwiftUI. Scroll Geometry" post.

The ScrollPhase enum defines different scrolling phases, which is useful if you build something super custom. However, you usually want to know whether or not a scrolling view is scrolling. For this particular case, the ScrollPhase enum introduces the calculatable property isScrolling, which is true when the scroll view is not idle.

=====================================================

As you can see in the example above, we use the isScrolling property to redact our content whenever the scroll view is scrolling.

Today, we learned how to observe scroll phases using the new onScrollPhaseChange view modifier.
