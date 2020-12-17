---
title: Hero animations in SwiftUI
layout: post
image: /public/swiftui.png
category: Interactions
---

Animation is one of the powerful features of SwiftUI. I was shocked when I saw how easy we could animate changes in view hierarchy by simply mutating @State properties and attaching animation modifiers. This week we will talk about another animation type called hero animations. We will learn how we can implement hero animations using the new matchedGeometryEffect view modifier.

Assume that we are working on an app that shows a grid of images. You can select an image by tapping on it. On the bottom of the screen, we have another grid that shows only selected images. Let's start implementing this app example.

==============================================

Here we have a view that defines the list of images and the empty list of selected images. We also structured our view's body to place the list of available images on the top and the list of selected images on the bottom of the screen. Let's move forward and implement a grid that displays our images.

====================================================

As you can see in the example above, we have a grid with a single adaptive column displaying squared images of 44pt. We also attach a tap gesture to every image that removes the image from the list and moves it to the selected image list. We wrap this mutation using the withAnimation function, which animates this change.

> To learn more about grids, look at my "Mastering grids in SwiftUI" post.

====================================================

Here is the source code of the selected images grid, which looks very similar to the previous one. There are two differences. The first one is the size of the images. Here we use 88pt instead of 44pt. The second difference is the tap gesture. In this case, we move an image from the list of the selected image to the all images list.

By default, SwiftUI uses fade-in and fade-out transitions to animate layout changes. For example, when you remove a view from a view hierarchy, SwiftUI uses a fade-out transition. You can change this behavior by adding a transition modifier to the view and providing another transition.

> If you are not familiar with transitions in SwiftUI, take a look at my "Animations in SwiftUI" post.

![fading-animation](/public/hero1.GIF)

As you can see here, SwiftUI removes the image you tap using fade-out transition and adds it to another grid using fade-in transition. Now, it is time to talk about hero animations.

Hero animation is a special effect in motion pictures and animations that changes one image into another through a seamless transition. For example, we want to achieve a morphing animation while moving an image from top to bottom by applying the scaling transformation.

Fortunately, SwiftUI provides us a special view modifier called matchedGeometryEffect to implement hero animations easily. By attaching matchedGeometryEffect to multiple views, we define a connection between them. SwiftUI can use this connection to understand the geometry of transition and automatically apply shape, position, and size transformation between these changes.

=====================================================

If inserting a view in the same transaction that another view with the same identifier is removed, the system will interpolate their frame rectangles in window space to make it appear that a single view moves from its old position to its new position. Remember that you should use unique identifiers for every view that applies a matched geometry effect.

![hero-animation](/public/hero2.GIF)

#### Conclusion
Today we learned about implementing hero animations in SwiftUI using the matchedGeometryEffect view modifier. I love how easy we can achieve this effect in SwiftUI. Unfortunately, it doesn't work between different views inside navigation links, but I hope to see this working during the next iteration of SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!


