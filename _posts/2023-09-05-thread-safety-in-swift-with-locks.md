---
title: Thread safety in Swift with locks
layout: post
category: Swift Language Features
image: /public/swift.png
---

Today, we will discuss thread safety, an essential programming aspect. I decided to cover this topic because of the issue I've noticed in the codebase I helped to build. This type of bug is straightforward to create but very hard to fix. So investing time into building a type-safe type in your codebase is much better.

{% include friends.html %}

Let's look at a simple example of the state management concept.

```swift
@dynamicMemberLookup final class Store<State, Action> {
    typealias Reduce = (State, Action) -> State
    
    private var state: State
    private let reduce: Reduce
    
    init(state: State, reduce: @escaping Reduce) {
        self.state = state
        self.reduce = reduce
    }
    
    subscript<T>(dynamicMember keyPath: KeyPath<State, T>) -> T {
        state[keyPath: keyPath]
    }
    
    func send(_ action: Action) {
        state = reduce(state, action)
    }
}
```

The *Store* type looks pretty simple. Is there a condition where this type can crash your app? Yes, this type may lead to data races and race conditions where the first one crashes your app, and the second may break the logic you use. We can prove it by writing a simple unit test.

```swift
import XCTest

final class StoreTests: XCTestCase {
    struct State {
        var value = 0
    }
    
    func testThreadSafety() {
        let store = Store<State, Void>(state: .init()) { state, _ in
            var state = state
            state.value += 1
            return state
        }
        
        DispatchQueue.concurrentPerform(iterations: 1_000_000) { _ in
            store.send(())
        }
        
        XCTAssertEqual(store.value, 1_000_000)
    }
}
```

Assume that you share a *Store* instance between different threads in your app, and whenever one thread tries to read the state variable while another thread is writing it crashes.

We can control the concurrent access to the state variable using a lock mechanism to solve the issue. Apple provides us with the *OSAllocatedUnfairLock* type from iOS 16. It allows us to lock the access to the variable while we read or write it to guarantee exclusive access.

```swift
@dynamicMemberLookup final class Store<State, Action> {
    typealias Reduce = (State, Action) -> State
    
    private var state: OSAllocatedUnfairLock<State>
    private let reduce: Reduce
    
    init(state: State, reduce: @escaping Reduce) {
        self.state = .init(initialState: state)
        self.reduce = reduce
    }
    
    subscript<T>(dynamicMember keyPath: KeyPath<State, T>) -> T {
        state.withLock { $0[keyPath: keyPath] }
    }
    
    func send(_ action: Action) {
        state.withLock { state in
            state = reduce(state, action)
        }
    }
}
```

As you can see in the example above, we wrap our *state* property with an instance of the *OSAllocatedUnfairLock* type. Whenever we need to read or write the *state* property, we cover the logic with the *withLock* function. It automatically locks and releases at the end of the closure we provide.

*OSAllocatedUnfairLock* is very fast, but it doesn't support recursive locking. It crashes whenever you lock it twice from the same thread, so you should be careful while using it and not call it recursively. Instead, you can use the *NSRecursiveLock* type, allowing the same thread to lock recursively, but this implementation is a bit slower than *OSAllocatedUnfairLock*.

```swift
@dynamicMemberLookup final class Store<State, Action> {
    typealias Reduce = (State, Action) -> State
    
    private var state: State
    private let reduce: Reduce
    
    private let lock = NSRecursiveLock()
    
    init(state: State, reduce: @escaping Reduce) {
        self.state = state
        self.reduce = reduce
    }
    
    subscript<T>(dynamicMember keyPath: KeyPath<State, T>) -> T {
        lock.withLock {
            state[keyPath: keyPath]
        }
    }
    
    func send(_ action: Action) {
        lock.withLock {
            state = reduce(state, action)
        }
    }
}
```

Finally, we can safely share an instance of the *Store* type between different threads and never worry about strange crashes. You should always make your classes thread-safe whenever possible to use them in the multithreaded environment, even accidentally. Invest earlier and save your time in the future.

Today we learned how to use the *NSRecursiveLock* and *OSAllocatedUnfairLock* types to make any class thread safe. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and happy multithreading!
