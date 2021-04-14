---
layout: post
title: Hiding third-party dependencies with protocols and extensions
category: Protocol-Oriented Programming
---

There are plenty of discussions on the Internet about using third-party dependencies in your apps. The first part of developers suggest ignoring the usage of libraries and write all the code yourself. The second part recommends using third-party dependencies to speed up app development.

{% include friends.html %}

Sometimes it is better to use a well-tested library, rather than implementing it yourself. A good example here is Cryptography. Cryptography is hard, and it is effortless to make a mistake during the implementation of some common cryptography algorithms. It is an excellent example of a situation where we should use a third-party library.

But here we can face another problem when the author of the library abandoned it or didn't update for the next Swift version. In this case, we have to replace this library with another one or implement our solution. It can be tough to remove the library if you use it across the codebase.

So this week we will talk about two techniques which help us to hide our third-party dependencies and make them easy to replace and refactor.

#### Extensions
Most of our apps fetch and display some data via API, very often we have some image URLs which should be downloaded and cached on the disk/memory. There are a lot of great libraries for downloading and caching images from the internet. Most famous is Kingfisher. 

It has an extension for UIImageView class which brings setImage method. We can easily use this method anywhere in our codebase by importing Kingfisher framework and calling setImage method on UIImageView. 

Assume that we need to replace KingFisher with another library like AlamofireImage. In this case, we have to go through the codebase, replace all Kingfisher imports and setImage method calls to AlamofireImage import and af_setImage method calls respectively. It is going to be tremendous work in case of a huge codebase. Let's check how we can use extensions to fix this problem.

```swift
import UIKit
import Kingfisher

extension UIImageView {
    func setImage(from url: URL) {
        kf.setImage(with: url)
    }
}
```

As you can see in the code sample above, we create an UIImageView extension with setImage method, which calls KingFisher framework's setImage method. Using this extension across the project give us an opportunity to replace KingFisher framework with any other library with a single file change. The only thing we need to change is the implementation of our setImage method.

#### Protocols
Another excellent example of a third-party dependency can be a Keychain access library. Keychain is the safest place on iOS device to keep user sensitive data like access tokens and passwords. Apple provides us with such ugly string based API for Keychain access, that's why I decide to use third-party wrapper around Apple's API. Here is an example of KeychainSwift library usage.

```swift
let keychain = KeychainSwift()
keychain.set("hello world", forKey: "my key")
keychain.get("my key") // Returns "hello world"
```

KeychainSwift library provides nice key-value API for Keychain access. But I don't want to expose library usage across the codebase. Let's create a protocol which defines user sensitive datastore and hides library usage.

```swift
protocol TokenStore {
    var accessToken: String { get set }
    var refreshToken: String { get set }
}
```

Next step is adding TokenStore protocol conformance to KeychainSwift library.
```swift
extension KeychainSwift: TokenStore {
    private enum Keys {
        static let accessToken = "accessToken"
        static let refreshToken = "refreshToken"
    }

    var accessToken: String {
        get { return get(Keys.accessToken) }
        set { set(newValue, forKey: Keys.accessToken) }
    }

    var refreshToken: String {
        get { return get(Keys.refreshToken) }
        set { set(newValue, forKey: Keys.refreshToken)}
    }
}
```

And now we can pass TokenStore protocol across our codebase instead of exposing usage of KeychainSwift library.

```swift
class AuthenticationService {
    private let tokenStore: TokenStore

    init(tokenStore: TokenStore) {
        self.tokenStore = tokenStore
    }

    func fetchToken(for credentials: Credentials) {
//        Save tokens here
//        tokenStore.accessToken =
//        tokenStore.refreshToken =
    }
}
```

The single place which knows about KeychainSwift library should be a dependency container which creates AuthenticationService object. More about Dependency Injection we will talk in the next posts.

#### Conclusion
Today we discussed how we could use protocols and extensions to build tiny abstractions which hide third-party dependencies and make our codebase safer. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!