---
title: Lock screen widgets in SwiftUI
layout: post
---

One of the most requested features for iOS was customizable lock screens. And finally, we have got it with the latest iOS 16. We can populate our lock screen with glanceable widgets. Implementing a lock screen widget is straightforward because its API shares the same code with home screen widgets. This week we will learn how to implement lock screen widgets for our apps.

> Look at my dedicated "Building widgets in SwiftUI" post to learn more about home screen widgets.

Let's start with a code you might already have in your app for displaying home screen widgets.

=====================================================

In the example above, we have a typical view defining a widget. We use the environment to understand the widget family and display a properly sized view. All we need to do to support lock screen widgets is to remove the default statement and implement all the new cases defining lock screen widgets.

=====================================================

It would be best to remember that the system uses different rendering modes for lock screen and home screen widgets. The system provides us with three different rendering modes.

Full-color mode for home screen widgets and watchOS complications supporting colors. And yes, you can also use WidgetKit to implement watchOS complications, starting with watchOS 9.
Vibrant mode is where the system desaturates text, images, and gauges into monochrome and colors them properly for the Lock Screen background.
The accented mode is used only on watchOS, where the system divides the widget into two groups, default and accented. The system colors the accented part of your widget with the tint color the user chooses in the watch face settings.

Rendering mode is available via the SwiftUI environment, so you can always check which rendering mode is active and reflect it in your design. For example, you might use different images with different rendering modes.

=====================================================

As you can see in the example above, we use the widgetRenderingMode environment value to get the actual rendering mode and behave differently. As I said before, in the accented mode, the system divides your widget into two parts and colors them specially. You can mark the part of your view hierarchy with the widgetAccentable view modifier. The system will know which views apply the tint color in this case.

=====================================================

Finally, we need to configure our widget with the supported types.

=====================================================

If you still support iOS 15, you can check the availability of new lock screen widgets.

=====================================================

Today we learned how to implement new lock screen widgets in iOS 16. Remember that we can reuse the same API to build watchOS complications. And you can easily share the widget codebase by looking into environment values to understand which rendering mode is active at the very moment.
