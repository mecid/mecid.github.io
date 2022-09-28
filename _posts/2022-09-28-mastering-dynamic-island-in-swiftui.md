---
title: Mastering Dynamic Island in SwiftUI
layout: post
---

In the previous post, we talked about live activity widgets displaying your app's ongoing events. Live activity widgets can utilize the dynamic island of the iPhone 14 Pro. In this post, we will discuss possible configurations and customization points of the dynamic island feature using the new API available in the WidgetKit framework.

> To learn about the basics of live activity widgets, take a look at my dedicated "Displaying live activities in iOS 16" post.

The WidgetKit framework provides us with a particular type of configuration called ActivityConfiguration, allowing us to define a live activity widget.

=====================================================

We use the ActivityConfiguration type to define a SwiftUI view to display on the lock screen. In this case, we will use our custom LiveActivityView. We also provide a dynamic island layout configuration to display on iPhone 14 Pro.

WidgetKit has a particular DynamicIsland type allowing us to specify how we want to use the dynamic island layout. The DynamicIsland type is not a SwiftUI view but requires us to provide views for compact leading and compact trailing, expanded, and minimal cases.

Whenever your app is the only app running live activity widget at the moment, the system displays both compact leading and compact trailing views accordingly. The system uses minimal view whenever there is more than one live activity widget at the very moment. The expanded view is used when the user uses a long press gesture to expand the dynamic island.

=====================image==========================

The expanded dynamic island divides its space into four areas and allows us to control where we want to place the content. We also can use the priority parameter to enable the system to prioritize the views while sizing them.

=====================================================

Sometimes we might have too wide content in the leading view that doesn't fit into the available space. In this case, we can use the dynamicIsland view modifier to move the leading view below the True Depth camera.

=====================================================

Another customization point is the background color of compact and minimal views. By default, the system uses black color to fill the compact and minimal views, but we can use the keylineTint view modifier to change the color.

=====================================================

Today we learned how to use the dynamic island feature to display live activities from your app on iPhone 14 Pro.

