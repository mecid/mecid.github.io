---
title: Accessibility focus in SwiftUI
layout: post
category: Accessibility
---

One of the new features of SwiftUI Release 3 is accessibility focus management. SwiftUI allows us easily handle the focus state for assistive technologies like VoiceOver and Switch Control. This week we will learn how to use the AccessibilityFocusState property wrapper to move the accessibility focus in SwiftUI.

SwiftUI Release 3 provides us a particular set of tools for managing accessibility focus. It includes AccessibilityFocusState property wrapper and the accessibilityFocused view modifier. We can handle accessibility focus in a similar way that we manage it without assistive technologies.

=====================================================

As you can see in the example above, we use the AccessibilityFocusState property wrapper to define a variable that represents whenever the email field is focused. SwiftUI initializes the variable with the false value by default because the user can focus on any other screen area. We also use the accessibilityFocused view modifier to bind the focus state of a particular view to a variable holding its value. 

Remember that you can declare as many variables as you need to cover your accessibility focus logic with the AccessibilityFocusState property wrapper.

=====================================================

The nice thing about the AccessibilityFocusState property wrapper is that you can limit its behavior to a dedicated assistive technology. For example, you can activate the AccessibilityFocusState property wrapper only for VoiceOver or Switch Control. By default, SwiftUI aggregates the value for all assistive technologies available on the device.     

=====================================================

Usually, you have more than one element on the screen, and you might want to move the focus between them. For this particular case, SwiftUI provides us a way to define our focusable fields via enum and switch between them.

=====================================================

In the example above, we use the AccessibilityFocusState view modifier with our new FocusableField enum that defines all the focusable views on the screen. Keep in mind that you should make the FocusableField enum hashable and define an optional variable with the help of the AccessibilityFocusState view modifier to allow the framework to set the value to nil whenever the user moves the focus from the views you define. 

We should also use another version of the accessibilityFocused view modifier to bind a view to a particular case of the hashable enum.

I love SwiftUI because the different APIs use the same style and are consistent across various features. You can learn it once and apply it in multiple places.
