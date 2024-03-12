---
title: Global actors in Swift
layout: post
---

The Swift language allows us to define thread-safe types using actors. Actor type automatically manages serial access to the data it protects. But what if we must have multiple types protected by a single serial access queue? That's why we have global actors, and today, we will learn how to use global actors in Swift.

The main thread rendering is the best example of why we need to protect multiple types with a single serial access queue. You may have a massive collection of UIViewControllers, UIViews, or SwiftUI views running concurrently, but in the end, you should update your user interface on the serial main thread. 

That's why Swift provides us @MainActor. Any UIViewController or UIView you create inherits @MainActor access from its definition. SwiftUI's View protocol also defines its body property with @MainActor. This means your view's body, view, or controller always runs on the main thread and protects you from accidentally updating the user interface from the background thread.

To fully understand the idea of the global actors, let's inspect the @MainActor a bit further.

=====================================================

As you can see in the code example above, the MainActor type is defined with actor keyword and conforms to the GlobalActor protocol. It also has the @globalActor attribute. The GlobalActor protocol requires you to specify the shared property to create a shared, also called a global instance of the actor. 

=====================================================

Now, we can easily mark any type we need with the @MainActor attribute to isolate it to the main actor. This means all the work in the particular type runs serially on the main actor.	

Let's move forward and build our own global actor. Assume that you have many types of defining access to the local storage and want to keep them conflict-free and run in a serial order.

=====================================================

As you can see in the example above, we define the StorageActor type conforming to the GlobalActor protocol using the actor keyword. The @globalActor attribute allows us to mark any type, function, or property with the @StorageActor.

=====================================================

Here, we create Сache and Database types using the @StorageActor attribute. It allows us to run them on a single serial queue managed by the StorageActor we created before. 

Why do we use global actors rather than defining Cache and Database types as actors? We can define Cache and Database as actors. Still, in this case, every instance of the Cache or Database types will run on an independent serial queue and protect its access alone. By marking our types with the @StorageActor, we belong them to a single serial queue managed by the shared instance of the StorageActor.

=====================================================

Remember that you can mark with the @StorageActor attribute not only types but also functions or properties of any type.

Today, we learned why and how to use global actors in Swift. You don't need to use global actors often in your apps. However, in particular cases, such as main thread rendering, they become handy.
