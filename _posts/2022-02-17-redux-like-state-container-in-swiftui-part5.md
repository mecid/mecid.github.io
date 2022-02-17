---
title: Redux-like state container in SwiftUI. Swift concurrency model.
layout: post
---

Over the last two years, I have actively used unidirectional flow to develop my apps. I covered the approach I use in the series of posts about building Redux-like state containers. This week I want to share with you how this approach adapts to the latest changes in Swift by applying the new concurrency model.

Swift concurrency is the new language feature to write concurrent code straightforwardly. I'm not going to cover this in detail and assume that you know how to use it. If you are not familiar with the Swift Concurrency model, I suggest starting with the official docs.

=====================================================

As you can see in the previous versions, I have been using the Combine framework heavily to do asynchronous work. But now I'm switching to the new Swift concurrency model. And I am pleased to see that the new concurrency model plays very well with the Combine framework, and we can mix them to don't break the working code in our apps and migrate step by step.

=====================================================

Here we pushed a few changes to support the new concurrency model:
We made our send method an async by adding the particular keyword to the method definition.
We use the values property on the Combine framework's Publisher type to convert it into an AsyncSequence.
We use for await keyword to iterate over values of AsyncSequence and apply them to the store.

We also use where keyword to support cooperative task cancellation. As you can see, we made very few changes to migrate to the new concurrency model and gain new features like task cancellation out of the box. We don't need to change anything else, and we still can use our old reducers with the Combine framework because they work together very nicely.

=====================================================

Let's go forward and replace the dependencies of our reducers with async closures. We still must return the Combine framework's Publisher type, but we need somehow to wrap an async closure call with the Combine publisher. Unfortunately, there is no way to do it automatically, but we can create the Publisher type that does it for us.

=====================================================

Let me introduce the SendablePublisher type that allows us to convert any async closure to the Combine publisher. Please look at how we use the handleEvents operator to implement the support for the cooperative cancellation. Now we can refactor our reducer to use async closures for side effects.

=====================================================

SwiftUI provides us with the task view modifier accepting an async closure. SwiftUI automatically runs past closure when the view appears and tracks its lifecycle. SwiftUI cancels the task created for an async closure whenever the view disappears, and if you support the cooperative cancellation model, your task automatically stops.

=====================================================

This week we learned how to migrate our codebase from the Combine framework to the new Swift concurrency model using small steps. I love the way they play together, but I believe that the new Swift concurrency model is the future of async tasks. I hope to eliminate the Combine framework whenever the AsyncSequence type gains all the needed transformation operators offered by the Combine framework's Publisher type.
