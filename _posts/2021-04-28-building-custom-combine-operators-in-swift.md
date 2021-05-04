---
title: Building custom Combine operators in Swift
layout: post
image: /public/combine.png
category: Combine framework
---
Combine looks like a very sophisticated framework and provides you all the needed things you might need to process your data. It comes with many valuable operators like map, filter, and reduce. This week we will learn how to build new operators that we might miss from the default package.

{% include friends.html %}

Rather than implementing the *Publisher* protocol yourself, you can create your own operator using composition and several standard operators and publishers provided by the Combine framework. Let's start with the simplest one.

#### Replace error or empty with value
The Combine framework has *replaceEmpty* and *replaceError* operators that we can use to inject the value into an empty publisher or replace the error with a value. I need both of them very often, and instead of typing these two operators every time, we can create a new one that combines them.

```swift
extension Publisher {
    func replaceErrorOrEmpty(with output: Output) -> AnyPublisher<Output, Never> {
        self
            .replaceEmpty(with: output)
            .replaceError(with: output)
            .eraseToAnyPublisher()
    }
}
```

As you can see in the example above, we can create an extension for *Publisher* type in a straightforward way and add the functionality we need by composing other operators. Now we can use the new operator as we use standard ones.

```swift
Reducer { state, action, environment in
    switch action {
    case .fetch:
        return environment.healthService
            .authorize()
            .replaceErrorOrEmpty(with: false)
            .map(AppAction.setAuthStatus)
            .eraseToAnyPublisher()
    }
}
```

> To learn more about the set of operators that the Combine framework provides us, take a look at my ["Catching errors in Combine"](/2020/04/22/catching-errors-in-combine/) post.

#### Finish on fail
Another helpful operator might be finish on fail. There are different circumstances where you don't need to handle the failure and want to finish silently. There is not standard operator for that in the Combine framework, but we can quickly achieve it by using the *catch* operator and *Empty* publisher.

```swift
extension Publisher {
    func finishOnFail() -> AnyPublisher<Output, Never> {
        self
            .catch { _ in Empty() }
            .eraseToAnyPublisher()
    }
}
```

Here we have another extension on *Publisher* type that adds an opportunity to finish the publisher without emitting an error. I usually use the *finishOnFail* operator in conjunction with the *Merge* publisher. For example, on every app launch, I start a network request to fetch the latest data, but at the same time, I fetch and display locally cached data. In this case, I don't worry if my network request fails or not because I have the data to show.

```swift
func newsReducer(
    state: State,
    action: Action,
    environment: Environment
) -> AnyPublisher<Action, Never> {
    switch action {
    case .fetch:
        return Publishers.Merge(
            environment.newsService.fetchCachedNews(),
            environment.newsService.fetchRemoteNews()
                .finishOnFail()
        )
        .map(Action.setNews)
        .eraseToAnyPublisher()
    default: 
        return Empty().eraseToAnyPublisher()
    }
}
```

#### Catch result
OK, we know how to ignore errors, but we should at least handle them. In many cases, an error can be a part of the app state that why we should not ignore it. Instead, we have to store and display it correctly. Let's first build an operator which we can use to wrap the value and error into a *Result* type that the publisher emits instead of plain value.

```swift
extension Publisher {
    func catchResult() -> AnyPublisher<Result<Output, Failure>, Never> {
        self
            .map(Result.success)
            .catch { Just(Result.failure($0)) }
            .eraseToAnyPublisher()
    }
}
```

Assume that we have a single state container that stores the whole app state. There is a dedicated field defining the state of an authorization request as an instance of *Result* enum. Let's see how it might look in code.

```swift
struct State {
    var auth: Result<Bool, Error>
}

enum Action {
    case fetchAuth
    case setAuth(Result<Bool, Error>)
}

func authReducer(
    state: inout State,
    action: Action,
    environment: Environment
) -> AnyPublisher<Action, Never> {
    switch action {
    case .fetchAuth:
        return environment.service
            .authorize()
            .catchResult()
            .map(Action.setAuth)
            .eraseToAnyPublisher()
    case .setAuth(let result):
        state.auth = result
        return Empty()
            .eraseToAnyPublisher()
    }
}
```

Side effects returned by reducers should never fail. Even if it fails under the hood, it should emit an action that defines a failure. It is a perfect use case for our new *catchResult* operator.

> If you are not familiar with the concept of a single source of truth, take a look at my dedicated series of ["Redux-like state container in SwiftUI"](/2019/09/18/redux-like-state-container-in-swiftui/) posts.

#### Conclusion
The Combine framework is a great tool to handle asynchronous operations in your app. It provides you with tons of operators to transform your data, but it is also effortless to extend it using the composition of standard operators that we learned today. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!