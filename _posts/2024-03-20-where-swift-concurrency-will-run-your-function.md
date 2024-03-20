---
title: Where Swift Concurrency will run your function?
layout: post
---

Apple released Swift 5.5 almost three years ago. The main addition to the release was the Swift Concurrency feature. It introduced async and await keywords, allowing us to build concurrent apps in a new way. This week, we will learn how Swift determines where to run your function in a concurrent environment.

First, let's look at creating an async function in Swift. To do so, you simply need to add the async keyword to the function's definition.

```swift
func foo() async {
    // ...
}
```

The async keyword in the function's definition means that the function may suspend its execution to switch threads. To run an async function, you have to use the await keyword.

```swift
await foo()
```

The await keyword allows the calling thread to wait while an async function performs its job. When the async function finishes, the calling thread resumes where it is suspended.

The Swift language introduces the Cooperative Thread Pool, which allows you to run concurrent parts of your apps. The number of threads in the pool is limited to your CPU's cores, which prevents thread explosion.

So, we have two crucial places where Swift Concurrency can run our code: Main Thread and Cooperative Thread Pool. We should use the main thread to update the UI of our apps, but we should avoid blocking it by running heavy work on it. That's why we have the Cooperative Thread Pool to run heavy jobs.

The next step is always to be sure where the Swift language will run your code. The Swift language uses a few rules to determine where to run your code.

If your function is isolated to an actor, it will run as part of that actor. It doesn't matter if it is an async function or not. Actors always run on the Cooperative Thread Pool, and all their functions are isolated. You can also isolate any type or function you need using global actors.

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

As you can see in the example above, we have an actor-isolated Store type. It doesn't matter where you call foo or boo functions. They will always run on the main thread because the Store type is isolated to the global @MainActor.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello")
            .task {
                boo()
            }
            .task {
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

Here, we have a more complex example confusing many developers in our community. You should remember that SwiftUI views are not isolated to any actor. Only the body property of the View protocol is isolated to the main actor.

So, we have two non-isolated functions here. The foo function is async, and Swift runs it in the cooperative thread pool. The boo function is not async, and Swift will run it on the calling thread. As I said before, the body property of the View protocol is isolated to the main actor, which means in this particular example boo function will run on the main thread, where you should avoid doing heavy work.

```swift
struct ContentView: View {
    var body: some View {
        content
    }
    
    private var content: some View {
        Text("Hello")
            .task {
                boo()
            }
            .task {
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

I've slightly changed the example by introducing the content property on the ContentView type. The content property isn't isolated to the main actor, so both functions will run on the cooperative thread pool.
 
I hope this post will make running async functions less confusing.
