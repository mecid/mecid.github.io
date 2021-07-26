---
title: Redux-like state container in SwiftUI. Basics.
layout: post
category: Architecture
image: /public/store.png
---

This week we will talk about building a state container similar to *Redux* and *The Elm Architecture* that provides a single source of truth for your app. A single state for the whole app makes it easier to debug and inspect. Single source of truth eliminates tons of bugs produced by creating multiple states across the app.

{% include friends.html %}

#### Single source of truth
The main idea here is describing the whole app state by using a single struct or composition of structs. Assume that we are working on a Github repos search app where the state is the repos array that we fetch matching some query using Github API.

```swift
struct AppState {
    var searchResult: [Repo] = []
}
```

Next step is passing the read-only app state to every view inside the app. The best way for doing that is *SwiftUI's Environment* feature. We can put an object holding the whole app state in the *Environment* of the root view. Root view will share its *Environment* with all child views.

> SwiftUI uses environment to pass system-wide and application-related information. You can also populate environment with your custom objects. To learn more about environment, take a look at my ["The power of Environment in SwiftUI"](/2019/08/21/the-power-of-environment-in-swiftui/) post.

```swift
final class Store: ObservableObject {
    @Published private(set) var state: AppState
}
```

In the example above, we create a store object that stores the app state and provides read-only access to it. State property uses *@Published* property wrapper that notifies SwiftUI during any changes. It allows us to keep up to date the whole app by deriving it from a single source of truth. 

> To learn more you can check ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

#### Reducer and Actions
It's time to talk about user actions which lead to state mutations. *Action* is a simple enum or composition of enums describing a change of the state. For example, set loading value during data fetch, assign fetched repositories to the state property. Let's take a look at the example code for *Action* enum.

```swift
enum AppAction {
    case search(query: String)
    case setSearchResult(repos: [Repo])
}
```

*Reducer* is a function that takes current state, applies *Action* to the state, and generates a new state. Generally, *reducer* or *composition of reducers* is the single place where your app should mutate the state. The fact that the only one function can modify the whole app state is super simple, debuggable, and testable. Here is an example of reduce function.

```swift
typealias Reducer<State, Action> = (inout State, Action) -> Void

func appReducer(state: inout AppState, action: AppAction) {
    switch action {
    case let .setSearchResults(repos):
        state.searchResult = repos
    case let .search(query):
        break
    }
}
```

#### Unidirectional flow
Let's talk about data flow now. Every view has a read-only access to the state via store object. Views can send *actions* to the store object. *Reducer* modifies the state, and then SwiftUI notifies all the views about state changes. SwiftUI has a super-efficient diffing algorithm that's why diffing of the whole app state and updating changed views works so fast. Let's modify our store object to support sending *actions*.

```swift
final class Store<State, Action>: ObservableObject {
    @Published private(set) var state: State

    private let reducer: Reducer<State, Action>

    init(initialState: State, reducer: @escaping Reducer<State, Action>) {
        self.state = initialState
        self.reducer = reducer
    }

    func send(_ action: Action) {
        reducer(&state, action)
    }
}
```

*State -> View -> Action -> State -> View*

This architecture revolves around a strict **unidirectional** data flow. It means that all the data in the application follows the same pattern, making the logic of your app more predictable and easier to understand.

#### Side effects
We already implemented a *unidirectional* flow that accepts user actions and modifies the state, but what about *async actions* which we usually call *side effects*. How to bake a support for async tasks into our store type? I think it is a good time to introduce the usage of *Combine framework* that perfectly fits async task processing.

```swift
typealias Reducer<State, Action, Environment> =
    (inout State, Action, Environment) -> AnyPublisher<Action, Never>?
```

We add support for *async tasks* by changing *Reducer* typealias, it has the additional parameter called *Environment*. *Environment* might be a plain struct that holds all needed dependencies like service and manager classes.

```swift
func appReducer(
    state: inout AppState,
    action: AppAction,
    environment: Environment
) -> AnyPublisher<AppAction, Never>? {
    switch action {
    case let .setSearchResults(repos):
        state.searchResult = repos
    case let .search(query):
        return environment.service
            .searchPublisher(matching: query)
            .replaceError(with: [])
            .map { AppAction.setSearchResults(repos: $0) }
            .eraseToAnyPublisher()
    }
    return nil
}
```

Side Effect is a sequence of *Actions* which we can publish using *Combine* framework's *Publisher* type. It allows us to handle async job and then publish an *action* that will be used by *reducer* to change the current state.

```swift
final class Store<State, Action, Environment>: ObservableObject {
    @Published private(set) var state: State

    private let environment: Environment
    private let reducer: Reducer<State, Action, Environment>
    private var cancellables: Set<AnyCancellable> = []

    init(
        initialState: State,
        reducer: @escaping Reducer<State, Action, Environment>,
        environment: Environment
    ) {
        self.state = initialState
        self.reducer = reducer
        self.environment = environment
    }

    func send(_ action: Action) {
        guard let effect = reducer(&state, action, environment) else {
            return
        }

        effect
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: send)
            .store(in: &cancellables)
    }
}
```

As you can see in the example above, we build a *Store* type that supports async tasks. Usually, reducer resolves an action by applying it on top of the state. In case of an async action, reducer returns it as *Combine Publisher*, then *Store* run it and send result back to the reducer as a plain action.

#### Real usage example
Finally, we can finish our repos search app that calls Github API asynchronously and fetches repositories matching a query. The full source code available on [Github](https://github.com/mecid/redux-like-state-container-in-swiftui).

```swift
typealias AppStore = Store<AppState, AppAction, AppEnvironment>

struct SearchContainerView: View {
    @EnvironmentObject var store: AppStore
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

We divide our screen into two views: *Container View* and *Rendering View*. *Container View* handles the actions and retrieves the needed piece of state from the global state. *Rendering View* accepts the data and renders it. 

> We already talked about *Container Views* in my previous posts, to learn more take a look at ["Introducing Container views in SwiftUI"](/2019/07/31/introducing-container-views-in-swiftui/) post.

#### Conclusion
Today we learned how to build *Redux-like* state container with *side-effects* in mind. To achieve that we used *Combine* framework. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 

1. Redux-like state container in SwiftUI. Basics
2. [Redux-like state container in SwiftUI. Best practices](/2019/09/25/redux-like-state-container-in-swiftui-part2/)
3. [Redux-like state container in SwiftUI. Container Views.](/2019/10/02/redux-like-state-container-in-swiftui-part3/)
4. [Redux-like state container in SwiftUI. Connectors.](/2021/02/03/redux-like-state-container-in-swiftui-part4/)

#### References
The series of posts have built on a foundation of ideas started by other libraries, particularly Redux, Elm, and TCA.
0. [WWDC20 - Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/)
1. [Redux](https://redux.js.org) - The JavaScript library that popularized unidirectional data flow.
2. [The Elm Architecture](https://guide.elm-lang.org/architecture/) - A purely functional language and runtime that inspired the creation of Redux.
3. [The Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture) - A library that bridges concepts from the Elm Architecture and Redux to Swift. It introduced the “environment” and “effect” patterns that this series covers.
