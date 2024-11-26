---
title: The power of previews in Xcode
layout: post
---

Previews in Xcode become more powerful every year. Previews in Xcode are not about SwiftUI; you can use them even with UIKit. This week, we will talk about enhancing Previewable and PreviewModifier types, allowing us to build reusable preview environments.

Let’s take a look at the situation where you have a SwiftUI view with a binding. To run a preview for views with binding requires additional boilerplate where you have to wrap the view with another view defining a state for your binding and then pass it to the previewing view.

======================================================

As you can see in the example above, we create the SleepDetailsPreview type only for the purpose of previewing the SleepDetailsView in Xcode. It simply clutters our codebase with yet another view type. Fortunately, Xcode 16 introduced the Previewable macro type.

The Previewable macro type allows us to inline the state definition into the Preview macro. It is specially designed to use inside of the Preview macro and use outside of Preview makes a compiler error.

======================================================

As you can see, we inline the State property into the Preview macro by adding the Previewable macro. It allows us to significantly reduce the code we need to run a preview in Xcode.

Another great addition to Xcode Previews is the PreviewModifier protocol. The PreviewModifier type allows us to create reusable preview environments that we can share across multiple previews. 

For example, we can create the MockDataPreviewModifier type that populates the Swift Data container with the mock data to display while using it in previews.


======================================================

Here we define the MockDataPreviewProvider type conforming to the PreviewProvider protocol. The PreviewProvider protocol has two requirements the makeSharedContext and body functions. 

In the makeSharedContext you can create and prepare anything you might need in your preview. For example, it might be a mock data-backed instance of the ModelContainer or any store type specific to your app logic.

In the body function, you have access to the previously created context. The body function is the place where you can apply your context. In our example, we use the modelContainer view modifier to set the model container for the view hierarchy. For the instance of a custom store type, you can use the environment view modifier to pass it in.

As you can see, we have to pass an instance of the MockDataPreviewProvider type to the Preview macro’s trait parameter to apply it.

The great thing about the PreviewModifier is that the Xcode Preview system caches instances returned from the makeSharedContext function. Which makes significant performance boost whenever you preview multiple instances with the same trait.
