---
title: Functional core Imperative shell in Swift. Unidirectional Flow.
layout: post
category: Architecture
image: /public/store.png
---

A few weeks ago, we talked about the idea of Functional Core and Imperative shell in Swift. The goal is to extract the pure logic using value types and keep side effects in the thin object layer. This week, we will look at how we can apply this approach in an opinionated way by using unidirectional flow.

> If you are not familiar with the idea of unidirectional flow, I highly encourage you to read my series of posts about ["Redux-like state container in SwiftUI"](/2019/09/18/redux-like-state-container-in-swiftui/).

#### Functional Core
The functional core is the layer responsible for all the logic in our app that we want to verify using unit tests. It should be pure, without any side effects. We want to provide the input and verify the output. Usually, the implementation of unidirectional flow requires many reducer functions that accept the state and action and return a new state. Let's define the reducer function in code.

```swift
typealias Reducer<State, Action> = (State, Action) -> State
```

As you can see, the reducer function takes the current state and action to apply on that state and returns a new state. I'm working on the app for intermittent fasting tracking. Let's take a look at how I could implement the timer logic.

```swift
struct TimerState: Equatable {
    var start: Date?
    var end: Date?
    var goal: TimeInterval
}

enum TimerAction {
    case start
    case finish
    case reset
}

let timerReducer: Reducer<TimerState, TimerAction> = { state, action in
    var state = state

    switch action {
    case .start:
        state.start = .now
    case .finish:
        state.end = .now
    case .reset:
        state.start = nil
        state.end = nil
    }

    return state
}
```

Here is the real example from my codebase implementing timer management logic. As you can see, it is pure and doesn't have any side effects. It allows me to quickly verify the logic using unit tests without mocks and stubs.

```swift
import XCTest

final class TimerReducerTests: XCTestCase {
    func testStart() {
        let state = TimerState(goal: 13 * 3600)
        XCTAssertNil(state.start)
        let newState = timerReducer(state, .start)
        XCTAssertNotNil(newState.start)
    }
}
```

Value types like structs and enums are great tools for implementing app logic in a pure and very testable. But we still need side effects. For example, I want to share the timer state with my friends using CloudKit.

#### Imperative Shell
The imperative shell is the object layer holding the app's state represented by a value type. We also utilize the object layer to make side-effects and apply results on top of the state. Let's start by defining a generic object that holds the state.

```swift
@MainActor public final class Store<State, Action>: ObservableObject {
    @Published public private(set) var state: State

    private let reducer: Reducer<State, Action>

    init(
        initialState state: State,
        reducer: @escaping Reducer<State, Action>
    ) {
        self.reducer = reducer
        self.state = state
    }

    public func send(_ action: Action) {
        state = reducer(state, action)
    }
}
```

Here is the imperative shell defined using the *Store* class. As you can see, we use the object layer to hold the app state represented by a value type. We also provide thread safety by utilizing the *MainActor* and allowing mutations only by feeding actions into the store using the *send* method on *Store* type. This is how we implement unidirectional flow with the idea of Functional core and Imperative shell. But we still miss side effects.

#### Side effects
The imperative shell should provide us with a way to make side effects. We should separate side effects from the pure logic of our app, but we still want to test side effects using integration tests. Let's introduce a new type called *Middleware* that defines a side effect handler.

```swift
typealias Middleware<State, Action, Dependencies> =
    (State, Action, Dependencies) async -> Action?
```

The main idea behind the *Middleware* type is to intercept pure actions, make side effects like async requests, and return a new action that we can feed into the store and reduce. Let's add this functionality to the *Store* type.

```swift
@MainActor public final class Store<State, Action, Dependencies>: ObservableObject {
    @Published public private(set) var state: State

    private let reducer: Reducer<State, Action>
    private let dependencies: Dependencies
    private let middlewares: [Middleware<State, Action, Dependencies>]

    init(
        initialState state: State,
        reducer: @escaping Reducer<State, Action>,
        dependencies: Dependencies,
        middlewares: [Middleware<State, Action, Dependencies>] = []
    ) {
        self.reducer = reducer
        self.state = state
        self.dependencies = dependencies
        self.middlewares = middlewares
    }

    public func send(_ action: Action) async {
        state = reducer(state, action)

        await withTaskGroup(of: Optional<Action>.self) { [state, dependencies] group in
            for middleware in middlewares {
                group.addTask {
                    await middleware(state, action, dependencies)
                }
            }

            for await case let action? in group {
                await send(action)
            }
        }
    }
}
```

As you can see, we use the new Swift concurrency feature to implement async work inside the *Store* type. It allows us to run our side effects in parallel and feed the actions into the store. We secure access to our state by marking the *Store* type with *@MainActor*. Using the *TaskGroup*, we automatically gain the cooperative cancellation of side effects. The *Store* type also holds all the dependencies like networking, notification center, etc., to provide them to middlewares.

```swift
struct TimerState: Equatable {
    var start: Date?
    var end: Date?
    var goal: TimeInterval
    var sharingStatus = SharingStatus.notShared
}

enum SharingStatus: Equatable {
    case shared
    case uploading
    case notShared
}

enum TimerAction: Equatable {
    case start
    case finish
    case reset
    case share
    case setSharingStatus(SharingStatus)
}

let timerReducer: Reducer<TimerState, TimerAction> = { state, action in
    var state = state

    switch action {
    case .start:
        state.start = .now
    case .finish:
        state.end = .now
    case .reset:
        state.start = nil
        state.end = nil
    case .share:
        state.sharingStatus = .uploading
    case let .setSharingStatus(status):
        state.sharingStatus = status
    }

    return state
}

struct TimerDependencies {
    let share: (Date, Date?) async throws -> Void
}

let timerMiddleware: Middleware<TimerState, TimerAction, TimerDependencies> = { state, action, dependencies in
    switch action {
    case .share:
        guard let start = state.start else {
            return .setSharingStatus(.notShared)
        }

        do {
            try await dependencies.share(start, state.end)
            return .setSharingStatus(.shared)
        } catch {
            return .setSharingStatus(.notShared)
        }
    default:
        return nil
    }
}

import XCTest

final class TimerMiddlewareTests: XCTestCase {
    func testSharing() async throws {
        let state = TimerState(goal: 13 * 3600)
        let dependencies: TimerDependencies = .init { _, _ in }
        let action = await timerMiddleware(state, .share, dependencies)
        XCTAssertEqual(action, .setSharingStatus(.shared))
    }
}
```

And here is the example code showing how to implement a middleware. As you can see, we intercept the action fed into the store, make an async request, and provide another action to the system.

```swift
import SwiftUI

struct RootView: View {
    @StateObject var store = Store(
        initialState: TimerState(goal: 13 * 3600),
        reducer: timerReducer,
        dependencies: TimerDependencies.production
    )

    var body: some View {
        NavigationView {
            VStack {
                if let start = store.state.start, store.state.end == nil {
                    Text(start, style: .timer)
                    Button("Stop") {
                        Task { await store.send(.finish) }
                    }

                    Button("Reset") {
                        Task { await store.send(.reset) }
                    }
                } else {
                    Button("Start") {
                        Task { await store.send(.start) }
                    }
                }
            }
            .navigationTitle("Timer")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button("Share") {
                        Task {
                            await store.send(.share)
                        }
                    }
                }
            }
        }
    }
}
```

#### Conclusion
I use the idea of Functional core and Imperative shell in two of my apps and enjoy the infrastructure it provides me. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
