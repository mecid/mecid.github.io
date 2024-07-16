---
title: Mastering ScrollView in SwiftUI. Scroll Visibility
layout: post
---

Another great addition to our scrolling APIs this year is the scroll visibility. Nowadays, you can fetch the list of visible identifiers or quickly check and monitor the view visibility inside a scroll view. This week, we will learn how to use the new onScrollTargetVisibilityChange and onScrollVisibilityChange view modifiers.

Let's kick off with the onScrollTargetVisibilityChange view modifier. It's designed for ease of use, allowing you to attach it to any scroll view with a scroll target layout. Let's explore this with an example.

=====================================================

As you can see in the example above, we use the scrollTargetLayout view modifier on the lazy stack to allow scroll view targeting for stack children instead of the stack itself. 

> To learn more about the `scrollTargetLayout` view modifier, look at my "Mastering ScrollView in SwiftUI. Target Behavior" post.

We also attach the onScrollTargetVisibilityChange view modifier to the scroll view by providing the identifier type and the action closure. Inside the action closure, we get the list of visible identifiers and can do whatever we need with visible items.

Sometimes, the view must act when it is visible or not in a scroll view. For these cases, the SwiftUI framework introduced the onScrollVisibilityChange view modifier, which you can attach to any view inside the scroll view to handle its visibility.

=====================================================

The example above defines the VideoPlayerView view, which auto-plays video content whenever it is visible in a scroll view. As you can see, we attach the onScrollVisibilityChange view modifier to the view itself and provide an action closure. We gain access to the visibility parameter inside the action closure and can react to its changes.

Both the onScrollVisibilityChange and onScrollTargetVisibilityChange modifiers have the threshold parameter. The threshold parameter allows us to tune the required amount of the viewport that should be visible to run the action closure. By default, the SwiftUI framework uses 0.5 as the threshold value, which means that at least 50% of the view should be visible to SwiftUI to run the action. But you can easily adjust this value.

=====================================================

In the example above, we define the threshold value, which means SwiftUI will run the action closure whenever at least 10% of the view is visible. Still, it also runs whenever the view transitions from visible to invisible by displaying less than 10% of the viewport.

Today, we learned how to track the visibility of the particular view inside the scroll view and monitor the list of visible identifiers.
