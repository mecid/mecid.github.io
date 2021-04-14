---
title: Pattern Matching with case let
layout: post
category: Swift Language Features
---

Today we will talk about Pattern Matching, one of my favorite features in Swift. Pattern Matching is the act of checking a given sequence of tokens for the presence of the constituents of some pattern. Swift has a particular keyword for applying Pattern Matching: **case let**. Let's dive into examples.

{% include friends.html %}

#### Enums
Pattern Matching is very useful while working with enums. As a part of ["Maintaining State in Your ViewControllers" post](/2019/01/23/maintaining-state-in-view-controllers/), we talk about State enum, which describes the state of ViewController. Let's see how we can efficiently use Pattern Matching with it.

```swift
enum State<Data> {
    case loading
    case loaded(Data)
    case failed(Error)
}

switch state {
case .loading: renderLoading()
case let .loaded(shows) where shows.isEmpty: renderEmptyView()
case let .loaded(shows): render(shows)
case let .failed(error): render(error)
}
```

While regular switching on enum with associated values we can also use **case let** keyword to match it to some pattern and assign associated value to a variable. Another beautiful option here is filtering associated value by using where keyword.

#### Optionals
Optional in Swift is the enum with two cases, so you can apply Pattern Matching as we do it before with enums. But in the case of optionals, we have some additional features. Let's check the example code.

```swift
let value: Int? = 10

switch value {
case let value? where value > 10: print("higher than ten")
case let .some(value): print(value)
case .none: print("none")
}
```

value? here means the non-nil value of optional. So, it is the same as .some(value).

#### Tuples
Another good use case for Pattern Matching can be tuples. Tuples often used as lightweight types for grouping some data. Let's see how we can use Pattern Matching on a tuple which presents authentication data.

```swift 
let auth = (username: "majid", password: "veryStrongPassword")

switch auth {
case ("admin", "admin"): renderAdmin()
case let (_, password) where password.count < 6: renderShortPasswordMessage()
case let (username, password): renderUserProfile(username, password)
}
```

As you can see, we can apply to tuples all the matching features which we used with enums. We can also match the particular value to the tuple as we do for matching admin data.

#### case let with flow statements
I want to mention that you can easily use **case let** keyword with any flow control statement, let's see how we can use it with if condition statements.

```swift
if case let .loaded(shows) = state, shows.isEmpty {
    renderEmptyView()
}
```

Same is possible with guard statement.

```swift
guard case let .loaded(shows) = state, shows.isEmpty else {
    // Do something here
}
```

Another compelling **case let** usage is possible with for-in loops, we can easily filter items.

```swift 
let stateHistory: [State<[Show]>] = [.loaded([]), .loading]
for case let .loaded(shows) in stateHistory where shows.count > 2 {
    // Do something here
}
```

#### Conclusion
Today we discussed how powerful can be Pattern Matching in Swift, and how we can use it in daily development. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!