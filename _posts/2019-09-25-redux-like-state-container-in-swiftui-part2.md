---
title: Redux-like state container in SwiftUI. Best practices.
layout: post
category: Architecture
image: /public/store.png
---

Last week we talked about [building a state container similar to Redux in SwiftUI](/2019/09/18/redux-like-state-container-in-swiftui/). *Redux* provides a single source of truth, which eliminates tons of bugs produced by multiple states across the app. This week we will talk about best practices in building *Redux-based* apps which allows us to keep our codebase simple and clean.

{% include friends.html %}

#### State normalization
*Redux* stores the whole app's state as a single source of truth. It allows us to keep our *User Interface* in sync with the app state. But to achieve this, we have to normalize our state. Let's take a look at the example.

```swift
struct AppState {
    var allTasks: [Task]
    var favorited: [Task]
}
```

Here we have an *AppState* struct which stores a task list and favorited tasks. It looks straightforward, but it has one big downside. Assume that you have the edit task screen where you can modify the selected task. Whenever the user hits the save button, you have to find and update a particular task in the allTasks list and favorited list. It can be error-prone and lead to a performance issue as soon as you have a long list of tasks.

Let's improve performance by normalizing our state struct. First of all, we need to store our tasks in *Dictionary* where task id is the key and task itself is the value. *Dictionary* can retrieve the value by key in constant **(O(1))** time, but it doesn't keep the order. We can create an array with ids to save the order. Let's take a look at the normalized version of our state.

```swift
struct AppState {
    var tasks: [Int: Task]
    var allTasks: [Int]
    var favorited: [Int]
}
```

As you can see in the example above, we store our tasks in the *Dictionary* where task *id* is the key, and the task is the value. We also store arrays of identifiers for all tasks and favorited ones. By using identifiers instead of copies, we achieve a centralized state persistence which keeps our *User Interface* and data in sync.

#### State composition
It is very natural to store your app's state as a single struct, but it simply can blow up as soon as you add more and more fields to your state struct. We can use state composition to solve this issue. Let's take a look at the example.

```swift
struct AppState {
    var calendar: CalendarState
    var trends: TrendsState
    var settings: SettingState
}
```

In the example above, we divide our state into three dedicated pieces and compose them into *AppState*.

#### Reducer composition
Another important component of our *Redux-like* state container is *Reducer*. We can extract and compose it as we do with state struct. It will allow us to respect a *Single Responsibility* principle and keep our reducers small and clean.

```swift
enum AppAction {
    case calendar(action: CalendarAction)
    case trends(action: TrendsAction)
}

func trendsReducer(
    state: inout TrendsState,
    action: TrendsAction
) -> AnyPublisher<TrendsAction, Never> {
    // Implement your state changes here
}

func calendarReducer(
    state: inout CalendarState,
    action: CalendarAction
) -> AnyPublisher<CalendarAction, Never>{
    // Implement your state changes here
}

func appReducer(
    state: inout AppState,
    action: AppAction
) -> AnyPublisher<AppAction, Never> {
    switch action {
    case let .calendar(action):
        return calendarReducer(&state.calendar, action)
            .map(AppAction.calendar)
            .eraseToAnyPublisher()
    case let .trends(action):
        trendsReducer(&state.trends, action)
            .map(AppAction.trends)
            .eraseToAnyPublisher()
    }

    return Empty().eraseToAnyPublisher()
}
```

#### Derived stores
Another way of composition that should simplify our architecture is derived stores. I don't want to expose the whole app state to every view or update views on not related state updates. Here is a small example from my CardioBot app.

```swift
import SwiftUI

struct RootView: View {
    @EnvironmentObject var store: Store<AppState, AppAction>

    var body: some View {
        TabView {
            NavigationView {
                SummaryContainerView()
                    .navigationBarTitle("today")
                    .environmentObject(
                        store.derived(
                            deriveState: \.summary,
                            deriveAction: AppAction.summary
                        )
                )
            }.tabItem {
                Image(systemName: "heart.fill")
                    .imageScale(.large)
                Text("today")
            }

            NavigationView {
                TrendsContainerView()
                    .navigationBarTitle("trends")
                    .environmentObject(
                        store.derived(
                            deriveState: \.trends,
                            deriveAction: AppAction.trends
                        )
                )
            }.tabItem {
                Image(systemName: "chevron.up.circle.fill")
                    .imageScale(.large)
                Text("trends")
            }
        }
    }
}
```

As you can see, every tab of my app gets its part of the state via the derived store. We still use the global store to handle all the state mutation. Derived store works as a pipeline that allows us to transform the state from the global store and redirect actions to the global store. Let's take a look at how we can implement the *derived* method for our *Store* class.

```swift
func derived<DerivedState: Equatable, ExtractedAction>(
    deriveState: @escaping (State) -> DerivedState,
    embedAction: @escaping (ExtractedAction) -> Action
) -> Store<DerivedState, ExtractedAction> {
    let store = Store<DerivedState, ExtractedAction>(
        initialState: deriveState(state),
        reducer: Reducer { _, action, _ in
            self.send(embedAction(action))
            return Empty().eraseToAnyPublisher()
        },
        environment: ()
    )
    $state
        .map(deriveState)
        .removeDuplicates()
        .receive(on: DispatchQueue.main)
        .assign(to: &store.$state)
    return store
}
```

#### Conclusion
Today we talked about two important strategies which we should use during app development using *Redux-like* state containers in SwiftUI. Both normalization and composition keep our app state simple and maintainable. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 

1. [Redux-like state container in SwiftUI. Basics](/2019/09/18/redux-like-state-container-in-swiftui/)
2. Redux-like state container in SwiftUI. Best practices
3. [Redux-like state container in SwiftUI. Container Views.](/2019/10/02/redux-like-state-container-in-swiftui-part3/)
4. [Redux-like state container in SwiftUI. Connectors.](/2021/02/03/redux-like-state-container-in-swiftui-part4/)

#### References
The series of posts have built on a foundation of ideas started by other libraries, particularly Redux, Elm, and TCA.
0. [WWDC20 - Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/)
1. [Redux](https://redux.js.org) - The JavaScript library that popularized unidirectional data flow.
2. [The Elm Architecture](https://guide.elm-lang.org/architecture/) - A purely functional language and runtime that inspired the creation of Redux.
3. [The Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture) - A library that bridges concepts from the Elm Architecture and Redux to Swift. It introduced the “environment” and “effect” patterns that this series covers.

