---
title: Streaming changes with Observations
layout: post
---

Apple introduced the Observation framework a few years ago. The Observation framework became the main tool for building observable models, replacing the Combine framework. This week, we will talk about a new piece of the framework called Observations.

{% include friends.html %}

The primary drawback of the Observation framework was its inability to convert observable types into asynchronous streams, allowing us to observe them in the same manner as we do with the *Publisher* in Combine.

```swift
func startObservation() {
    withObservationTracking {
        render(store.state)
    } onChange: {
        Task { startObservation() }
    }
}
```

While the Observation framework offers the *withObservationTracking* function, which allows us to manually monitor changes in an observable type, it still has some limitations. 

First of all, you have to manually initiate recursive observation, because it only fires for the first change. Second, it doesn’t fit into the Swift Concurrency world, because you can’t use it as an async stream inside the asynchronous for loop. Fortunately, Apple fixed all of these by introducing the new *Observations* type. It is designed to work in pair with the *Observable* macro.

```swift
let store = Store() // Observable

struct State {
    let items: [String]
    let isLoading: Bool
}

let streamOfStates = Observations {
    let state = State(
        items: store.items,
        isLoading: store.isLoading
    )
    return state
}

for await state in streamOfStates {
    render(state)
}
```

The *Observations* type conforms to the *AsyncSequence* protocol, allowing us to use instances of this type inside asynchronous for loops. The closure that we use to initiate the instance of the *Observations* type implicitly observes all the properties of observable instances that you touch.

As you can see in the example below, we create a new state inside the closure. We touch the *items* and *isLoading* properties of the *Store* type conforming to the *Observable* protocol. 

We observe instances of the *State* type within the async for loop. Whenever the *items* and *isLoading* properties change, it emits a new instance of the *State* type, which we asynchronously retrieve within the loop.

The *Observations* type is intelligent enough to utilize transactional updates, which means it doesn’t emit a value for each change. It can group updates when you have changes in both the *items* and *isLoading* properties together.

```swift
extension Observable {
    func stream<Value: Sendable>(
        of keyPath: KeyPath<Self, Value>
    ) -> any AsyncSequence<Value, Never> {
        Observations {
            self[keyPath: keyPath]
        }
    }
}
```

Here is a small extension on the *Observable* protocol allowing you easily create asynchronous sequences for a *KeyPath* on observable type.

```swift
for await items in store.stream(of: \.items) {
    print(items)
}
```

The introduction of the *Observations* type marks a significant improvement in Swift’s data observation capabilities, especially for developers embracing Swift Concurrency. It bridges the gap between reactive-style updates and modern asynchronous patterns, making it easier to build clean, efficient, and responsive UIs.
