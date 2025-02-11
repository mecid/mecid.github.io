---
title: Task Cancellation in Swift Concurrency
layout: post
---

Swift Concurrency provides a cooperative cancellation model to handle task cancellation. This week, we will learn what a cooperative cancellation model is, how to use it, and how to be a good citizen and handle it correctly.

Cooperative cancellation means that Swift will never stop your task automatically, but it will provide you with information about the cancellation. It is totally up to you to decide how to handle this information.

Swift Concurrency provides us with the Task API, which we can use to understand when the task is cancelled by the caller. The caller can’t stop the task in any way; it can only mark it as cancelled. It is our responsibility to take care of cancellation and decide how to stop the execution. We can return from the function with an empty result or deliver partial results. But it is totally up to us how to handle the case.

======================================================

As you can see in the example above, we use the task view modifier with the id parameter, which means it cancels the previous task as soon as the id changes and creates a new one. 

I will rephrase it to make it more obvious: as soon as the id changes, SwiftUI marks the task as cancelled. It only marks the task as cancelled but doesn’t stop the execution. So, on every new symbol in the search field, SwiftUI starts a task and marks the previous one as cancelled.

======================================================

Here is the example of the Store type having an async search function. Inside the search function, we make an async network request. After that, we call the checkCancellation function on the Task type. The checkCancellation function is pretty simple; it throws an error whenever the task is already cancelled, and that way, we can stop the execution of the search function.

In our example, we just catch the error and clean the results variable. In more complex cases, you might have multiple sequenced async calls, and the checkCancellation function can save you from doing unnecessary work.

There is another option allowing us to check cancellation without throwing an error. The Task type provides us with the isCancelled property, which is a boolean value indicating whenever the task is cancelled.


======================================================

The isCancelled property provides you information about task status, you can check it whenever needed to decide how to model your next steps.

======================================================

Usually, you don’t need to manually cancel a task using Swift Concurrency, and it handles it for you. In some cases, you might need unstructured tasks. You can create them using the Task type, and it also provides us with the cancel function, allowing us to mark the task as cancelled.

