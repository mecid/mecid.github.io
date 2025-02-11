---
title: Task Cancellation in Swift Concurrency
layout: post
---

Swift Concurrency provides a cooperative cancellation model to handle task cancellation. This week, we will learn what a cooperative cancellation model is, how to use it, and how to be a good citizen and handle it correctly.

Cooperative cancellation means that Swift will never stop your task automatically, but it will provide you with information about the cancellation. It is totally up to you to decide how to handle this information.

Swift Concurrency provides us with the Task API, which we can use to understand when the task is cancelled by the caller. The caller can’t stop the task in any way; it can only mark it as cancelled. It is our responsibility to take care of cancellation and decide how to stop the execution. We can return from the function with an empty result or deliver partial results. But it is totally up to us how to handle the case.

```swift
struct ContentView: View {
    @State private var store = Store()
    @State private var query = ""
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.self) { result in
                Text(verbatim: result)
            }
            .searchable(text: $query)
            .task(id: query) {
                await store.search(matching: query)
            }
        }
    }
}
```

As you can see in the example above, we use the *task* view modifier with the *id* parameter, which means it cancels the previous task as soon as the *id* changes and creates a new one. 

I will rephrase it to make it more obvious: as soon as the *id* changes, SwiftUI marks the task as cancelled. It only marks the task as cancelled but doesn’t stop the execution. So, on every new symbol in the search field, SwiftUI starts a task and marks the previous one as cancelled.

```swift
@MainActor @Observable final class Store {
    static let searchURL = URL(string: "https://domain.com/api/search")!
    private(set) var results: [String] = []
    
    func search(matching query: String) async {
        let url = Self.searchURL.appending(
            queryItems: [
                .init(name: "q", value: query)
            ]
        )
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            try Task.checkCancellation()
            results = try JSONDecoder().decode([String].self, from: data)
        } catch {
            results = []
        }
    }
}
```

Here is the example of the *Store* type having the async *search* function. Inside the *search* function, we make an async network request. After that, we call the *checkCancellation* function on the *Task* type. The *checkCancellation* function is pretty simple; it throws an error whenever the task is already cancelled, and that way, we can stop the execution of the *search* function.

In our example, we just catch the error and clean the *results* variable. In more complex cases, you might have multiple sequenced async calls, and the *checkCancellation* function can save you from doing unnecessary work.

```swift
@MainActor @Observable final class Store {
    static let searchURL = URL(string: "https://domain.com/api/search")!
    private(set) var results: [String] = []
    
    func search(matching query: String) async {
        let url = Self.searchURL.appending(
            queryItems: [
                .init(name: "q", value: query)
            ]
        )
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            try Task.checkCancellation()
            // make another network request
            try Task.checkCancellation()
            results = try JSONDecoder().decode([String].self, from: data)
        } catch {
            results = []
        }
    }
}
```

There is another option allowing us to check cancellation without throwing an error. The *Task* type provides us with the *isCancelled* property, which is a boolean value indicating whenever the task is cancelled.


```swift
actor SearchService {
    static let searchURL = URL(string: "https://domain.com/api/search")!
    private var cachedResults: [String] = []
    
    func search(matching query: String) async throws -> [String] {
        guard !Task.isCancelled else {
            return cachedResults
        }
        
        let url = Self.searchURL.appending(
            queryItems: [
                .init(name: "q", value: query)
            ]
        )
        
        let (data, _) = try await URLSession.shared.data(from: url)
        cachedResults = try JSONDecoder().decode([String].self, from: data)
        return cachedResults
    }
}
```

The *isCancelled* property provides you information about task status, you can check it whenever needed to decide how to model your next steps.

Usually, you don’t need to manually cancel a task using Swift Concurrency, and it handles it for you. In some cases, you might need unstructured tasks. You can create them using the Task type, and it also provides us with the *cancel* function, allowing us to mark the task as cancelled.

```swift
struct ExampleView: View {
    @State private var task: Task<Void, Never>?
    
    var body: some View {
        Button("Tap me") {
            task = Task {
                // call async function here
            }
        }
        
        Button("Cancel") {
            task?.cancel()
        }
    }
}
```

Today, we learned how to use the cooperative cancellation model provided by Swift’s Concurrency feature. In the future post, I will continue the topic by covering more advanced subjects related to Swift’s Concurrency language feature. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
