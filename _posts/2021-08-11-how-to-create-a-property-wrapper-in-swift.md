---
title: How to create a property wrapper in Swift
layout: post
---

Property wrapper is a Swift language feature. The main goal here is wrapping properties with logic that we extract into a separate type to reuse it across the codebase. This week, we will learn how to create a property wrapper to read Keychain data and be a good citizen in the SwiftUI world by reacting to data changes.

#### Basics
SwiftUI provides us SceneStorage and AppStorage property wrappers to access data in scene memory and user defaults, respectively. Unfortunately, it doesn't give us a similar property wrapper to access Keychain, but we can build it. Let me show you a code example that defines the usage of the property wrapper we try to make.

=====================================================

As you can see in the example above, we use the SecureStorage property wrapper that needs a key to read data from Keychain. We also provide a default value that we can override over time. And now, we can move to the implementation details of the SecureStorage property wrapper.

To create a property wrapper, we have to create a new type annotated with the @propertyWrapper attribute. A property wrapper type must contain a field named wrappedValue. Wrapped value is the heart of any property wrapper. Swift will transparently access the value of wrapped value whenever you read or write to any property that uses a property wrapper.

=====================================================

We declare the SecureStorage property wrapper with generic constraint to decode/encode the value while writing and reading from the Keychain service. We also define an initializer that takes a key and an initial value. Swift automatically calls this initializer when you define a field with a property wrapper with a default value.

Now, we can implement the calculatable property called wrappedValue to provide access to the Keychain service.

=====================================================

Here we hide all the implementation details of using Keychain service inside the property wrapper type. We also use the third-party KeychainAccess package to utilize its nice API. 

Another great thing about property wrappers is that you can use them to hide third-party code. You can easily update the SecureStorage type whenever you want to change the implementation or migrate to another Swift Package for Keychain. And this is going to be the only place for changes because the whole codebase uses the SecureStorage property wrapper and doesn't know about implementation details.

=====================================================

#### SwiftUI support
We used to obtain bindings from property wrappers that SwiftUI provides us by using the $ sign. We can achieve the same behavior in our custom property wrappers by implementing the projectedValue property on the property wrapper type.

=====================================================

Here we create a calculated property that returns the binding to the value controlled by the property wrapper type. Now we can easily provide that binding to any SwiftUI view.

Another thing that we expect from SwiftUI is that it updates the view as soon as data hidden by a property wrapper changes. To achieve that, we should do two things:
Conform our property wrapper type to the DynamicProperty protocol that SwiftUI gives us. DynamicProperty protocol has only one requirement that has a default implementation.
Use some of SwiftUI provided property wrappers like @State, @StateObject, etc., inside our custom property wrapper type to react to data changes.

=====================================================

In the current implementation, I've created a private class that holds all the logic of data read/write and conforms to ObservableObject protocol to use with the StateObject property wrapper. Finally, SwiftUI will update the corresponded views as soon as the data under the SecureStorage property wrapper changes.

#### Conclusion
Property wrapper is a great way to extract reusable logic into a separate type and use it across the codebase. It also has an advantage while hiding the third-party dependencies by providing a nice API to the client-side code.
