---
title: What is new in SwiftUI after WWDC22
layout: post
---

WWDC22 brings tons of new features to SwiftUI and makes it a full-featured UI framework that we can use daily. Unfortunately, most of the new features are available on iOS 16 and macOS 13. This post will cover the most significant addition and changes to the SwiftUI framework.

#### Navigation
SwiftUI 4 provides a new way to build programmatic navigation with deep linking quickly. The old NavigationView is deprecated now, and we should use the new NavigationStack instead. Here is a quick example of how we can use it.

=====================================================

We use the new version of NavigationLink type, allowing us to bind a link to a value. Then we can use the navigationDestination view modifier to provide a destination view for a particular value. Remember that we should add the navigationDestination view modifier to the view inside the instance of NavigationStack.

I love the new Navigation API and the ability to manipulate the navigation stack completely programmatically using the type-safe API.

=====================================================

Here we use another version of the NavigationStack view's initializer to bind its navigation stack to an array of values. It allows us to add and remove views in the navigation stack programmatically.

#### Layout
This year another great addition to SwiftUI is the Layout protocol that allows us to build super-custom container types. You can create a flow layout or a container size that fits the children equally. You need to create a type conforming to the Layout protocol.

=====================================================

Here we have a layout type that finds the biggest child and places the items equally to the biggest one with the defined spacing.

#### Swift Charts
SwiftUI provides us with a robust charting framework supporting accessibility and localization out of the box.

=====================================================

It supports different types of charts that you can mix and combine using declarative syntax.

#### Bottom Sheet
A new presentationDetents view modifier allows us to present sheets as bottom sheets. 

=====================================================

By default, it uses the large appearance, but we allow the user to drag the bottom sheet by adding the medium one.

#### ViewThatFits
There is a new type of view called ViewThatFits. It allows you to place the first or the second provided view depending on the size that fits.

=====================================================

#### Conclusion
Today we have a massive update for SwiftUI. There are also many new small things like Transferable protocol allowing you to share or drag and drop items in your apps and Grid view, allowing you to place views in a simple grid. I will try to cover all the new features in the dedicated posts over the following weeks.

