---
title: Storing Codable structs on the disk
layout: post
category: Protocol-Oriented Programming
---

Most of our apps are REST clients for some backends. During the development of this kind of apps, we want to keep it working offline. In this case, we have to cache data somewhere locally on the device to make it readable without an internet connection. 

{% include friends.html %}

Apple provides a CoreData framework, which is the best way to store your app data locally. It has a lot of excellent features which help you to boost development. However, it is tough to use it as a simple cache. Most of the time, we just need to display cached data without any additional manipulations. In my opinion, all we need is pure disk storage. This week we will discuss how easily we can implement straightforward disk store for our Codable structs.

Let's start with defining a couple of protocols for our storage logic. I want to separate access to writable and readable parts of the storage, and this is where we can use protocol composition feature of Swift language.

```swift
import Foundation

typealias Handler<T> = (Result<T, Error>) -> Void

protocol ReadableStorage {
    func fetchValue(for key: String) throws -> Data
    func fetchValue(for key: String, handler: @escaping Handler<Data>)
}

protocol WritableStorage {
    func save(value: Data, for key: String) throws
    func save(value: Data, for key: String, handler: @escaping Handler<Data>)
}

typealias Storage = ReadableStorage & WritableStorage
```

Here we have two protocols describing reading and writing operations on storage. Protocols also provide an asynchronous version for reading and writing actions with completion handlers. We also create typealias Storage, which is a composition of two protocols. Now we can start to work on DiskStorage class which implements our Storage protocols.

```swift
enum StorageError: Error {
    case notFound
    case cantWrite(Error)
}

class DiskStorage {
    private let queue: DispatchQueue
    private let fileManager: FileManager
    private let path: URL

    init(
        path: URL,
        queue: DispatchQueue = .init(label: "DiskCache.Queue"),
        fileManager: FileManager = FileManager.default
    ) {
        self.path = path
        self.queue = queue
        self.fileManager = fileManager
    }
}
```

First of all, let's describe some variables for root path of our storage, DispatchQueue for asynchronous work and FileManager, which we will use to navigate through the file system.

```swift
extension DiskStorage: WritableStorage {
    func save(value: Data, for key: String) throws {
        let url = path.appendingPathComponent(key)
        do {
            try self.createFolders(in: url)
            try value.write(to: url, options: .atomic)
        } catch {
            throw StorageError.cantWrite(error)
        }
    }

    func save(value: Data, for key: String, handler: @escaping Handler<Data>) {
        queue.async {
            do {
                try self.save(value: value, for: key)
                handler(.success(value))
            } catch {
                handler(.failure(error))
            }
        }
    }
}

extension DiskStorage {
    private func createFolders(in url: URL) throws {
        let folderUrl = url.deletingLastPathComponent()
        if !fileManager.fileExists(atPath: folderUrl.path) {
            try fileManager.createDirectory(
                at: folderUrl,
                withIntermediateDirectories: true,
                attributes: nil
            )
        }
    }
}
```

Next step is the implementation of the writable part of our storage. It is a little bit tricky because the key is a path to our data on the file system. That's why we need append the key to our root path and generate new URL for the storing data. New URL can contain subfolders, that's why we create a createFolders function which creates needed folders according to the path.

```swift
extension DiskStorage: ReadableStorage {
    func fetchValue(for key: String) throws -> Data {
        let url = path.appendingPathComponent(key)
        guard let data = fileManager.contents(atPath: url.path) else {
            throw StorageError.notFound
        }
        return data
    }

    func fetchValue(for key: String, handler: @escaping Handler<Data>) {
        queue.async {
            handler(Result { try self.fetchValue(for: key) })
        }
    }
}
```

Here is the readable part of our Storage protocol, where we implement data fetching for a passed key. Again, we use the key as a path to our data on disk. Now we have a working example of straightforward disk storage. Next step is implementing a simple adapter for our DiskStorage class, which will handle JSON coding/decoding.

```swift
class CodableStorage {
    private let storage: DiskStorage
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder

    init(
        storage: DiskStorage,
        decoder: JSONDecoder = .init(),
        encoder: JSONEncoder = .init()
    ) {
        self.storage = storage
        self.decoder = decoder
        self.encoder = encoder
    }

    func fetch<T: Decodable>(for key: String) throws -> T {
        let data = try storage.fetchValue(for: key)
        return try decoder.decode(T.self, from: data)
    }

    func save<T: Encodable>(_ value: T, for key: String) throws {
        let data = try encoder.encode(value)
        try storage.save(value: data, for: key)
    }
}
```

CodableStorage class wraps our DiskStorage class to add JSON coding-decoding logic. It uses generic constraints to understand how to decode and encode data. It's time to use our CodableStorage in real life sample.

```swift
struct Timeline: Codable {
    let tweets: [String]
}

let path = URL(fileURLWithPath: NSTemporaryDirectory())
let disk = DiskStorage(path: path)
let storage = CodableStorage(storage: disk)

let timeline = Timeline(tweets: ["Hello", "World", "!!!"])
try storage.save(timeline, for: "timeline")
let cached: Timeline = try storage.fetch(for: "timeline")
```

In the code sample above, you can see the usage of CodableStorage class. We create DiskCache class instance which uses a temporary folder to store data. Timeline is a simple codable struct representing an array of strings which we store in our CodableStorage.

#### Conclusion
Today we discussed a simple way of storing Codable structs which we can fetch via REST API. Sometimes we don't need complicated features of CoreData for simple JSON caching and it is enough to implement disk storage.

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!
