---
title: Where Swift Concurrency will run your function?
layout: post
image: /public/swift.png
category: Swift Language Features
---

Apple released Swift 5.5 almost three years ago. The main addition to the release was the Swift Concurrency feature. It introduced **async** and **await** keywords, allowing us to build concurrent apps in a new way. This week, we will learn how Swift determines where to run your function in a concurrent environment.

{% include friends.html %}

First, let's look at creating an async function in Swift. To do so, you simply need to add the **async** keyword to the function's definition.

```swift
func foo() async {
    // ...
}
```

The *async* keyword in the function's definition means that the function may suspend its execution to switch threads. To run an async function, you have to use the **await** keyword.

```swift
await foo()
```

The **await** keyword allows the calling thread to wait while an async function performs its job. When the async function finishes, the calling thread resumes where it is suspended.

The Swift language introduces the Cooperative Thread Pool, which allows you to run concurrent parts of your apps. The number of threads in the pool is limited to your CPU's cores, which prevents thread explosion.

So, we have two crucial places where Swift can run our code: Main Thread and Cooperative Thread Pool. We should use the main thread to update the UI of our apps and we should avoid blocking it by running heavy work on it. That's why we have the Cooperative Thread Pool to run heavy jobs.

The next step is always to be sure where the Swift language will run your code. The Swift language uses a few rules to determine where to run your code.

If your function is isolated to an actor, it will run as part of that actor. It doesn't matter if it is an async function or not. Almost all actors run on the Cooperative Thread Pool. The main actor is exception, because it runs on the main thread. You can also isolate any type or function you need using global actors.

> To learn more about global actors, take a look at my ["Global actors in Swift"](https://swiftwithmajid.com/2024/03/12/global-actors-in-swift/) post.

Swift applies the second rule if your function isn't isolated to an actor. The Swift language runs your function on the Cooperative Thread Pool whenever your function is async. On the other hand, non-async functions run as part of the calling thread, which means they don't switch threads and will run where you call them.

Let's dive into some examples.

```swift
@MainActor final class Store {
    var messages: [String] = []
    
    func boo() {
        messages = ["boo"]
    }
    
    
    func foo() async {
        messages = ["foo"]
    }
}
```

As you can see in the example above, we have an actor-isolated *Store* type. It doesn't matter where you call *foo* or *boo* functions. They will always run on the main thread because the *Store* type is isolated to the global *@MainActor*.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello")
            .task {
                // runs on the main thread
                boo()
            }
            .task {
                // runs on the cooperative thread pool
                await foo()
            }
    }
    
    func boo() {
        // ...
    }
    
    func foo() async {
        // ...
    }
}
```

Here, we have a more complex example confusing many developers in our community. You should remember that SwiftUI views are isolated to the *MainActor*. SwiftUI views inherits theirs isolation from the *View* protocol which is main actor isolated in its definition. That is why both *foo* and *boo* functions will run on the main thread.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello")
            .task {
                // runs on the main thread
                boo()
            }
            .task {
                // runs on the cooperative thread pool
                await foo()
            }
    }
    
    func boo() {
        // ...
    }
    
    nonisolated func foo() async {
        // ...
    }
}
```

I've slightly changed the example by adding *nonisolated* attribute to the *foo* function to escape its actor isolation. In this case, the *foo* function becomes nonisolated async function which means it will run on the Cooperative Thread Pool.
 
I hope this post will make running async functions less confusing. Feel free to follow me on Twitter and ask your questions related to this post. Thanks for reading, and see you next week!
