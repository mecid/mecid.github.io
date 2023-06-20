---
title: Mastering ScrollView in SwiftUI. Target Behavior
layout: post
---

This year we have massive additions for all APIs related to the ScrollView in SwiftUI. This week we will talk about snapping behavior in ScrollView and how we can customize the scroll target.

The SwiftUI framework provides the brand-new scrollTargetBehavior view modifier allowing us to specify a particular snapping behavior. Let's take a look at a quick example.

=====================================================

Here we use the scrollTargetBehavior view modifier with the paging option to enable scrolling by pages. In this case, the ScrollView uses containing size to calculate the next visible target.

The paging instance of the PagingScrollTargetBehavior type conforms to the ScrollTargetBehavior protocol. Another type conforming to the ScrollTargetBehavior protocol is ViewAlignedScrollTargetBehavior.

=====================================================

As you can see in the example above, we use the scrollTargetBehavior view modifier with the viewAligned option to enable view snapping. ScrollView automatically decelerates after scrolling to align with the first visible item in its viewport.

The ScrollView uses the views inside to find the next item to target. Usually, you define the ScrollView with the lazy container inside, like LazyVGrid or LazyVStack. In this case, you should use the scrollTargetLayout view modifier on an instance of the LazyVGrid or LazyVStack to allow the ScrollView to target lazy views outside of the view bounds that still need to be created.

=====================================================

You can also use the scrollTarget view modifier to mark individual views as scroll targets.

=====================================================

Remember that you can use the scrollTargetBehavior view modifier to set the horizontal and vertical scrolling target. It depends on how you allow axes on the ScrollView.

=====================================================

The SwiftUI provides two options for scrolling target behavior: viewAligned and paging. But you can create your types by conforming to the ScrollTargetBehavior protocol.

=====================================================

As you can see in the example above, we define the CustomScrollTargetBehavior type conforming to the ScrollTargetBehavior protocol. It needs to implement the only required function called updateTarget. The updateTarget function has two parameters target and context.

=====================================================

The target parameter is an inout instance of the ScrollTarget type and allows us to set the rect and anchor point for our target inside the ScrollView.

The context parameter is an instance of the ScrollTargetBehaviorContext type and provides information about the context in which the scroll target updates. It provides access to enabled axes, velocity, container size, content size, original target, etc.

With these additions, the ScrollView type gains a lot of power and allows us to build a super-custom experience for our users on all Apple platforms.
