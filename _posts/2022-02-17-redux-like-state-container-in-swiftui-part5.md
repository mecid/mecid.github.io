---
title: Redux-like state container in SwiftUI. Swift concurrency model.
layout: post
category: Architecture
image: /public/store.png
---

Over the last two years, I have actively used unidirectional flow to develop my apps. I covered the approach I use in the series of posts about building Redux-like state containers. This week I want to share with you how this approach adapts to the latest changes in Swift by applying the new concurrency model.

{% include friends.html %}

Swift concurrency is the new language feature allowing us to write concurrent code straightforwardly. I'm not going to cover this in detail and assume that you know how to use it. If you are not familiar with the Swift Concurrency model, I suggest starting with the [official docs](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html).

```swift
typealias Reducer<State, Action, Dependencies> =
    (inout State, Action, Dependencies) -> AnyPublisher<Action, Never>?
    
final class Store<State, Action, Dependencies>: ObservableObject {
    @Published private(set) var state: State

    private let dependencies: Dependencies
    private let reducer: Reducer<State, Action, Dependencies>
    private var cancellables: Set<AnyCancellable> = []

    init(
        initialState: State,
        reducer: @escaping Reducer<State, Action, Dependencies>,
        dependencies: Dependencies
    ) {
        self.state = initialState
        self.reducer = reducer
        self.dependencies = dependencies
    }

    func send(_ action: Action) {
        guard let effect = reducer(&state, action, dependencies) else {
            return
        }

        effect
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: send)
            .store(in: &cancellables)
    }
}
```

As you can see above, I have been using the Combine framework heavily to do asynchronous work. But now I'm switching to the new Swift concurrency model. And I am pleased to see that the new concurrency model plays very well with the Combine framework, and we can mix them to don't break the working code in our apps and migrate step by step.

```swift
typealias Reducer<State, Action, Dependencies> =
    (inout State, Action, Dependencies) -> AnyPublisher<Action, Never>?

@MainActor final class Store<State, Action, Dependencies>: ObservableObject {
    @Published private(set) var state: State

    private let dependencies: Dependencies
    private let reducer: Reducer<State, Action, Dependencies>

    init(
        initialState: State,
        reducer: @escaping Reducer<State, Action, Dependencies>,
        dependencies: Dependencies
    ) {
        self.state = initialState
        self.reducer = reducer
        self.dependencies = dependencies
    }

    func send(_ action: Action) async {
        guard let effect = reducer(&state, action, dependencies) else {
            return
        }

        for await action in effect.values {
            await send(action)
        }
    }
}
```

Here we pushed a few changes to support the new concurrency model:
1. We made our *send* method an async by adding the particular keyword to the method definition.
2. We use the *values* property on the Combine framework's *Publisher* type to convert it into an *AsyncSequence*.
3. We use *for await* keyword to iterate over values of *AsyncSequence* and apply them to the store.
4. We added *@MainActor* annotation to access our state only via the main thread.

As you can see, we made very few changes to migrate to the new concurrency model and gain new features like task cancellation out of the box. We don't need to change anything else, and we still can use our old reducers with the Combine framework because they work together very nicely.

```swift
func appReducer(
    state: inout AppState,
    action: AppAction,
    dependencies: AppDependencies
) -> AnyPublisher<AppAction, Never>? {
    switch action {
    case let .setSearchResults(repos):
        state.searchResult = repos
    case let .search(query):
        return dependencies.service
            .searchPublisher(matching: query)
            .replaceError(with: [])
            .map { AppAction.setSearchResults(repos: $0) }
            .eraseToAnyPublisher()
    }
    return nil
}
```

Let's go forward and replace the dependencies of our reducers with async closures. We still must return the Combine framework's *Publisher* type, but we need somehow to wrap an async closure call with the Combine publisher. Unfortunately, there is no way to do it automatically, but we can create the *Publisher* type that does it for us.

