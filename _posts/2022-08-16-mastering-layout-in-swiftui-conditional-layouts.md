---
title: Mastering layout in SwiftUI. Conditional Layouts.
layout: post
---

From the first day of the SwiftUI framework, we have primary layout containers like VStack, HStack, and ZStack. The current iteration of the SwiftUI framework brings us another layout container allowing us to place views in a grid. But the most important addition was the Layout protocol that all layout containers conform to. It also allows us to build our super-custom layout containers from scratch. This week we will learn the basics of the Layout protocol in SwiftUI and what benefits it gives us.

#### Basics
The way layout works in SwiftUI was always a hidden gem because of private APIs that Apple doesn't show us. Nowadays, we have the Layout protocol that expands the magical world of layout calculations in SwiftUI. 

=====================================================

The Layout protocol has two functions to implement. The first one calculates the final size of the layout with all children inside. And the second one places children according to your layout logic.

The current iteration of SwiftUI introduces new layout primitives conforming to the new Layout protocol. Now we have HStackLayout instead of HStack, VStackLayout instead of VStack, ZStackLayout instead of ZStack, and GridLayout instead of Grid. 

=====================================================

As you can see in the example above, the usage of the VStackLayout is the same as the VStack. You can replace your VStacks with VStackLayout if you support only the latest platform versions. But keep in mind that Apple will not remove VStack, HStack, and ZStack anytime soon. Instead, it recommends we use new layout primitives only when we need conditional layouts.

#### Conditional Layouts
To understand what conditional layout is, let's take a look at the small example.

=====================================================

As you can see in the example above, we display our views conditionally in a horizontal or vertical stack. The logic depends on the horizontal size class. Nothing is wrong with the code above, but there is a hidden issue. Whenever the size class changes, the SwiftUI framework recreates the views inside the if statement. 

This is how conditions work in ViewBuilders. While SwiftUI destroys your views, it also clears all the state of destroyed views, which might not be suitable for your app's user experience. It happens because the structural identity of your view changes while switching from HStack to VStack.

> I highly encourage you to read my "Structural identity in SwiftUI" post to understand better how SwiftUI identifies your views and the way conditions work in SwiftUI.

SwiftUI provides a new way to keep the structural identity of our view hierarchy while changing the layout container using the new AnyLayout type.

=====================================================

We use the new AnyLayout type to erase the actual type of the layout that depends on the current horizontal size class. The structural identity of our view stays the same. In this case, the SwiftUI doesn't recreate the views. It only moves them according to the new layout, and this transition can be easily animated.

=====================================================

#### Conclusion
Today we learned about the new Layout protocol and the type-eraser AnyLayout type. Now we can build more fluid transitions in our apps using these new tools.


