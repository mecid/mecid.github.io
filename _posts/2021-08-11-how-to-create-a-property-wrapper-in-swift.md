---
title: How to create a property wrapper in Swift
layout: post
category: Swift Language Features
image: /public/pw.png
---

Property wrapper is a Swift language feature. The main goal here is wrapping properties with a logic that we extract into a separate type to reuse it across the codebase. This week, we will learn how to create a property wrapper to read data in Keychain and be a good citizen in the SwiftUI world by reacting to data changes.

{% include friends.html %}

#### Basics
SwiftUI provides us *SceneStorage* and *AppStorage* property wrappers to access data in scene memory and user defaults, respectively. Unfortunately, it doesn't give us a similar property wrapper to access Keychain, but we can build it. Let me show you a code example that defines the usage of the property wrapper we try to make.

```swift
final class SettingsStore: ObservableObject {
    @SecureStorage(key: Constants.Keychain.useSleepMode)
    var useSleepMode = true
}
```

As you can see in the example above, we use the *SecureStorage* property wrapper that needs a key to read data from Keychain. We also provide a default value that we can override over time. And now, we can move to the implementation details of the *SecureStorage* property wrapper.

> To learn more about property wrappers in SwiftUI, take a look at my dedicated ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/) post.

To create a property wrapper, we have to create a new type annotated with the **@propertyWrapper** attribute. A property wrapper type must contain a field named **wrappedValue**. Wrapped value is the heart of any property wrapper. Swift will transparently access the wrapped value whenever you read or write to any property that uses a property wrapper.

```swift
import KeychainAccess
import Foundation

@propertyWrapper struct SecureStorage<Value: Codable> {
    private let key: String
    private let initialValue: Value

    private let decoder = JSONDecoder()
    private let encoder = JSONEncoder()

    private let keychain = Keychain(
        service: Constants.sharedGroup,
        accessGroup: Constants.sharedGroup
    )
        .synchronizable(true)
        .accessibility(.always)

    init(wrappedValue initialValue: Value, key: String) {
        self.key = key
        self.initialValue = initialValue
    }
}
```

We declare the *SecureStorage* property wrapper with generic constraint to decode/encode the value while writing and reading from the Keychain service. We also define an initializer that takes a key and an initial value. Swift automatically calls this initializer when you define a field with a property wrapper and a default value.

Now, we can implement the calculatable property called *wrappedValue* to provide access to the Keychain service.

```swift
@propertyWrapper struct SecureStorage<Value: Codable> {
//  ....
    var wrappedValue: Value {
        get {
            guard
                let data = try? keychain.getData(key),
                let value = try? decoder.decode(Value.self, from: data)
            else {
                return initialValue
            }

            return value
        }

        set {
            guard let data = try? encoder.encode(newValue) else {
                return
            }

            try? keychain.set(data, key: key)
        }
    }
}
```

Here we hide all the implementation details of using Keychain service inside the property wrapper type. We also use the third-party [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess) package to utilize its nice API. 

Another great thing about property wrappers is that you can use them to hide third-party code. You can easily update the SecureStorage type whenever you want to change the implementation or migrate to another Swift Package for Keychain. And this is going to be the only place for changes because the whole codebase uses the *SecureStorage* property wrapper and doesn't know about implementation details.

#### SwiftUI support
We used to obtain bindings from property wrappers that SwiftUI provides us by using the **$** sign. We can achieve the same behavior in our custom property wrappers by implementing the **projectedValue** property on the property wrapper type.

```swift
@propertyWrapper struct SecureStorage<Value: Codable> {
//  ....
    var projectedValue: Binding<Value> {
        .init(
            get: { wrappedValue },
            set: { wrappedValue = $0 }
        )
    }
}
```

> To learn more about bindings in SwiftUI, take a look at my dedicated ["Binding in SwiftUI"](/2020/04/08/binding-in-swiftui/) post.

Here we create a calculated property that returns the binding to the value controlled by the property wrapper type. Now we can easily provide that binding to any SwiftUI view. Keep in mind that you can make *projectedValue* of any type you need, not only *Binding*.

Another thing that we expect from SwiftUI is that it updates the view as soon as data hidden by a property wrapper changes. To achieve that, we should do two things:
1. Conform our property wrapper type to the *DynamicProperty* protocol that SwiftUI gives us. *DynamicProperty* protocol has only one requirement with a default implementation.
2. Use some of SwiftUI provided property wrappers like *@State, @StateObject, etc.*, inside our custom property wrapper type to react to data changes.

```swift
private final class KeychainStorage<Value: Codable>: ObservableObject {
    var value: Value {
        set {
            objectWillChange.send()
            save(newValue)
        }
        get { fetch() }
    }

    let objectWillChange = PassthroughSubject<Void, Never>()

    private let key: String
    private let defaultValue: Value
    private let decoder = JSONDecoder()
    private let encoder = JSONEncoder()

    private let keychain = Keychain(
        service: Settings.appGroup,
        accessGroup: Settings.appGroup
    )
        .synchronizable(true)
        .accessibility(.always)

    init(defaultValue: Value, for key: String) {
        self.defaultValue = defaultValue
        self.key = key
    }

    private func save(_ newValue: Value) {
        guard let data = try? encoder.encode(newValue) else {
            return
        }

        try? keychain.set(data, key: key)
    }

    private func fetch() -> Value {
        guard
            let data = try? keychain.getData(key),
            let freshValue = try? decoder.decode(Value.self, from: data)
        else {
            return defaultValue 
        }

        return freshValue
    }
}
```

In the current implementation, I've created a private class that holds all the logic of data read/write and conforms to *ObservableObject* protocol to use with the *StateObject* property wrapper. Finally, SwiftUI will update the corresponded views as soon as the data under the *SecureStorage* property wrapper changes.

```swift
@propertyWrapper struct SecureStorage<Value: Codable>: DynamicProperty {
    @StateObject private var storage: KeychainStorage<Value>

    var wrappedValue: Value {
        get { storage.value }

        nonmutating set {
            storage.value = newValue
        }
    }

    init(wrappedValue: Value, _ key: String) {
        self._storage = StateObject(
            wrappedValue: KeychainStorage(
                defaultValue: wrappedValue,
                for: key
            )
        )
    }

    var projectedValue: Binding<Value> {
        .init(
            get: { wrappedValue },
            set: { wrappedValue = $0 }
        )
    }
}

struct HeartPointsSettings: View {
    @SecureStorage(Settings.heartPoinstWeeklyGoal)
    var goal: Int = 150

    var body: some View {
        Section(header: Text("heartMinutesGoal")) {
            Stepper(value: $goal, in: 150...900, step: 10) {
                Text("\(goal) heartMinutesGoalValue")
            }
        }
    }
}
```

#### Conclusion
Property wrapper is a great way to extract reusable logic into a separate type and use it across the codebase. It also has an advantage while hiding the third-party dependencies by providing a nice API to the client-side code. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
