---
title: Thread safety in Swift with locks
layout: post
---

Today, we will discuss thread safety, an essential programming aspect. I decided to cover this topic because of the issue I've noticed in the codebase I helped to build. This type of bug is straightforward to create but very hard to fix. So investing time into building a type-safe type in your codebase is much better.

Let's look at a simple example of the state management concept.

=====================================================

The Store type looks pretty simple. Is there a condition where this type can crash your app? Yes, this type may lead to data races and race conditions where the first one crashes your app, and the second may break the logic you use.

Assume that you share a Store instance between different threads in your app, and whenever one thread tries to read the state variable while another thread is writing it crashes.

We can control the concurrent access to the state variable using a lock mechanism to solve the issue. Apple provides us with a brand new OSAllocatedUnfairLock type from iOS 16. It allows us to lock the access to the variable while we read or write it to guarantee exclusive access.

=====================================================

As you can see in the example above, we wrap our state property with an instance of the OSAllocatedUnfairLock type. Whenever we need to read or write the state property, we cover the logic with the withLock function. It automatically locks and releases at the end of the closure we provide.

OSAllocatedUnfairLock is very fast, but it doesn't support recursive locking. It crashes whenever you lock it twice from the same thread, so you should be careful while using it and not call it recursively. Instead, you can use the NSRecursiveLock type, allowing the same thread to lock recursively, but this implementation is a bit slower than OSAllocatedUnfairLock.

=====================================================

Finally, we can mark our Store type with the Sendable protocol, which means you can share an instance of the type between different threads, and it implements an internal synchronization mechanism.

=====================================================

We use the @unchecked attribute to turn off compiler checks on Sendable conformance because, in our case, the Store type doesn't rely on other Sendable types and instead implements internal synchronization.

Finally, we can safely share an instance of the Store type between different threads and never worry about strange crashes. You should always make your classes thread-safe whenever possible to use them in the multithreaded environment, even accidentally. Invest earlier and save your time in the future.

Today we learned how to use the NSRecursiveLock and OSAllocatedUnfairLock types to make any class thread safe. Happy multithreading!
