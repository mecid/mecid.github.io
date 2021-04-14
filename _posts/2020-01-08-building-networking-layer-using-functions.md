---
title: Building networking layer using functions
layout: post
image: /public/networking.png
category: Architecture
---

This week I want to talk about building a networking layer in Swift using *Functional programming*. *Functional programming* is a way of making programs using *pure functions and function composition*. Let's see how we can use it to build a flexible and composable network layer.

{% include friends.html %}

Usually, Swift developers use the *Protocol-Oriented* style of programming to build any abstractions like the networking layer. In most of the cases, protocols generate more boilerplate than needed. Let's instead model our networking layer using a pure function and function composition. 

#### Pure functions
Pure functions calculate output using input and don't affect or rely on any state outside itself. It means pure function takes an argument to transform and return a new value. Here is a very simple example of a pure function that takes two arguments, sums them, and returns a new value.

```swift
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}
```

We know how to use pure functions. Let's model our networking layer using a function. Usually, we need a way of transforming URL request into raw data and HTTP response.

```swift
typealias Networking = (URLRequest) ->
    Result<(data: Data, response: URLResponse), Error>
```

Networking function is a function that accepts a request and returns an error or response. We want to do our networking asynchronously, and we can achieve it by using the *Combine* framework. Let's change our networking function definition to use *Publisher* instead of the Result type. 

```swift
typealias Networking = (URLRequest) ->
    AnyPublisher<(data: Data, response: URLResponse), Error>
```

You should be familiar with this type alias. *URLSession* class has a similar type for its *dataTaskPublisher* method. We can create an extension for *URLSession* to create a new method that *"conforms"* to our *Networking* type.

```swift
extension URLSession {
    func erasedDataTaskPublisher(
        for request: URLRequest
    ) -> AnyPublisher<(data: Data, response: URLResponse), Error> {
        dataTaskPublisher(for: request)
            .mapError { $0 }
            .eraseToAnyPublisher()
    }
}
```

Now let's see how we can inject our networking function into some service layer objects which use the network to load and decode domain specific objects.

```swift
struct FeedService {
    let networking: Networking

    func fetchFeed() -> AnyPublisher<Feed, Error> {
        networking(.feed())
            .map { $0.data }
            .decode(type: Feed.self, decoder: JSONDecoder())
            .eraseToAnyPublisher()
    }
}

let feedService = FeedService(networking: URLSession.shared.erasedDataTaskPublisher)
feedService.fetchFeed()
```

One of the benefits of this approach is the ability to replace our real-world networking function with any mock implementation. Let's see how we can do that.

```swift
func mockNetworking(
    data: Data = .init(),
    response: URLResponse = .init()
) -> Networking {
    return { _ in
        Just((data: data, response: response))
            .setFailureType(to: Error.self)
            .eraseToAnyPublisher()
    }
}

let mockedFeedService = FeedService(networking: mockNetworking())
```

In the example above, we use one of the powerful techniques from functional programming called **partial function application**. Partial application refers to the process of fixing a number of arguments to a function, producing another function of smaller arity. In other words, we create a function that returns another function that captures arguments and can use them in its own body.

#### Function composition
Assume that you need to add an OAuth token header to every request that you run, but you don't want to pass it every time while creating a request. 

We can use function composition to solve our problem. Function composition is a way of generating new functions by chaining two or more other functions.

1. We need a function that will modify our requests by adding OAuth token to the headers.
2. We need a new function that composes a header modifier function with our old networking function.

```swift
func tokenRequestModifier(_ token: String) -> (URLRequest) -> URLRequest {
    return { request in
        var request = request
        request.addValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        return request
    }
}
```

In the example above, we have *tokenRequestModifier* function. Again, we use partial application techniques to fix the token parameter and generate a new request modifier function. Now we can compose it with our networking function to create a brand new authorized networking function.

```swift
func compose<A, B, C>(
    _ f: @escaping (A) -> B,
    _ g: @escaping (B) -> C
) -> (A) -> C {
    return { g(f($0)) }
}

let token = UserDefaults.standard.string(forKey: "token") ?? ""
let authorizedNetworking = compose(
    tokenRequestModifier(token),
    networking
)

authorizedNetworking(.feed())
```

Here we compose two functions: *tokenRequestModifier* and *networking* to create a brand new authorized networking function. Function composition allows us to run *tokenRequestModifier* before every networking request, which we run via authorized networking function.

#### Conclusion
Today we talked about building a networking layer in a very functional way. I'm not saying that protocols are bad or something similar. Protocols are awesome when you need to build a huge hierarchy of types like collections that we have in Swift standard library. 

But usually, we have mocked and production implementation, and this is the case where functions are more than enough. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!