> To learn more about using async closures as dependencies, take a look at my ["Microapps architecture in Swift. Dependency Injection"](/2022/02/02/microapps-architecture-in-swift-dependency-injection/) post.

```swift
import Foundation
import Combine

public struct SendablePublisher<Output, Failure: Error>: Publisher {
    let upstream: AnyPublisher<Output, Failure>

    public init(
        fullFill: @Sendable @escaping () async throws -> Output
    ) where Failure == Error {
        var task: Task<Void, Never>?
        upstream = Deferred {
            Future { promise in
                task = Task {
                    do {
                        let result = try await fullFill()
                        try Task.checkCancellation()
                        promise(.success(result))
                    } catch {
                        promise(.failure(error))
                    }
                }
            }
        }
        .handleEvents(receiveCancel: { task?.cancel() })
        .eraseToAnyPublisher()
    }

    public func receive<S>(subscriber: S) where S : Subscriber, Failure == S.Failure, Output == S.Input {
        upstream.subscribe(subscriber)
    }
}
```

Let me introduce the *SendablePublisher* type that allows us to convert any async closure to the Combine publisher. Please look at how we use the *handleEvents* operator to implement the support for the cooperative cancellation. Now we can refactor our reducer to use async closures for side effects.

```swift
struct AppDependencies {
    let search: @Sendable (String) async throws -> [Repo]
}

func appReducer(
    state: inout AppState,
    action: AppAction,
    dependencies: AppDependencies
) -> AnyPublisher<AppAction, Never>? {
    switch action {
    case let .setSearchResults(repos):
        state.searchResult = repos
    case let .search(query):
        return SendablePublisher {
            try await dependencies.search(query)
        }
            .replaceError(with: [])
            .map(AppAction.setSearchResults)
            .eraseToAnyPublisher()
    }
    return nil
}
```

SwiftUI provides us with the *task* view modifier accepting an async closure. It automatically runs the closure when the view appears and tracks its lifecycle. SwiftUI cancels the task created for an async closure whenever the view disappears, and if you support the cooperative cancellation model, your task automatically stops.

```swift
import SwiftUI

typealias AppStore = Store<AppState, AppAction, AppDependencies>

struct SearchView: View {
    @ObservedObject var store: AppStore
    let query: String

    var body: some View {
        NavigationView {
            List {
                ForEach(store.state.searchResult, id: \.id) { repo in
                    Text(repo.title)
                }
            }
            .navigationTitle("Search")
            .task {
                await store.send(.search(query))
            }
        }
    }
}
```

This week we learned how to migrate our codebase from the Combine framework to the new Swift concurrency model using small steps. I love the way they play together, but I believe that the new Swift concurrency model is the future of async tasks. I hope to eliminate the Combine framework whenever the *AsyncSequence* type gains all the needed transformation operators offered by the Combine framework's *Publisher* type.

I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

1. [Redux-like state container in SwiftUI. Basics](/2019/09/18/redux-like-state-container-in-swiftui/)
2. [Redux-like state container in SwiftUI. Best practices](/2019/09/25/redux-like-state-container-in-swiftui-part2/)
3. [Redux-like state container in SwiftUI. Container Views.](/2019/10/02/redux-like-state-container-in-swiftui-part3/)
4. [Redux-like state container in SwiftUI. Connectors.](/2021/02/03/redux-like-state-container-in-swiftui-part4/)
5. Redux-like state container in SwiftUI. Swift concurrency model.

#### References
The series of posts have built on a foundation of ideas started by other libraries, particularly Redux, Elm, and TCA.
0. [WWDC20 - Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/)
1. [Redux](https://redux.js.org) - The JavaScript library that popularized unidirectional data flow.
2. [The Elm Architecture](https://guide.elm-lang.org/architecture/) - A purely functional language and runtime that inspired the creation of Redux.
3. [The Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture) - A library that bridges concepts from the Elm Architecture and Redux to Swift. It introduced the “environment” and “effect” patterns that this series covers.
