---
title: Mastering NavigationStack in SwiftUI. Deep Linking.
layout: post
---

This week we will continue exploring the new Navigation API in SwiftUI. One of the benefits of the new data-driven Navigation API is the programmatic navigation with deep-linking possibilities. Let's dive into the new API by learning how to build programmatic deep navigation flows.

> To learn about the basics of the new data-driven Navigation API in SwiftUI, look at my "Mastering NavigationStack in SwiftUI. Navigator Pattern." post.

#### Programmatic navigation
There is a special NavigationStack initializer accepting a binding to a mutable collection. SwiftUI maps values of the mutable collection into a view hierarchy and allows us to push and pop views into the NavigationStack programmatically. Let's take a look at the example.

=====================================================

As you can see in the example above, we define a piece of state that drives our navigation via binding. We also display a button that adds another product to the path. SwiftUI automatically maps the contents of the path binding to the view hierarchy in the navigation stack and automatically removes the product from the path whenever the user presses the back button.

=====================================================

We also can quickly implement pop to the root view by removing all the items from the path. With this new data-driven approach, SwiftUI is responsible for keeping your navigation stack in sync with the path binding.

Usually, our navigation stack contains different screens representing different values. In this case, we can define the path as an array of enum where every case specifies a particular destination in our app. 

=====================================================


#### Programmatic navigation with multiple scenes
One thing I have to mention is that you never should define the path in the App protocol. In this case, you will have a synchronized navigation stack across all of the scenes of your app. Usually, users create multiple scenes of our apps to use different parts of our apps simultaneously.

=====================================================

Instead, we should hold our state defining the path in the root view of our app or inside a container view representing a particular user flow.

=====================================================

#### Deep linking and handoff
Let's move to the next thing we might need in our app: deep linking and handoff. With the new data-driven Navigation API, we can quickly implement deep linking and handoff support. All we need to do is handle the URL and add the corresponding value to the array specifying the path of the navigation stack.

=====================================================

#### State restoration
State restoration is one of the essential features that you should implement to provide a pleasant user experience. SwiftUI provides the SceneStorage property wrapper, allowing us to keep data in the specific storage bound to the scene and survive when the system shuts down the app.

> To learn more about state restoration in SwiftUI, look at my "State restoration in SwiftUI" post.

We can use the SceneStorage property wrapper to encode our navigation path and store it in the scene memory. Whenever the system kills the app, we can restore the path from the scene storage and programmatically navigate to the last entry.

=====================================================

Here we have the NavigationStore class providing the common functionality for deep linking and handoff support. It also allows us to encode our path and decode it from the serialized representation. Now we can use it in our root view for state restoration whenever needed.

=====================================================

As you can see in the example above, we use the task view modifier to restore the navigation from the scene storage and observe the navigation path to save it as soon as any changes appear.

#### Conclusion
Today we learned how to use the new data-driven Navigation API to control our navigation stack programmatically and implement the deep linking feature.
