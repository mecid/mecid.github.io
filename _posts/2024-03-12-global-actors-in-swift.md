---
title: Global actors in Swift
layout: post
image: /public/swift.png
category: Swift Language Features
---

The Swift language allows us to define thread-safe types using actors. Actor type automatically manages exclusive access to the data it protects. But what if we need multiple types protected with a mutually exclusive access? That's why we have global actors, and today, we will learn how to use global actors in Swift.

{% include friends.html %}

The main thread rendering is the best example of why we need to protect multiple types with a mutually exclusive access. You may have a massive collection of *UIViewControllers*, *UIViews*, or SwiftUI views running in paralel, but in the end, you should update your user interface on the main thread. 

> If you are unfamiliar with the actor concept, look at my dedicated ["Thread safety in Swift with actors"](/2023/09/19/thread-safety-in-swift-with-actors/) post.

That's why Swift provides us *@MainActor*. Any *UIViewController* or *UIView* you create inherits *@MainActor* access from its definition. SwiftUI's *View* protocol also defines its *body* property with *@MainActor*. This means your view's body, view, or controller always runs on the main thread and protects you from accidentally updating the user interface from the background thread.

To fully understand the idea of the global actors, let's inspect the *@MainActor* type a bit further.

```swift
@globalActor actor MainActor : GlobalActor {
    static let shared: MainActor
}
```

As you can see in the code example above, the *MainActor* type is defined with **actor** keyword and conforms to the *GlobalActor* protocol. It also has the **@globalActor** attribute. The *GlobalActor* protocol requires you to specify the *shared* property to create a shared, also called a global instance of the actor. 

```swift
@Observable @MainActor final class Store {
    // ...
}
```

Now, we can easily mark any type we need with the *@MainActor* attribute to isolate it to the main actor. This means all the work in the particular type runs exclusively on the main actor.	

Let's move forward and build our own global actor. Assume that you have a set of types accessing the local storage and you want to keep files conflict-free on the disk by running exclusively.

```swift
@globalActor actor StorageActor: GlobalActor {
    static let shared = StorageActor()
}
```

As you can see in the example above, we define the *StorageActor* type conforming to the *GlobalActor* protocol using the **actor** keyword. The *@globalActor* attribute allows us to mark any type, function, or property with the *@StorageActor*.

```swift
@StorageActor final class Cache {
    let folder: URL
    
    init(folder: URL) {
        self.folder = folder
    }
    
    func get(_ key: String) -> Data? {
        // ...
    }
    
    func set(data: Data, for key: String) {
        // ...
    }
}

@StorageActor final class Database<Value> {
    let folder: URL
    
    init(folder: URL) {
        self.folder = folder
    }
    
    func search(matching query: String) -> [Value] {
        // ...
    }
}
```

Here, we create *Сache* and *Database* types using the *@StorageActor* attribute. It allows us to run them on a shared, mutually exclusive actor, managed by the *StorageActor* we created before. 

Why do we use global actors rather than defining *Cache* and *Database* types as actors? We can define *Cache* and *Database* as actors. Still, in this case, every instance of the *Cache* or *Database* types will run on an independent actor and protect its access alone. By marking our types with the *@StorageActor*, we belong them to a single, mutually exclusive, shared instance of the *StorageActor*.

```swift
@Observable final class Store {
    private(set) var data: Data?
    
    @StorageActor func load() async {
        let path: String = "some path"
        let content = FileManager.default.contents(atPath: path)
        
        await MainActor.run {
            self.data = content
        }
    }
}
```

Remember that you can mark with the *@StorageActor* attribute not only types but also functions or properties of any type.

Today, we learned why and how to use global actors in Swift. You don't need to use global actors often in your apps. However, they become handy in particular cases, such as main thread rendering. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
