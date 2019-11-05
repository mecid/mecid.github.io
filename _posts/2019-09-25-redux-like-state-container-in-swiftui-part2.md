---
title: Redux-like state container in SwiftUI. Best practices.
layout: post
---

Last week we talked about [building a state container similar to Redux in SwiftUI](/2019/09/18/redux-like-state-container-in-swiftui/). *Redux* provides a single source of truth, which eliminates tons of bugs produced by multiple states across the app. This week we will talk about best practices in building *Redux-based* apps which allows us to keep our codebase simple and clean.

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

let trendsReducer: Reducer<TrendsState, TrendsAction> = Reducer { state, action in
    // Implement your state changes here
}

let calendarReducer: Reducer<CalendarState, CalendarAction> = Reducer { state, action in
    // Implement your state changes here
}

let appReducer: Reducer<AppState, AppAction> = Reducer { state, action in
    switch action {
    case let .calendar(action):
        calendarReducer.reduce(&state.calendar, action)
    case let .trends(action):
        trendsReducer.reduce(&state.trends, action)
    }
}
```

#### Conclusion
Today we talked about two important strategies which we should use during app development using *Redux-like* state containers in *SwiftUI*. Both normalization and composition keep our app state simple and maintainable. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 

1. [Redux-like state container in SwiftUI. Basics](/2019/09/18/redux-like-state-container-in-swiftui/)
2. Redux-like state container in SwiftUI. Best practices
3. [Redux-like state container in SwiftUI. Container Views.](/2019/10/02/redux-like-state-container-in-swiftui-part3/)