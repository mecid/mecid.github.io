---
title: Building widgets in SwiftUI
layout: post
image: /public/widget.jpg
category: Mastering SwiftUI views
---

This week I want to talk to you about home-screen widgets for iOS 14. I've built several widget collections for my apps, and it is a perfect time to share with you that experience. Today we will learn all about building and updating widgets with SwiftUI.

#### Basics
Widgets display relevant, glanceable content, letting users quickly get to your app for more details. Your app can provide multiple kinds of widgets, allowing users to focus on essential information. They can add multiple copies of the same widget, tailoring each one to their unique needs and layout. I think it is an excellent definition for widgets, thanks to Apple.

Let's start by adding a widget extension to your app using the Xcode menu: File -> New -> Target -> Widget Extension. Xcode creates a widget from the template. It might look like this.

=====================================================

As you can see, we develop a widget by creating a struct that conforms to the Widget protocol. The only requirement of the Widget protocol is body property that should return an instance of WidgetConfiguration. SwiftUI provides us two structs that conform to WidgetConfiguration: StaticConfiguration and IntentConfiguration. 

We can use IntentConfiguration to provide user-configurable options for our widget. In this post, we will talk about StaticConfiguration. StaticConfiguration allows us to register a widget configured by the developer, and time-to-time updates its data.

StaticConfiguration needs three parameters. Let's take a look at them one-by-one.

Kind is a string that identifies the type of widget. Your app might have multiple widgets. In this case, a kind identifier allows you to update widgets of a particular kind.
The provider is a type conforming to Provider protocol and used by SwiftUI to fetch widget data.
View builder closure that describes the SwiftUI view for displaying widget data.
You can also attach a few modifiers to your widget configuration for setting supported widget families or providing description and name.

#### Provider
The system will call your provider to fetch new data. Let's look at the provider example that I use in my CardioBot app to display daily heart points.

=====================================================

We start by defining type alias for the associated type of Provider protocol. Provider protocol uses the Entry type to describe an item that should be displayed in the widget. Entry must conform to the TimelineEntry protocol that requires only date property.

=====================================================

There are three required functions in the protocol:
The placeholder function is used by the system to generate a template view in the widget gallery. You can provide mock data here that will look nice in the gallery.
getSnapshot function is used when the widget is in a transient state. Call the completion handler as soon as possible to provide data for your widget.
getTimeline function is used by the system to fetch current and future entries for your widget.

getTimeline is the most exciting function here. It allows us to return a timeline of entries for our widget and define a date for the next update.

#### Widget view
Usually, we create a SwiftUI view that accepts an entry and displays it. Then we can easily use it inside a view builder closure in widget configuration. We can use widgetFamily environment value to understand the current widget configuration's size and present different views.

=====================================================

#### Widget updates
The system will try to predict the best time to perform widget updates using providers. But you can always ask to update your widgets using WidgetCenter.

=====================================================

As you can see, there are two options for updating your widgets. You can update all of your widgets or update only the particular kind.

#### Conclusion
Today we learned about another feature that is released during WWDC20. Remember that widgets are available on macOS in the notification center also. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!