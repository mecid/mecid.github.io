---
title: Task Cancellation in Swift Concurrency
layout: post
category: Swift Language Features
image: /public/swift.png
---

Swift Concurrency provides a cooperative cancellation model to handle task cancellation. This week, we will learn what a cooperative cancellation model is, how to use it, and how to be a good citizen and handle it correctly.

{% include friends.html %}

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
import HealthKit

@MainActor @Observable final class Store {
    private(set) var results: [HKCorrelation] = []
    private let store = HKHealthStore()
    
    func search(matching query: String) async {
        let foodQuery = HKSampleQueryDescriptor(
            predicates: [.correlation(type: .init(.food))],
            sortDescriptors: []
        )
        
        do {
            let food = try await foodQuery.result(for: store)
            
            try Task.checkCancellation()
            
            results = food.filter { food in
                let title = food.metadata?["title"] as? String ?? ""
                return title.localizedStandardContains(query)
            }
        } catch {
            results = []
        }
    }
}
```

Here is the example of the *Store* type having the async *search* function. Inside the *search* function, we make an async health request. After that, we call the *checkCancellation* function on the *Task* type. The *checkCancellation* function is pretty simple; It throws an error whenever the task is already cancelled, and that way, we can stop the execution of the *search* function and avoid redundant filtering.

In our example, we just catch the error and clean the *results* variable. In more complex cases, you might have multiple sequenced async calls, and the *checkCancellation* function can save you from doing unnecessary work.

```swift
import HealthKit

@MainActor @Observable final class Store {
    private(set) var results: [HKCorrelation] = []
    private let store = HKHealthStore()
    
    func search(matching query: String) async {
        let foodQuery = HKSampleQueryDescriptor(
            predicates: [.correlation(type: .init(.food))],
            sortDescriptors: []
        )
        
        do {
            let food = try await foodQuery.result(for: store)
            
            try Task.checkCancellation()
            // another query here
            try Task.checkCancellation()
            
            results = food.filter { food in
                let title = food.metadata?["title"] as? String ?? ""
                return title.localizedStandardContains(query)
            }
        } catch {
            results = []
        }
    }
}
```

There is another option allowing us to check cancellation without throwing an error. The *Task* type provides us with the *isCancelled* property, which is a boolean value indicating whenever the task is cancelled.


```swift
actor SearchService {
    private var cachedResults: [HKCorrelation] = []
    private let store = HKHealthStore()
    
    func search(matching query: String) async throws -> [HKCorrelation] {
        guard !Task.isCancelled else {
            return cachedResults
        }
        
        let foodQuery = HKSampleQueryDescriptor(
            predicates: [.correlation(type: .init(.food))],
            sortDescriptors: []
        )
        
        let food = try await foodQuery.result(for: store)
        
        guard !Task.isCancelled else {
            return cachedResults
        }
        
        cachedResults = food.filter { food in
            let title = food.metadata?["title"] as? String ?? ""
            return title.localizedStandardContains(query)
        }
        
        return cachedResults
    }
}
```

The *isCancelled* property provides you information about task status, you can check it whenever needed to decide how to model your next steps.

Usually, you don’t need to manually cancel a task using Swift Concurrency, and it handles it for you. In some cases, you might need unstructured tasks. You can create them using the Task type, and it also provides us with the *cancel* function, allowing us to mark the task as cancelled.

```swift
struct ExampleView: View {
    @State private var store = Store()
    @State private var task: Task<Void, Never>?
    
    var body: some View {
        Button("Tap me") {
            task = Task {
                await store.fetch()
            }
        }
        
        Button("Cancel") {
            task?.cancel()
        }
    }
}
```

Even in this case, the task is not stopped by the button action; Swift only marks it as cancelled, and it is still up to you to handle cancellation in the *fetch* function of the *Store* type.

Today, we learned how to use the cooperative cancellation model provided by Swift’s Concurrency feature. In the future post, I will continue the topic by covering more advanced subjects related to Swift’s Concurrency language feature. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
