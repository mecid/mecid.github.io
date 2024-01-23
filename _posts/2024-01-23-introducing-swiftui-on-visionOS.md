---
title: Introducing SwiftUI on visionOS
layout: post
image: /public/visionOS.webp
---

Apple Vision Pro is coming soon, and it is the perfect time to look at SwiftUI API, which allows us to adapt our apps to the immersive world that visionOS provides us. Apple states that the best way to build an app is with Swift and SwiftUI. This week, we will learn how to use SwiftUI to build a visionOS app.

#### Windows
What I love about SwiftUI is how it adapts automatically to the platform. You don't need to do anything to run your app written in SwiftUI on visionOS. It works out of the box. But you can always improve the user experience by going forward and adapting the platform features.

=====================================================

In the example above, we use the new toolbar placement called bottomOrnament. Ornament in visionOS is the place outside the window presenting controls connected to the window. You can also create them manually by using the new ornament view modifier.

=====================================================

The new ornament view modifier allows us to create an ornament with a particular anchor point for the window it is connected to. Another option to adapt your app content to the immersive experience that visionOS provides is to use the transform3DEffect and rotation3DEffect view modifiers to incorporate depth effects.

#### Volumes
Your apps can display 2D and 3D content side by side in the same scene on visionOS. We can use the RealityKit framework to present 3D content in this case. For example, RealityKit provides us with the Model3D SwiftUI view, allowing us to display 3D models from the USDZ or reality files.

=====================================================

Model3D view works similarly to the AsyncImage view and loads the model asynchronously. You can also use another variant of the Model3D initializer, which allows you to customize the model configuration and add a placeholder view.

=====================================================

While presenting 3D content in your app, you can use the windowStyle modifier to enable volumetric display of your content. The volumetric style allows your content to grow in the third dimension to match the model's size.

For more complex 3D scenes, we can use the RealityView and populate it with 3D content.

=====================================================

#### Immersive spaces
The third option on visionOS is the fully immersive experience, allowing us to dive into the 3D scene by hiding everything around by focusing on your scene.

=====================================================

As you can see in the example above, we define a scene by using the ImmersiveSpace type. It allows us to enable it by using the openImmersiveSpace environment value.

=====================================================

We can also use the dismissImmersiveSpace environment value to dismiss the immersive space. Remember that you can only display one immersive space at a time.

#### Conclusion
Today, we learned the basics of the SwiftUI framework for the brand new visionOS platform.
