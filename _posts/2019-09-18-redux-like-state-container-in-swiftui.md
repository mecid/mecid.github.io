---
title: Redux-like state container in SwiftUI
layout: post
---

This week we will talk about building a *Redux-like* state container which provides a single source of truth for your app. A single state for the whole app makes it easier to debug and inspect. Single source of truth eliminates tons of bugs produced by creating multiple states across the app.

#### Single source of truth
The main idea here is describing the whole app state by using a single struct or composition of structs. Assume that we are working on a Github repos search app where the state is a repos array which we fetch matching some query using Github API.

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

In the example above, we create a store object which stores the app state and provide read-only access to it. State property uses *@Published* property wrapper, which notifies *SwiftUI* during any changes. It allows us to keep up to date the whole app by deriving it from a single source of truth. We already talked about store objects in the previous posts, to learn more you can check ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

#### Actions, Mutations, and Reducer
It's time to talk about user actions which lead to state mutations. *Action* is a simple enum or composition of enums describing user actions like repos search, add to favorites, fork the repo, etc. *Mutation* is another enum type which describes a state change. For example, set loading value during data fetch, assign fetched repositories to a state's property, etc. *Mutation* usually comes from *Action*. Let's take a looks at an example code for *Action and Mutation* enums.

```swift
enum AppMutation {
    case setSearchResults(repos: [Repo])
}

protocol Action {
    associatedtype Mutation
    func mapToMutation() -> Mutation
}

enum AppAction: Action {
    case search(query: String)

    func mapToMutation() -> Mutation {
        // implement your mapping here
    }
}
```

*Reducer* is a function which takes current state, applies *mutation* to the state, and generates a new state. Generally, *reducer* or *composition of reducers* is the single place where your app mutates the state. The fact that the only one function can modify the whole app state is super simple, debuggable, and testable. Here is an example of our reduce function.

```swift
typealias Reducer<State, Mutation> = (inout State, Mutation) -> Void

let appReducer: Reducer<AppState, AppMutation> = { state, mutation in
    switch mutation {
    case let .setSearchResults(repos):
        state.searchResult = repos
    }
}
```

#### Unidirectional flow
Let's talk about a data flow now. Every view has read-only access to a state via store object. Views can send *actions* to a store object. Store object generates a *mutation* from the *action* and passes it to the *reducer*. *Reducer* modifies the state, and then *SwiftUI* notifies all the views about state changes. *SwiftUI* has a super-efficient diffing algorithm that's why diffing of the whole app state and updating changed views works very fast.

**State -> View -> Action -> State -> View**

This architecture revolves around a strict **unidirectional** data flow. It means that all data in an application follows the same lifecycle pattern, making the logic of your app more predictable and easier to understand. Let's modify our store object to support sending *actions* and state *mutations* via *reducer*.

```swift
final class Store<AppState, AppAction: Action>: ObservableObject {
    @Published private(set) var state: AppState

    private let appReducer: Reducer<AppState, AppAction.Mutation>

    init(
        initialState: AppState,
        appReducer: @escaping Reducer<AppState, AppAction.Mutation>
    ) {
        self.state = initialState
        self.appReducer = appReducer
    }

    func send(_ action: AppAction) {
        let mutation = action.mapToMutation()
        appReducer(&state, mutation)
    }
}
```

#### Side effects
We already implemented a *unidirectional* flow which accepts user actions and modifies the state, but what about *async action* which we usually call *side effects*. How to add support for an async task to our store type? I think it is a time to introduce the usage of *Combine framework* which perfectly fits async task processing.

```swift
import Foundation
import Combine

protocol Action {
    associatedtype Mutation
    func mapToMutation() -> AnyPublisher<Mutation, Never>
}

enum AppAction: Action {
    case search(query: String)

    func mapToMutation() -> AnyPublisher<AppMutation, Never> {
        switch self {
        case let .search(query):
            return dependencies.githubService
                .searchPublisher(matching: query)
                .replaceError(with: [])
                .map { AppMutation.searchResults(repos: $0) }
                .eraseToAnyPublisher()
        }
    }
}
```

We add support for *async tasks* by changing the signature of the *Action* protocol. Instead of mapping *State* to a *Mutation*, we map it to a *Mutation Publisher*. It allows us to handle async jobs using *Combine* and then publish a *mutation* which will be used by *reducer* to apply on the current state.

```swift
typealias Reducer<State, Mutation> = (inout State, Mutation) -> Void

final class Store<AppState, AppAction: Action>: ObservableObject {
    @Published private(set) var state: AppState

    private let appReducer: Reducer<AppState, AppAction.Mutation>
    private var cancellables: Set<AnyCancellable> = []

    init(
        initialState: AppState,
        appReducer: @escaping Reducer<AppState, AppAction.Mutation>
    ) {
        self.state = initialState
        self.appReducer = appReducer
    }

    func send(_ action: AppAction) {
        action
            .mapToMutation()
            .receive(on: DispatchQueue.main)
            .sink { self.appReducer(&self.state, $0) }
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
        store.send(.search(query: query))
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

