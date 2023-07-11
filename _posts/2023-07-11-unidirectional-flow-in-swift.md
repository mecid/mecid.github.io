---
title: Unidirectional flow in Swift
layout: post
---

This week I will talk about the state management approach I have used in my apps for years. We will cover building a predictable, testable, debuggable, and serializable state management system in Swift.

Swift promotes type-safe code by using a set of language features allowing us to encode the correct behavior into the type system. We aim to model our types so that the logical mistake becomes a compile-time error instead of a runtime error. We can achieve that by using value types, enums, optionals, protocols, generics, phantom types, etc.

But what about type-safe state management systems? How can we build it using Swift language features? We will apply the same tools to build a functional and safe state management system.

```swift
import Observation

@Observable final class Store<State, Action> {
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

Here is a basic example of the state management system we can build in Swift. We express the *Store* type that defines two generic types: one for state and another for action. It holds the current state in the variable that we can only mutate inside the *Store* type. It also provides us with the *send* function, which takes a generic action as a parameter and mutates the current state. The whole type is marked with the *Observable* macro allowing us to be notified whenever the state changes.

#### Predictable 
The first goal in building a state management system we want to achieve is to make it predictable. As you can see in the example above, the only way to mutate the state is by sending a predefined action. All the view and view controllers in your app take an instance of the *Store* type and only can read the actual state. There is no way to change something in the state directly. The only way to update the state is to send an action.

```swift
struct ShopState: Equatable {
    var products: [String] = []
}

enum ShopAction: Equatable {
    case add(String)
    case remove(String)
}

let reduce: (ShopState, ShopAction) -> ShopState = { state, action in
    var newState = state
    
    switch action {
    case let .add(product):
        newState.products.append(product)
    case let .remove(product):
        newState.products.removeAll { $0 == product }
    }
    
    return newState
}

typealias ShopStore = Store<ShopState, ShopAction>
```

*Reduce* function is the only place containing state update logic. It doesn't mean you should have a single function for the whole app. You can compose multiple *reduce* functions into a single one. Usually, I have a *reduce* function per feature module of my app.

```swift
import SwiftUI

struct ShopView: View {
    @State private var store = ShopStore(
        initialState: .init(products: []),
        reduce: reduce
    )
    
    var body: some View {
        List(store.state.products, id: \.self) { product in
            Text(verbatim: product)
                .swipeActions {
                    Button(role: .destructive) {
                        store.send(.remove(product))
                    } label: {
                        Label("Delete", systemImage: "trash")
                    }
                }
        }
    }
}
```

As you can see in the example above, we define a list displaying the products. We also allow to remove a product from the list by using swipe actions. The view has read-only access to the state and can only render it as is. The only way to mutate the state is to use the *send* function with one of the predefined cases of the *ShopAction* enum.

This approach is called unidirectional flow. As its name says, there is only one direction to go. The view sends actions, the store updates the state, and the view takes the updated state. By applying this approach, we make our state management predictable. You always know where to look for the mutation and where your app's business logic lives.

#### Testable
As we said before, the app logic lives in the *reduce* function. It is a pure function that takes the current state and action to apply as parameters and returns a new state. Both state and action are value types. Usually, we use a struct for the state and enum for the action. It means we can easily verify any *reduce* functions in unit tests.

```swift
import XCTest

final class ShopReducerTests: XCTestCase {
    func testRemove() {
        let initialState = ShopState(products: ["p1"])
        let newState = reduce(initialState, .remove("p1"))
        
        XCTAssertTrue(newState.products.isEmpty)
    }
}
```

All we need to do is to create an initial state and call the *reduce* function with the particular action. Then, we can verify that the new state returned by the *reduce* function contains all the required changes.

#### Previewable
This approach works great with Xcode previews. You can create multiple previews with different initial states. For example, one for the empty list and another for the list of products.

```swift
#Preview {
    AnotherShopView(
        store: .init(
            initialState: .init(products: ["Product"]),
            reduce: reduce
        )
    )
}

#Preview {
    AnotherShopView(
        store: .init(
            initialState: .init(products: []),
            reduce: reduce
        )
    )
}
```

There is no need for mocking protocols because the state is a simple struct you can create and put into the store to render in the view.

#### Debuggable
Debugging and logging became very easy. If something goes wrong, you know where to look. The *reduce* function is the only place containing app logic, and it is the best place to put log messages because all actions go through the *reduce* function. It means your logs will never miss any state change.

By keeping the feature state in a single place, we can easily track the history of the state changes. It will help us to understand which action sequence led to a bug. We also can encode the state into a JSON and retrieve it to analyze or restore it under the debugger to inspect the bug.

#### Modular
Many of my colleagues correlate the unidirectional flow with a single-state container where the whole app state lives in a single instance of the particular *AppState* struct. Yes, it is possible, but it is not the only way.

Usually, I define a store per feature. So every independent feature has its own store. It allows us to optimize for the performance because, in the case of a huge app, a single store in the app can lead to performance degradation where the whole app hierarchy refreshes on every small state change.

#### References
After years of building apps similarly, these ideas resulted in a Swift Package called swift-unidirectional-flow. It implements all the ideas we discussed in a production-ready code supporting concurrency and other features you might need to build a real-life app.

#### Conclusion
