---
title: Taking Control of Toolbar Items in SwiftUI
layout: post
---

I already wrote about Toolbar APIs a few times because it is one of the most important and highly used APIs in SwiftUI. Toolbars play a crucial role, especially with the recent update to the design language where we have a content layer and the UI controls layer above it. This week, we will talk about new APIs allowing us to control toolbar appearance even more.

{% include friends.html %}

Let’s start with a simple example showing us why we need these new APIs.
Toolbar API is highly declarative and adaptive, but sometimes you might need more control over it. For example, here is a toolbar with a bunch of items inside it. It may look and feel really different on macOS, iOS and iPadOS.

=======================================================

The platform running the code above may react differently to this code. On macOS, you will get all actions in the top trailing position. On iOS, you will get some of them in the top trailing position and some of them collapsed in the overflow menu.

=======================================================

That’s why SwiftUI introduces the visibilityPriority modifier that we can use with ToolbarContent. You can use one of the provided options like automatic, low, and high. In this case, SwiftUI will take into consideration the priority and collapse low-priority items.


=======================================================

Let’s take a look at another example. This one is a bit tricky, because on macOS, you will get Action 1 in the top trailing position and the whole group of actions centered in the navigation bar. On iOS, you will get Action 1 pinned to trailing and all other items collapsed.

=======================================================

SwiftUI introduced the special ToolbarOverflowMenu type, allowing you to create collapsed groups of items. It is collapsed by default on all platforms, and you will never guess which toolbar item collapsed or not.

=======================================================

SwiftUI also introduces the topBarPinnedTrailing placement of toolbar item allowing you to pin the item to the top bar trailing. It may collapse the pinned item only when search is active and there is no more room for the item.

=======================================================

The final addition to the Toolbar API is the toolbarMinimizeBehavior view modifier allowing us to minimize toolbars while scrolling up or down. You can minimize tab, bottom, window and navigation bars with it.

With new modifiers SwiftUI gives us a much better balance between system-driven adaptation and explicit control. We can now decide which actions are more important, which items should always stay visible, which groups should intentionally live in the overflow menu, and how toolbars should react to scrolling. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
