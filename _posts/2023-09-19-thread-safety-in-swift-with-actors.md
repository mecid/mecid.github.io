---
title: Thread safety in Swift with actors
layout: post
---

Actors is the new Swift language feature, making your types thread-safe. This week, we will learn how to use actors and their benefits over locks. We will also discuss actor reentrancy, the main confusing point of using actors.

In the previous post, we modeled a Store type, allowing us to implement state management predictably.

=====================================================

At this point, the Store type is not thread-safe, and using it from multiple threads may lead to data races and race conditions. We solve the issue by using an instance of the NSRecursiveLock type.

=====================================================

As you can see in the example above, we use an instance of the NSRecursiveLock type whenever we access the protected part of the store. We made our store thread-safe, but let me tell you about a few downsides of using locks.

First, you must wrap every use of the state property with the withLock function. Whenever you miss it accidentally or not, the Store type is not thread-safe anymore. So, you should be very careful while using locks.

Second, when you call the lock or withLock functions, they completely hang the current thread until the lock is released, which may lead to performance issues when many threads access the protected value.

Swift language introduced a feature called Actors to solve these complex issues. Actors like classes are reference types but protect their stored properties from multithread access.

=====================================================

As you can see in the example above, we define our Store type by using actor keyword instead of class. That's all you need to make your Store thread-safe. Actor isolation guarantees exclusive access to the stored fields of an actor type. You must use the await keyword to access or function on an actor type.

====================================================

The await keyword is part of the Swift Concurrency feature, allowing us to await the results whenever we switch threads. In the example above, we use await on every touch to the actor because another thread might use the actor, and we must wait for our exclusive access. Swift Compiler guarantees exclusive access, and we have compiler time verification on thread safety.

====================================================

Now let's talk about actor reentrancy. What if we run async code on an actor? In this case, the actor suspends execution and switches threads to run an async function outside the actor. During this time, the actor allows other threads access its isolated properties and functions because it doesn't run the async code itself. When the async code finishes, the actor switches threads back to run actor-isolated code.

Remember that every use of the await keyword inside an actor type is a possible suspension point where other threads may access or mutate actor-isolated properties. This situation is called actor reentrancy. You may have race conditions during actor reentrancy if you assume that actors always run code atomically.
