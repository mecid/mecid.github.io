---
title: Dependency container on top of task local values in Swift
layout: post
category: Swift Language Features
image: /public/swift.png
---

Task local values is the recent addition to the concurrency feature of the Swift language. This week, we will not only learn the basics of task local values, but also discuss the interesting usage where we will build the dependency injection container using this language feature.

{% include friends.html %}

Task local values is the new way to create a *Task* shared value. It is implicitly shared across child tasks and accessible both from the sync and async context.

```swift
struct Request: Identifiable {
    let id = UUID()
}

extension Request {
    @TaskLocal static var current = Request()
}
```

As you can see in the example above, we define the *Request* type with a unique identifier. We also create an extension for the *Request* type that uses the *@TaskLocal* macro to define a static property for the current *Request* instance. We should always provide a default value for the task local values or make them optional.

You can use the *@TaskLocal* macro only with static properties because the main goal is to create a shared instance of the type implicitly available for the async tasks. It works very similarly to the environment feature of SwiftUI, allowing you to implicitly carry on data down to the view hierarchy.

```swift
func fetchData() async throws -> Data? {
    let newRequest = Request()
    
    return try await Request.$current.withValue(newRequest) {
        try await withThrowingTaskGroup(of: Data.self) { group in
            group.addTask {
                let url = URL(string: "https://example.com/api/\(Request.current.id.uuidString)")!
                let (data, _) = try await URLSession.shared.data(from: url)
                return data
            }
            
            group.addTask {
                // You can access Request.current anywhere in the async-context
            }
            
            for try await data in group {
                return data
            }
        }
    }
}
```

Task local values are read-only when you try to access them directly. You can modify a task local value using the *withValue* function. The updated value will be available in the scope of the provided closure and implicitly shared across the async context of the current *Task*.

You are not going to use task local values very often, but you can use them whenever you need to propagate some piece of state down into the hierarchy of an async task. In our example, we share a request to make logging of the request identifier easier in the world of the concurrent tasks.

All of these inspired me to use the task local values for implicit dependency injection. We can create a dependency container holding services that we might need to replace for mocked versions. Let’s see how we can achieve it.

```swift
struct Dependencies {
    let fetchStatistics: (DateInterval) async throws -> [HKStatistics]
}

extension Dependencies {
    static var production: Dependencies {
        let store = HKHealthStore()
        
        return .init(
            fetchStatistics: { interval in
                let query = HKStatisticsCollectionQueryDescriptor(
                    predicate: .quantitySample(type: HKQuantityType(.bodyMass)),
                    options: .discreteAverage,
                    anchorDate: interval.start,
                    intervalComponents: DateComponents(day: 1)
                )
                return try await query.result(for: store).statistics()
            }
        )
    }
}

extension Dependencies {
    static var mock: Dependencies {
        let mockedStatistics: [HKStatistics] = [
            // mocked value here
        ]
        
        return .init(
            fetchStatistics: { _ in mockedStatistics }
        )
    }
}
```

As you can see in the example above, we define the *Dependencies* type holding the fetching statistics function. In the real world, you would have much more service functions there. We also define the production-ready and mocked version of our service function.

```swift
extension Dependencies {
    @TaskLocal static var active: Dependencies = .production
}
```

Here we define the task local value for the active dependency container. By default, we use the production-ready version of the dependency container. For the purpose of testing, we can replace the production-ready container with a mocked version implicitly using the withValue function.

```swift
@Test func verifySomething() async throws {
    Dependencies.$active.withValue(.mock) {
        // Mocked environment activated
        let interval: DateInterval = //...
        let statistics = try await Dependencies.active.fetchStatistics(interval)
        
        #expect(statistics.count == 1)
    }
}
```

Inside the test target, we want to avoid accessing the real APIs and use the mocked version to make our tests less flaky. That's why we use the *withValue* function to replace the real service with the mocked one implicitly by providing the access via *active* property. 

Today, we learned the basics of task local values and discussed how we would engage with it even more by building a dependency container using this feature. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
