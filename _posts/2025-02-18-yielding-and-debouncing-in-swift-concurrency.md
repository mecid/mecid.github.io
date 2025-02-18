---
title: Yielding and debouncing in Swift Concurrency
layout: post
category: Swift Language Features
image: /public/swift.png
---

I decided to continue the topic of Swift Concurrency to cover some not-obvious things. This week we will talk about task yielding and debouncing. Swift concurrency language features provide us with two simple but very powerful functions: *yield* and *sleep*. We will try to learn how and when to use them.

{% include friends.html %}

What is the task debouncing? Assume that you have a search field doing heavy search on a huge data structure. While the user types a search query, you start a search task to display results for the entered search term.

```swift
@MainActor @Observable final class Store {
    private(set) var results: [HKCorrelation] = []
    private let store = HKHealthStore()
    
    func search(matching query: String) async {
        // heavy search
    }
}

struct ContentView: View {
    @State private var store = Store()
    @State private var query = ""
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.uuid) { result in
                Text(verbatim: result.endDate.formatted())
            }
            .searchable(text: $query)
            .task(id: query) {
                await store.search(matching: query)
            }
        }
    }
}
```

As you can see in the example above, we use the *task* view modifier with the *id* parameter. The code above starts a new task on every keystroke and cancels the previous one. As we know, the Swift concurrency language feature uses a cooperative cancellation model, which means it doesn’t halt your task; it only provides information about cancellation.

As soon as you run the example, you will notice that you run a task on every keystroke, and even worse, they can race with each other. Fortunately, there is a solution for this kind of situation called debouncing.

Debouncing is a simple technique allowing us to handle streaming data in a right way. Whenever the user types the word “apple”, we usually want to ignore terms: “a”, “ap”, “app”, “appl” and run the task on “apple”. 

Debouncing simply means that you don’t start the task immediately and wait for some amount of time. If the data changes, you don’t start a task and wait again until the data stays the same for some time. Then you can run your task. It allows us to eliminate unnecessary work for intermediate data and run only for final data.

```swift
@MainActor @Observable final class Store {
    private(set) var results: [HKCorrelation] = []
    private let store = HKHealthStore()
    
    func search(matching query: String) async {
        // heavy search
    }
}

struct ContentView: View {
    @State private var store = Store()
    @State private var query = ""
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.uuid) { result in
                Text(verbatim: result.endDate.formatted())
            }
            .searchable(text: $query)
            .task(id: query) {
                do {
                    try await Task.sleep(for: .seconds(1))
                    await store.search(matching: query)
                } catch {
                    // task is cancelled because of a new query or view disappearance
                }
            }
        }
    }
}
```

Swift concurrency doesn’t provide a particular function for debouncing tasks, but we can easily implement it using the *sleep* function on the *Task* type. All we need to do is to call the *sleep* function by providing some amount of time and then run our heavy job.

Whenever a task is cancelled while sleeping, it throws an error and interrupts the execution without running the heavy job. This way, you can reduce the amount of work you run to display the final results.

Let’s talk about task yielding. Assume that you have a non-async function call that can take a long time to run. For example, in one of my recent projects, I receive a collection of huge JSON files, then I should process them and save them on disk.

```swift
struct Item: Decodable {
    // decodable fields
}

struct DataHandler {
    func process(json files: [Data]) async throws -> [Item] {
        let decoder = JSONDecoder()
        var result: [Item] = []
        
        for file in files {
            let items = try decoder.decode([Item].self, from: file)
            result.append(contentsOf: items)
        }
        
        return result
    }
}
```

In the example above, I have the *DataHandler* type defining the *process* function. I made it asynchronous to run on the Cooperative Thread Pool provided by the Swift concurrency. Keep in mind that the Cooperative Thread Pool has a limited number of threads and its count is not so large.

While executing a long-running task that can take minutes to run, we completely block a thread on the Cooperative Thread Pool. Running a few similar tasks might be a real bottleneck, stopping other tasks from running on the Cooperative Thread Pool. For this particular case, Swift concurrency provides us with a task yielding API.

```swift
struct DataHandler {
    func process(json files: [Data]) async throws -> [Item] {
        let decoder = JSONDecoder()
        var result: [Item] = []
        
        for file in files {
            let items = try decoder.decode([Item].self, from: file)
            result.append(contentsOf: items)
            
            await Task.yield()
        }
        
        return result
    }
}
```

As you can see in the example above, we use the *yield* function after every decoded JSON file. The yield function allows us to suspend the execution and return the thread to the pool to run other asynchronous tasks. After that, it resumes execution of our task. This way, we can fairly distribute the computing power between parallel tasks. 

Today we learned how to be a good citizen while using Swift concurrency language features by utilizing yielding and debouncing. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
