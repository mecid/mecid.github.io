---
title: Custom accessibility content in SwiftUI
layout: post
---

SwiftUI Release 3 brought a lot of new accessibility APIs, which we can use to improve user experience drastically in an effortless way. This week I want to talk about another new API that allows us to provide customized accessibility content using the new accessibilityCustomContent view modifier in SwiftUI.

Let's start with a simple example that defines the User struct and a view presenting an instance of the User struct.

=====================================================

SwiftUI provides us with excellent accessibility support out of the box. You don't need to do anything to make your UserView accessible. Every piece of text inside the UserView is accessible for assistive technologies like VoiceOver and Switch Control. It might sound good, but it can overwhelm Voice Over with a lot of data. Let's improve accessibility support a little bit but adding a few accessibility modifiers to our UserView.

=====================================================

As you can see in the example above, we use accessibility modifiers to ignore the accessibility content of the children and make the stack itself an accessibility element. We also added the accessibility label to the stack but still missed the other data. So we have to make all the data accessible. We use different fonts and colors to prioritize text visually, but how can we achieve the same impact for assistive technologies?

Fortunately, SwiftUI provides a way to provide customized accessibility content with different importance using the brand new accessibilityCustomContent view modifier. Let's take a look at how we can use it.

=====================================================

Here we add a bunch of accessibilityCustomContent view modifiers to define custom accessibility content with various priorities. The accessibilityCustomContent view modifier has three parameters.

The first one is the localized label for your custom content that VoiceOver uses to announce.
The second one is the localized label or string value presenting custom content.
The third one is the importance level of your custom content. It can be default or high. VoiceOver reads content with high importance immediately, while the content with default importance is spoken only in verbose mode when the user uses vertical swipes to access more data.
You can use as many accessibilityCustomContent view modifiers as needed to present a massive subset of your data. Remember that you can also replace and override data or importance by introducing accessibilityCustomContent view modifiers with the same label.

An excellent way to keep your custom accessibility content labels consistent across the large codebase is by using the AccessibilityCustomContentKey type. You can use it as the first parameter of the accessibilityCustomContent view modifier.

=====================================================

In the example above, we define a few shortcuts for our custom accessibility content keys and use them in conjunction with the accessibilityCustomContent view modifier.

Today we learned how to use the accessibilityCustomContent view modifier to make our apps more accessible by prioritizing our data for assistive technologies and allowing user access as more details are needed.
