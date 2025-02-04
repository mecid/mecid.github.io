---
title: Mastering TaskGroups in Swift
layout: post
---

Structured Swift Concurrency makes our lives much easier by introducing task groups. Task groups are a way to run a dynamic number of child tasks, await all of them, or cancel. This week, we will learn how to use and optimize TaskGroups in Swift.

{% include friends.html %}

Let’s begin with a fundamental example to illustrate the usage of task groups in Swift.

```swift
@MainActor @Observable final class Store {
    private(set) var messages: [String] = []
    
    func fetch(urls: [URL]) async {
        messages = await withTaskGroup(
            of: String.self,
            returning: [String].self
        ) { group in
            for url in urls {
                group.addTask(priority: .high) {
                    do {
                        let (data, _) = try await URLSession.shared.data(from: url)
                        return String(decoding: data, as: UTF8.self)
                    } catch {
                        return ""
                    }
                }
            }
            
            var messages: [String] = []
            
            for await message in group where !message.isEmpty {
                messages.append(message)
            }
            
            return messages
        }
    }
}
```

As you can see in the example above, we use the withTaskGroup function to create a task group. It takes a few parameters, where the first is the result type of the every child task in the group. The second parameter is the type of the accumulated result. The third is the closure where we can populate the task group with child tasks.

The closure provides us with an instance of the TaskGroup type conforming to the AsyncSequence protocol, allowing us to await the results of the task group in the for loop.

We use the add function on the TaskGroup to add child tasks. We also can add the tasks with particular priorities. Keep in mind that the task group doesn’t inherit the actor from the parent and runs the task group on the Cooperative Thread Pool.

Task groups are part of the Structured Swift Concurrency, which means they use cooperative cancellation, and it is up to you to check the Task.isCancelled property and decide what to do next when task is cancelled.

```swift
@MainActor @Observable final class Store {
    private(set) var messages: [String] = []
    
    func fetch(urls: [URL]) async {
        messages = await withTaskGroup(
            of: String.self,
            returning: [String].self
        ) { group in
            for url in urls {
                group.addTask {
                    do {
                        if Task.isCancelled {
                            return ""
                        } else {
                            let (data, _) = try await URLSession.shared.data(from: url)
                            return String(decoding: data, as: UTF8.self)
                        }
                    } catch {
                        return ""
                    }
                }
            }
            
            var messages: [String] = []
            
            for await message in group where !message.isEmpty {
                messages.append(message)
                
                if messages.count > 3 {
                    group.cancelAll()
                }
            }
            
            return messages
        }
    }
}
```

As you can see, we check the Task.isCancelled property, and if the task is cancelled, we return an empty string instead of doing the network request. In the example above, we cancelled the group itself, but it also can be cancelled whenever the parent task is cancelled.

Let’s talk a bit about how we can optimize task groups in Swift. As I said before, task groups run on the Cooperative Thread Pool, which means it doesn’t matter how many child tasks you add; they will run on the limited amount of threads. And it is great because our goal is to speed up our app logic and not slow it down because of too many threads.

```swift
@MainActor @Observable final class Store {
    private(set) var messages: [String] = []
    
    func fetch(urls: [URL]) async {
        messages = await withTaskGroup(
            of: String.self,
            returning: [String].self
        ) { group in
            // 1000 urls are here
            for url in urls {
                group.addTask {
                    do {
                        try Task.checkCancellation()
                        let (data, _) = try await URLSession.shared.data(from: url)
                        return String(decoding: data, as: UTF8.self)
                    } catch {
                        return ""
                    }
                }
            }
            
            var messages: [String] = []
            
            for await message in group where !message.isEmpty {
                messages.append(message)
            }
            
            return messages
        }
    }
}
```

Let’s take a look at the example above. We add 1000 child tasks to the group that should fetch the data from the URLs. We check the cooperative cancellation property and return an empty string whenever the task group is cancelled.

It looks like there is nothing wrong with the code, but even if it doesn’t run the 1000 tasks at the same time, it allocates the memory for 1000 tasks and suspends. Because the Cooperative Thread Pool can’t run them at the moment, they await the pool. The code above may lead to high memory consumption, and we can optimize it by adding tasks only when a few of them have already finished the work.

```swift
@MainActor @Observable final class Store {
    private(set) var messages: [String] = []
   
    func fetch(urls: [URL]) async {
        messages = await withTaskGroup(
            of: String.self,
            returning: [String].self
        ) { group in
            let maxConcurrentRequests = min(urls.count, 10)
            var index = 0
            
            // 1000 urls are here
            for _ in 0..<maxConcurrentRequests {
                group.addTask {
                    do {
                        try Task.checkCancellation()
                        let (data, _) = try await URLSession.shared.data(from: urls[index])
                        
                        index += 1
                        
                        return String(decoding: data, as: UTF8.self)
                    } catch {
                        return ""
                    }
                }
            }
            
            var messages: [String] = []
            
            for await message in group where !message.isEmpty {
                messages.append(message)
                
                if index < urls.count {
                    group.addTaskUnlessCancelled {
                        let nextURL = urls[index]
                        // fetch next url
                    }
                }
            }
            
            return messages
        }
    }
}
```

Here we set a limit of 10 child tasks and add the initial portion of the child tasks. Then we add tasks one by one after the previous task finishes its work. In this way, we can significantly improve the memory consumption of our code.

Another nice API, that task groups provide us, is the addTaskUnlessCancelled function, which adds a task only if the task group is not cancelled. We can use this function to optimize task creation and avoid running new tasks when a task group is cancelled.

Swift Concurrency features are back-deployed to iOS 13 and macOS 10.15, which means there is no reason to ignore them until you support older platform versions. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
