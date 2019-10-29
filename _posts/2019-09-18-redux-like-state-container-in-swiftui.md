---
title: Redux-like state container in SwiftUI. Basics.
layout: post
---

This week we will talk about building a state container similar to *Redux* which provides a single source of truth for your app. A single state for the whole app makes it easier to debug and inspect. Single source of truth eliminates tons of bugs produced by creating multiple states across the app.

#### Single source of truth
The main idea here is describing the whole app state by using a single struct or composition of structs. Assume that we are working on a Github repos search app where the state is repos array which we fetch matching some query using Github API.

```swift
struct AppState {
    var searchResult: [Repo] = []
}
```

Next step is passing the read-only app state to every view inside the app. The best way for doing that is *SwiftUI's Environment* feature. We can give an object holding the whole state of the app to the *Environment* of the root view. Root view will share its *Environment* with all child views. To learn more about the *Environment* feature of *SwiftUI*, take a look at ["The power of Environment in SwiftUI"](/2019/08/21/the-power-of-environment-in-swiftui/) post.

```swift
final class Store: ObservableObject {
    @Published private(set) var state: AppState
}
```

In the example above, we create a store object which stores the app state and provides read-only access to it. State property uses *@Published* property wrapper, which notifies *SwiftUI* during any changes. It allows us to keep up to date the whole app by deriving it from a single source of truth. We already talked about store objects in the previous posts, to learn more you can check ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

#### Reducer and Actions
It's time to talk about user actions which lead to state mutations. *Action* is a simple enum or composition of enums describing a state change. For example, set loading value during data fetch, assign fetched repositories to a state's property, etc. Let's take a look at an example code for *Action* enum.

```swift
enum AppAction {
    case search(query: String)
    case setSearchResult(repos: [Repo])
}
```

*Reducer* is a function which takes current state, applies *Action* to the state, and generates a new state. Generally, *reducer* or *composition of reducers* is a single place where your app mutates the state. The fact that the only one function can modify the whole app state is super simple, debuggable, and testable. Here is an example of our reduce function.

```swift
struct Reducer<State, Action> {
    let reduce: (inout State, Action) -> Void
}

let appReducer: Reducer<AppState, AppAction> = Reducer { state, action in
    switch action {
    case let .setSearchResults(repos):
        state.searchResult = repos
    }
}
```

#### Unidirectional flow
Let's talk about a data flow now. Every view has read-only access to a state via store object. Views can send *actions* to a store object. *Reducer* modifies the state, and then *SwiftUI* notifies all the views about state changes. *SwiftUI* has a super-efficient diffing algorithm that's why diffing of the whole app state and updating changed views works very fast.

**State -> View -> Action -> State -> View**

This architecture revolves around a strict **unidirectional** data flow. It means that all the data in the application follows the same pattern, making the logic of your app more predictable and easier to understand. Let's modify our store object to support sending *actions*.

```swift
final class Store<State, Action>: ObservableObject {
    @Published private(set) var state: State

    private let appReducer: Reducer<State, Action>

    init(initialState: State, appReducer: @escaping Reducer<State, Action>) {
        self.state = initialState
        self.appReducer = appReducer
    }

    func send(_ action: Action) {
        appReducer(&state, action)
    }
}
```

#### Side effects
We already implemented a *unidirectional* flow which accepts user actions and modifies the state, but what about *async action* which we usually call *side effects*. How to add support for an async task to our store type? I think it is a time to introduce the usage of *Combine framework* which perfectly fits async task processing.

```swift
import Foundation
import Combine

protocol Effect {
    associatedtype Action
    func mapToAction() -> AnyPublisher<Action, Never>
}

enum SideEffect: Effect {
    case search(query: String)

    func mapToAction() -> AnyPublisher<Action, Never> {
        switch self {
        case let .search(query):
            return dependencies.githubService
                .searchPublisher(matching: query)
                .replaceError(with: [])
                .map { AppAction.setSearchResults(repos: $0) }
                .eraseToAnyPublisher()
        }
    }
}
```

We add support for *async tasks* by introducing *Effect* protocol. *Effect* is a sequence of *Actions* which we can publish using Combine framework's *Publisher* type. It allows us to handle async jobs using *Combine* and then publish *actions* which will be used by *reducer* to apply on the current state.

```swift
final class Store<State, Action>: ObservableObject {
    @Published private(set) var state: State

    private let appReducer: Reducer<State, Action>
    private var cancellables: Set<AnyCancellable> = []

    init(initialState: State, appReducer: Reducer<State, Action>) {
        self.state = initialState
        self.appReducer = appReducer
    }

    func send(_ action: Action) {
        appReducer.reduce(&state, action)
    }

    func send<E: Effect>(_ effect: E) where E.Action == Action {
        effect
            .mapToAction()
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: send)
            .store(in: &cancellables)
    }
}
```

#### Real usage example
Finally, we can finish our repos search app which calls Github API asynchronously and fetches repositories matching a query. The full source code available on [Github](https://github.com/mecid/redux-like-state-container-in-swiftui).

```swift
struct SearchContainerView: View {
    @EnvironmentObject var store: Store<AppState, AppAction>
    @State private var query: String = "Swift"

    var body: some View {
        SearchView(
            query: $query,
            repos: store.state.searchResult,
            onCommit: fetch
        ).onAppear(perform: fetch)
    }

    private func fetch() {
        store.send(SideEffect.search(query: query))
    }
}

struct SearchView : View {
    @Binding var query: String
    let repos: [Repo]
    let onCommit: () -> Void

    var body: some View {
        NavigationView {
            List {
                TextField("Type something", text: $query, onCommit: onCommit)

                if repos.isEmpty {
                    Text("Loading...")
                } else {
                    ForEach(repos) { repo in
                        RepoRow(repo: repo)
                    }
                }
            }.navigationBarTitle(Text("Search"))
        }
    }
}
```

We divide our screen into two views: *Container View* and *Rendering View*. *Container View* handles the actions and retrieves the needed piece of state from the global state. *Rendering View* accepts the data and renders it. We already talked about *Container Views* in my previous posts, to learn more take a look at ["Introducing Container views in SwiftUI"](/2019/07/31/introducing-container-views-in-swiftui/) post.

#### Conclusion
Today we learned how to build *Redux-like* state container with *side-effects* in mind. To achieve that we used *SwiftUI's Environment* feature and *Combine* framework. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 