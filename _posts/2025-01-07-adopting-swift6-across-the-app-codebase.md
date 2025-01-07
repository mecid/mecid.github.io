---
title: Adopting Swift 6 across the app codebase
layout: post
category: Swift Language Features
image: /public/swift.png
---

I’ve been using Swift Concurrency since its initial version, which introduced the **async** and **await** keywords to enable asynchronous work. Over time, Swift Concurrency has become more powerful and provides robust data-race safety by allowing the Swift compiler to identify potential issues. 

{% include friends.html %}

However, using Swift 6 mode with all the warnings and errors it generates can be cumbersome. This week, I’ll share some tips and guidance that I use in my codebase to keep Swift 6 mode enabled and maximize the benefits of code safety. 

Swift 6’s compiler frequently complains about sendability because it’s the primary cause of data races. A data race occurs when one part of your code writes to a memory simultaneously with another part reading the same memory reference. In this case your app crashes with strange **EXC_BAD_ACCESS** error.

Usually, data races occur when multiple threads share an instance of a class while one of them writes to it. To avoid this, I strive to minimize the use of classes in my codebase. Let me share some tips that help me maintain a data-race-free codebase.

If there’s no way to modify something, there’s no chance of a data race. This principle encourages the use of structs as much as possible. For instance, I define all my data types as sendable and immutable structs.

```swift
public struct Statistics: Sendable, Hashable {
    public let value: Double
    public let interval: DateInterval
    
    public init(value: Double, interval: DateInterval) {
        self.value = value
        self.interval = interval
    }
}
```

In the example above, we define the *Statistics* type as sendable structure. In many cases, structs can implicitly infer sendability but whenever you define it as public you have to do it explicitly.

Not only data types can be structs. I define my service types that doesn’t hold any state as structs also.

```swift
public struct HealthService: Sendable {
    private let store: HKHealthStore
    
    public init(store: HKHealthStore) {
        self.store = store
    }
    
    public var age: Int {
        guard
            let dateOfBirthComponents = try? store.dateOfBirthComponents(),
            let dateOfBirth = Calendar.current.date(from: dateOfBirthComponents)
        else {
            return 0
        }
        
        let age = Calendar.current.dateComponents([.year], from: dateOfBirth, to: .now)
        return age.year ?? 0
    }
}
```

Structs are incredibly powerful and cost-effective in terms of creation compared to classes. Swift 6 allows you to effortlessly pass sendable structs between threads without encountering any warnings or data races. Therefore, I strongly recommend incorporating structs into your code as much as possible.

Structs are great, but it is not possible to build the whole app without reference types. Classes are good for one particular thing; they allow us to share state without duplicating it. A set of views in your app may depend on a single piece of state that you want to share between them. Unfortunately, structs will not help you here because every view will get its own copy of the struct instance, that is not what we usually want.

> To learn more about global actors and their benefits, take a look at my ["Global actors in Swift"](/2024/03/12/global-actors-in-swift/) post.

I try to use classes in my app only for view-related stuff. So, it might be a view model or delegate type. Both of them are view-related, that’s why I annotate them with *@MainActor*. Global actors are another way to make a type  implicitly sendable. Whenever your type is isolated to a global actor, there is no chance to concurrently read and write the data it stores. Thanks to actors.

```swift
@MainActor @Observable final class Store<State, Action> {
    private(set) var state: State
    private let reduce: (State, Action) -> State
    
    init(
        initialState state: State,
        reduce: @escaping (State, Action) -> State
    ) {
        self.state = state
        self.reduce = reduce
    }
    
    func send(_ action: Action) {
        state = reduce(state, action)
    }
}
```

We already talked about data types, stateless service types and view-related stuff. The last category of types that my apps have is statefull service types. For example, you might have a search service that caches results. It might be used concurrently by multiple threads and it hold the state of the cache. 

```swift
actor SearchService {
    private var history = Tree<Food>()
    
    func search() async throws -> [Food] {
        // read history
        
        // make network request
        
        // mutate history
    }
}
```

This category of types are potentially source of data races in our apps and I’m happy to say that it is easy to solve by using a single **actor** keyword. As you may know an actor protect its state and allows only mutually exclusive access eliminating data-races.

> To learn more about actors in Swift, take a look at my ["Thread safety in Swift with actors"](/2023/09/19/thread-safety-in-swift-with-actors/) post.

Swift is an excellent language that enables you to write expressive and secure code. It offers a comprehensive set of tools to help you accomplish your goals. It’s always beneficial to be aware of the available tools and select the most suitable one for each specific task.

Today, I’ve shared my tips and guidance on transitioning to Swift 6 language mode in a codebase. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
