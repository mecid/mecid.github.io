---
title: Dependency container on top of task local values in Swift
layout: post
---

Task local values is the recent addition to the concurrency feature of the Swift language. This week, we will not only learn the basics of task local values, but also discuss the interesting usage where we will build the dependency injection container using this language feature.

Task local values is the new way to create a Task shared value. It is implicitly shared across child tasks and accessible both from the sync and async context.

======================================================

As you can see in the example above, we define the Request type with unique identifier. We also create an extension for the Request type that uses @TaskLocal macro to define a static property for the current Request instance. We always should provide a default value for the task local values or make it optional.

You can use @TaskLocal macro only with static properties because the main goal is to create a shared instance of the type implicitly available for the async tasks. It works very similar to the environment feature of SwiftUI allowing you to implicitly carry on data down to the view hierarchy.

======================================================

Task local values are read-only when you try to access it directly. You can modify a task local value using the withValue function. The updated value will be available in the scope of the provided closure and implicitly shared across the async context of the current Task.

You are not going to use task local values very often, but you can use them whenever you need to propagate some piece of state down into the hierarchy of an async task. In our example, we share a request to make logging of the request identifier easier in the world of the concurrent tasks.

All of these inspired me to use the task local values for implicit dependency injection. We can create a dependency container holding services that we might need to replace for mocked versions. Let’s see how we can achieve it.

======================================================

As you can see in the example above, we define the Dependencies type holding the fetching statistics function. In the real world, you would have there much more service functions. We also define the production-ready and mocked version of our service function.

======================================================

Here we define the task local value for the active dependency container. By default we use the production-ready version of the dependency container. For the purpose of testing we can replace the production-ready container with mocked version implicitly using the withValue function.

======================================================

Inside the test target we want to avoid accessing the real APIs and use the mocked version to make our tests less flaky.

Today, we learned the basics of the task local values and discussed how would we engage with it even more by building a dependency container using this feature.
