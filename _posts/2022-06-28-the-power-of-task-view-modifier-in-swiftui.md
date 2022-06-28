---
title: The power of task view modifier in SwiftUI
layout: post
---

Task view modifier is the key to the Swift Concurrency world through SwiftUI. It allows us to build complex async tasks by leveraging the power of cooperative cancellation and the lifecycle of a SwiftUI view. This week we will learn all the powerful features of the task view modifier in SwiftUI.

==================================

The task view modifier starts the unstructured async task and binds it to the view lifecycle. SwiftUI automatically cancels ongoing tasks whenever the view disappears by propagating cooperative cancellation.

==================================

As you can see in the example above, we pause the task by using the sleep function that throws CancellationError whenever a task is canceled during the sleep. Alternatively, you can use the isCancelled property on the Task type to indicate whether or not the current task is canceled.

By default, the task view modifier uses the user-initiated (highest) priority for the created task, but you can also use lower preferences like utility or background. 

==================================

Another variant of the task view modifier allows us to observe equitable data and run the async task whenever the data changes. The task lifecycle is still bound to the view lifecycle, but SwiftUI also cancels the ongoing job whenever data changes and creates a new one for the latest data.

==================================

In the example above, whenever the user types the query in the search bar SwiftUI creates a task. SwiftUI makes a task for every change in the search query in this case. Usually, we want to debounce requests to our servers and make them after a slight pause. We can quickly achieve this effect by leveraging the power of the cooperative cancellation and data observing capabilities of the task view modifier.

===================================================

Here we try to sleep for a bit and make a network query only if the user doesn't type a new query. In another case, SwiftUI cancels the task and creates a new one.

Debouncing via task view modifier becomes very handy in my projects. That's why I created a small wrapper around the task view modifier, allowing us to debounce tasks without this boilerplate.

===================================================

Today we learned how to use the task view modifier and the different opportunities it provides to handle complex async flows.
