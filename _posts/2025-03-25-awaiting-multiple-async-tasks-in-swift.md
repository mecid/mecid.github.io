---
title: Awaiting multiple async tasks in Swift
layout: post
---

A few weeks ago, we discussed Task Groups in Swift, which is an explicit way of executing multiple concurrent tasks and waiting for them to complete. This week, we’ll delve deeper into the topic by exploring the async-let syntax in Swift, which offers a convenient way to work with Task Groups implicitly.

Imagine you have two asynchronous tasks that you need to wait for simultaneously and process their results together.

======================================================

As you can see in the provided example, we employ the await keyword twice, which implies that the execution will pause twice to await each result. In other words, we await the first result, retrieve it, then await the second result, retrieve it, and finally print the sum.

The first thing that might come to mind is why we don’t simultaneously wait for both results and print their sum as soon as both are available. Fortunately, Swift Concurrency’s Task Groups feature allows us to achieve this.

======================================================

Here, we create a task group with two children, enabling us to execute tasks A and B concurrently and accumulate their results. However, the code provided above appears overly complex for such a straightforward task. Task Groups are particularly useful for constructing asynchronous workflows with numerous tasks. Nevertheless, for simple tasks like this where we need to wait for two tasks to complete, the code seems overly cumbersome.

Fortunately, Swift introduced syntactic sugar over Task Groups API, allowing us easily run predefined amount of tasks using async-let syntax.

======================================================

As illustrated in the provided example, we employ the async-let syntax to define variables initialized through asynchronous calls to the taskA and taskB functions. 

Notably, we avoid using the await keyword before invoking the taskA and taskB functions. This is because the async-let syntax facilitates the lazy initialization of values as soon as the asynchronous task returns its result. In this case, both tasks run concurrently in parallel, allowing us to improve the performance of our code.

We still need to wait for the results of tasks because it can take some time to complete asynchronous tasks. Therefore, we should only use the await keyword whenever accessing the a and b variables.

