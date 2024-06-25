---
title: Mastering ScrollView in SwiftUI. Scroll Geometry
layout: post
---

The ScrollPosition type is all you need to programmatically read or change the scroll position. Still, it doesn't provide enough information when a user interacts with a scroll view using gestures. SwiftUI solves this problem by introducing the new ScrollGeometry type. This week, we will learn how to use the new onScrollGeometryChange view modifier to monitor scroll geometry.

Let me refresh your memory with the example showing the pros and cons of the ScrollPosition type.

=====================================================

We bind our scroll view to the state property. When we press the button, the scroll view moves its content offset to a particular point. Whenever the user interacts with the scroll view, our state property changes its value to be positioned by the user. In this case, we can't read the particular content offset.

Fortunately, the SwiftUI framework introduces the ScrollGeometry type with the onScrollGeometryChange view modifier, allowing us to read content offset accurately. Let's take a look at how we can use it.

=====================================================

As you can see in the example above, the onScrollGeometryChange view modifier takes three parameters.

The first is the type parameter. Updating the view on every change of the scroll geometry is a very resource-consuming process. That's why SwiftUI asks us to provide a type of piece of the scroll geometry we want to track. In the example, we use CGFloat to track the content offset's Y-axis.

The second parameter is the transformation closure, which takes an instance of the ScrollGeometry type as a parameter. In this closure, we have to extract the information we need from the instance of the ScrollGeometry and return it in a type we defined earlier.

The third parameter is the action closure, which takes old and new values after transformation. We can assign the new value to a view's state property inside the action closure to react to changes.

The ScrollGeometry type provides many valuable properties, such as content offset, bounds, container size, visible rectangle, content insets, and content size.

Remember, you can extract a single property of the ScrollGeometry type and combine two or more properties. For example, we can track the scroll view's content size and visible rectangle.

=====================================================

As you can see in the example above, we introduce the ScrollData holding size and rectangle properties. While using the onScrollGeometryChange view modifier, we pass the ScrollData type as the returning type of the transform closure, where we extract all the needed data from the instance of the ScrollGeometry type.

Today, we learned how to efficiently track and monitor scroll geometry changes in SwiftUI.
