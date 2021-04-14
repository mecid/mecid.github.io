---
title: Building type-safe networking in Swift
layout: post
image: /public/networking.jpg
category: Architecture
---

More than half of the apps I built during my career had networking code. Usually, we build apps for web services. Today we will talk about building the type-safe networking layer using Swift language features like enums, phantom types, and extensions.

{% include friends.html %}

#### Basics
Let's first take a look at the typical networking code and recognize the issues that we want to avoid in our solution.

```swift
struct Repo: Decodable {
    let id: Int
    let name: String
}

struct SearchResponse: Decodable {
    let items: [Repo]
}

let query = "Swift"
let url = URL(string: "https://api.github.com/search/repositories?q=\(query)")!
var request = URLRequest(url: url)
request.httpMethod = "GET"
request.httpBody = nil

let cancellable = URLSession.shared.dataTaskPublisher(for: request)
    .map(\.data)
    .decode(type: SearchResponse.self, decoder: JSONDecoder())
    .map(\.items)
    .replaceError(with: [])
    .sink { print($0) }
```

In the example above, we have a standard networking code that creates the request and decodes the response. There are a few things that I want to avoid in the future.
1. We create a GET request, but there is a possibility to set an HTTP body for a GET request, which looks meaningless.
2. We try to decode the response by providing the type of resulting data. Usually, every request has one and only one type that we can obtain. In this case, it is better to have the response type encoded into the request.

Let's start building our type-safe networking by introducing the *Request* type. The *Request* type should contain the URL we need to access, headers, and HTTP method.

```swift
struct Request {
    let url: URL
    let method: HttpMethod
    var headers: [String: String] = [:]
}
```

The HTTP method is an exclusive thing. You can't use both GET and POST, and you can choose only one HTTP method. It looks like a perfect use-case for an enum type.

```swift
enum HttpMethod: Equatable {
    case get([URLQueryItem])
    case put(Data?)
    case post(Data?)
    case delete
    case head

    var name: String {
        switch self {
        case .get: return "GET"
        case .put: return "PUT"
        case .post: return "POST"
        case .delete: return "DELETE"
        case .head: return "HEAD"
        }
    }
}
```

As you can see in the example above, we define the *HTTPMethod* enum that describes various HTTP methods. We use cases with associated types to hold the data correlated with the HTTP method. For example, the GET method contains URL query items, POST and PUT methods have the data we use as the HTTP body.

Now we need to make somehow *URLSession* working with our *Request* type. The easiest way is defining a calculated property on the *Request* type that converts it to the *URLRequest*.

```swift
extension Request {
    var urlRequest: URLRequest {
        var request = URLRequest(url: url)

        switch method {
        case .post(let data), .put(let data):
            request.httpBody = data
        case let .get(queryItems):
            var components = URLComponents(url: url, resolvingAgainstBaseURL: false)
            components?.queryItems = queryItems
            guard let url = components?.url else {
                preconditionFailure("Couldn't create a url from components...")
            }
            request = URLRequest(url: url)
        default:
            break
        }

        request.allHTTPHeaderFields = headers
        request.httpMethod = method.name
        return request
    }
}
```

Finally, we can create an extension on *URLSession* to make requests with our new type.

```swift
extension URLSession {
    func publisher(for request: Request) -> AnyPublisher<Data, Error> {
        dataTaskPublisher(for: request.urlRequest)
            .map(\.data)
            .mapError { $0 }
            .eraseToAnyPublisher()
    }
}
```

#### Phantom response type
One thing we forgot to do is encoding of result type into the request. We can make it really easy by introducing phantom type. Phantom type is a generic constraint defined in any type but is not used inside. Let's take a look at the example.

```swift
struct Request<Response> {
    let url: URL
    let method: HttpMethod
    var headers: [String: String] = [:]
}
```

As you can see, we define *Response* type, but we didn't use it anywhere in the *Request* type. That's why it is called phantom type. Defining phantom types allows us to store information about the response in our request type. For example, it might be a type conforming to *Decodable* or even an instance of the *Data* type. Let's update our extension on *URLSession* to support response decoding.

```swift
extension URLSession {
    enum Error: Swift.Error {
        case networking(URLError)
        case decoding(Swift.Error)
    }

    func publisher(
        for request: Request<Data>
    ) -> AnyPublisher<Data, Swift.Error> {
        dataTaskPublisher(for: request.urlRequest)
            .mapError(Error.networking)
            .map(\.data)
            .eraseToAnyPublisher()
    }

    func publisher(
        for resource: Resource<URLResponse>
    ) -> AnyPublisher<URLResponse, Swift.Error> {
        dataTaskPublisher(for: resource.urlRequest)
            .mapError(Error.networking)
            .map(\.response)
            .eraseToAnyPublisher()
    }

    func publisher<Value: Decodable>(
        for request: Request<Value>,
        using decoder: JSONDecoder = .init()
    ) -> AnyPublisher<Value, Swift.Error> {
        dataTaskPublisher(for: request.urlRequest)
            .mapError(Error.networking)
            .map(\.data)
            .decode(type: Value.self, decoder: decoder)
            .mapError(Error.decoding)
            .eraseToAnyPublisher()
    }
}
```

In the example above, we also introduced proper error management to handle errors more gracefully. Let's define a Github repo search request using the new API.

```swift
extension Request where Response == SearchResponse {
    static func search(matching query: String) -> Self {
        Request(
            url: URL(string: "https://api.github.com/search/repositories")!,
            method: .get(
                [.init(name: "q", value: query)]
            )
        )
    }
}

let request: Request<SearchResponse> = .search(matching: "Swift")
let cancellable = URLSession.shared.publisher(for: request)
    .map(\.items)
    .replaceError(with: [])
    .sink { print($0) }
```

> To learn more about the benefits of using phantom types, look at my ["Phantom types in Swift"](/2021/02/18/phantom-types-in-swift/) post.

#### Conclusion
Today we built a type-safe networking layer using Swift features like enums, phantom types, and extensions. This toolbox allows you to transform any old API into a safe and modern API. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!