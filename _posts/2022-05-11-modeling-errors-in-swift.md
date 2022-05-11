---
title: Modeling errors in Swift
layout: post
category: Swift Language Features
image: /public/swift.png
---

The new Swift Concurrency feature doesn't only bring new opportunities for writing safer and more maintainable async code but also changes the way we handle errors. I didn't use *throw-catch* keywords too much in my legacy code because usually, I had a completion callback with the *Result* type handled by the *switch* operator. This week we will talk about modeling error types and how we will address them in Swift with *throw-catch* keywords.

#### Basics
Let's look at a small but typical example of error handling in Swift. Here is the in-memory cache implementation that we can use in our apps to store some data by key. There is an option to limit the capacity of the in-memory cache and the list of errors that a cache instance can throw.

We use an enum to define the list of exceptional situations. An enum type has an excellent fit for mutually exclusive cases. All you need to do to throw an error in Swift is to call **throw** with an instance of any type conforming to the *Error* protocol.

```swift
actor InMemoryCache<Key: Hashable & Codable, Value: Codable> {
    enum ErrorKind: Error {
        case noValue(Key)
        case outOfMemory(availableBytes: Int)
    }

    private var memoryLimit: Int
    init(memoryLimit: Int) {
        self.memoryLimit = memoryLimit
    }

    func get(for key: Key) throws -> Value {
        guard contans(key: key) else {
            throw ErrorKind.noValue(key)
        }

        // fetch and return value here
    }

    func set(value: Value, for key: Key) throws -> Void {
        let size = calculateSize(for: value)
        let availableSpace = calculateAvailableSpace()

        guard size <= availableSpace else {
            throw ErrorKind.outOfMemory(availableBytes: availableSpace)
        }

        memoryLimit -= size
        // store value here
    }

    func deleteValue(for key: Key) throws -> Void {
        guard contans(key: key) else {
            throw ErrorKind.noValue(key)
        }

        let value = try get(for: key)
        let size = calculateSize(for: value)
        memoryLimit += size
        // delete value here
    }

    // more code here
}
```

At first glance, the API we have modeled here looks nice, but we will see all the issues as soon as we start using it.

```swift
final class APIClient {
    typealias Cache = InMemoryCache<URL, User>
    
    private static let logger = Logger(
        subsystem: Bundle.main.bundleIdentifier!,
        category: String(describing: APIClient.self)
    )
    
    private let cache = Cache(memoryLimit: 1_000)

    func fetchUser(from url: URL) async throws -> User {
        do {
            let user = try await cache.get(for: url)
            return user
        } catch Cache.ErrorKind.noValue {
            do {
                let user = try await fetchRemoteUser(from: url)
                try await cache.set(value: user, for: url)
            } catch Cache.ErrorKind.outOfMemory {
                Self.logger.warning("Cache is full")
            } catch {
                // Catch the errors occuring while
                // fetching user from the remote server
            }
        }
    }

    private func fetchRemoteUser(from url: URL) async throws -> User {
        // ...
    }
}
```

As you can see in the example above, we implement the *APIClient* class that uses our *InMemoryCache* class to store downloaded users in memory. The code here really smells, it has a bunch of nested do-catch blocks, and the code path is confusing. Let's refactor our *InMemoryCache* class to make it friendlier for client code.

> To learn more about logging in Swift, take a look at my ["Logging in Swift"](/2022/04/06/logging-in-swift/) post.

We will refactor our error modeling code using three principles described in "A philosophy of software design" book written by John Ousterhout. I enjoyed reading it and can recommend it to you without any doubt.

#### Define errors out of existence
Sometimes we define too many errors. We define an error as a situation that is not an error at all. For example, in our case, we have the *noValue* case. Instead of throwing the error whenever a value is not available, we can use optionals and silently return **nil** without disrupting the execution with an error.

```swift
func get(for key: Key) -> Value? {
    guard contans(key: key) else {
        return nil
    }

    // fetch value here
}
```

#### Mask exception
We should understand that we do not need to propagate every error to the high-level client code. Sometimes we can easily catch and solve it on the lower level, and the client code will not suffer from handling all the possible error cases.

```swift
import CloudKit

final class CloudService {
    private static let logger = Logger(
        subsystem: Bundle.main.bundleIdentifier!,
        category: String(describing: CloudService.self)
    )

    private let container = CKContainer.default()
    private var pending: Set<CKRecord> = []

    func save(_ user: CKRecord) async throws {
        pending.insert(user)

        while let user = pending.popFirst() {
            do {
                try await container.privateCloudDatabase.save(user)
            } catch CKError.networkUnavailable {
                pending.insert(user)
                break
            }
        }
    }
}
```

#### Error aggregation
Not every error has its unique handler. There is a set of errors in our apps that only can be logged or presented using an alert. There is no need to create an entire case for every error. Sometimes we can generalize errors by using a single case with different messages.

```swift
actor InMemoryCache<Key: Hashable & Codable, Value: Codable> {
    enum ErrorKind: Error {
        case general(String)
        case outOfMemory
    }

    private var storage: [Key: Value] = [:]
    private var memoryLimit: Int
    init(memoryLimit: Int) {
        self.memoryLimit = memoryLimit
    }

    func swapToDisk() async throws -> Void {
        let encoder = JSONEncoder()
        do {
            try encoder.encode(storage)
        } catch {
            throw ErrorKind.general(error.localizedDescription)
        }
        // ...
    }

    func loadFromDisk() async throws -> Void {
        let data: Data = // ...
        let decoder = JSONDecoder()
        do {
            storage = try decoder.decode([Key: Value].self, from: data)
        } catch {
            throw ErrorKind.general(error.localizedDescription)
        }
    }
    
    // more code here
}
```

#### Conclusion
Today we learned how to model errors in Swift using three different ways. Error handling is a complex topic, and we should treat it carefully while designing our APIs. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
