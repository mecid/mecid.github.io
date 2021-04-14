---
title: Asynchronous completion handlers with Result type
layout: post
category: Swift Language Features
---

[Enums are one of my favorite features in Swift language.](/2019/01/23/maintaining-state-in-view-controllers/) This week we will talk about Result enum, which had been a part of the standard library since Swift 5. With Result enum, we can easily describe the resulting state of an asynchronous operation. It can be success or failure at one time not both of them. Let's take a look at Result enum definition in the Swift standard library.

{% include friends.html %}

```swift
public enum Result<Success, Failure> where Failure : Error {
    case success(Success)
    case failure(Failure)
```

Result type described as two case enum, which has success and failure cases. Both of them have generic associated types, while Failure type is constrained to conform Error protocol, Success type can be anything that we want to return as a proper result of our operation. Let's take a look at completion handler in the URLSession's dataTask function which passes both data and error to the handler.

```swift
URLSession.shared.dataTask(with: API.history) { data, _ , error in
}
```

The downside of this approach is the undefined state where we have both data and error in the completion handler. So let's clarify completion handler by using Result type instead.

```swift
typealias Handler<T> = (Result<T, Error>) -> Void

extension URLSession {
    func dataTask(with url: URL, completionHandler: @escaping Handler<Data>) {
        dataTask(with: url) { data, _, error in
            if let error = error {
                completionHandler(.failure(error))
            } else {
                completionHandler(.success(data ?? Data()))
            }
        }
    }
}
```

Here we have an extension on the URLSession class which adds dataTask method overload. Instead of passing both data and error, we give the instance of Result enum which stores data value or error. I am using Result enum in many places across my codebase, that's why I created type alias for Handler type which is closure with a generic Result as a parameter. Let's move to the usage of our new extension.

```swift
class HistoryService {
    private let session: URLSession
    private let decoder: JSONDecoder

    init(session: URLSession, decoder: JSONDecoder) {
        self.session = session
        self.decoder = decoder
    }

    func fetch(handler: @escaping Handler<History>) {
        session.dataTask(with: API.history) { [weak self] result in
            guard let self = self else { return }

            do {
                let data = try result.get()
                let user = try self.decoder.decode(History.self, from: data)
                handler(.success(user))
            } catch {
                handler(.failure(error))
            }
        }
    }
}
```

In the code samples above we have a HistoryService class which uses URLSession to fetch data and deserialize it into History structure instance. Result type provides the particular *get* method which tries to return the value of Result enum or throws the error. I feel like I have a lot of places across my codebase where I need to fetch data and deserialize into some structure. We can easily create another extension, this time extension on Result type.

```swift
extension Result where Success == Data {
    func decode<T: Decodable>(with decoder: JSONDecoder = .init()) -> Result<T, Error> {
        do {
            let data = try get()
            let decoded = try decoder.decode(T.self, from: data)
            return .success(decoded)
        } catch {
            return .failure(error)
        }
    }
}
```

The extension which we have above tries to decode data into decodable generic by returning value wrapped into Result type or by returning failure with error. Here is the new version of the HistoryService which uses our extension. One of the benefits here is the type inference, which saves us from indicating type in which we are going to decode data.  Decode function uses generic constraint which infers from the completion handler definition. Now it looks in a very nice way.

```swift
func fetch(handler: @escaping Handler<History>) {
    session.dataTask(with: API.history) { result in
        handler(result.decode())
    }
}
```
#### Conclusion
This week we talked about Result type which comes with Swift 5 standard library. It helps us to make our codebase cleaner and easy to understand. I think it is a perfect time to move our asynchronous code to use Result enum for completion handlers. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!