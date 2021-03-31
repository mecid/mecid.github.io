---
title: Writing idiomatic Swift code
layout: post
category: Swift Language Features
image:
---

Today is a great day to start learning iOS development. There are plenty of new tools and frameworks to learn from zero and apply every day. iOS development evolves every year and brings us new things to learn. This post should be valuable for the people who move to Swift from another programming language. This week we will talk about Swift idioms and how to write idiomatic Swift code.

Swift language encourages you to write safe, fast, and expressive code. It provides you all the needed programming language features to allow you to write safe and bug-free code.

#### Reference and Value types
Swift language provides you reference and value types. Reference types are classes and closures. Classes are fundamental building blocks in Object-Oriented Programming. Value types are structs and enums. The main difference between them is the lifecycle. Swift copies value for value types when you pass it around. On the other hand, Swift copies reference for reference types when you pass them.

What does it mean for us? Classes allow us to share mutable states by sharing a reference to the data. For example, if you need to have a piece of data shared between different components and every component should have an opportunity to mutate the shared state, class is a way to go. For all other cases, value types are what you need.

=====================================================

Here we have a class called SharedState, and we pass it inside a function and mutate it, then we print it outside the function. We model SharedState as a class because we need a shared state that we can mutate. We can't do the same with structs because value types encourage immutability.

#### Enums
Enum is a great way to model an exclusive piece of state. I love enums, and it is one of my favorite Swift language features that allow us to build a very type-safe code. Let's take a look at the example that defines possible sorting options in my app.

=====================================================

Enum is a compelling Swift language feature because it can have different associated values in different cases.

=====================================================

Now we can use the switch with case let to extract the associated values for every case. But remember that we should use enums for mutually exclusive cases. You can always use structs with static fields for inclusive cases.

=====================================================

#### Optionals
Another language feature that Swift provides us to write type-safe code is Optional. Swift types can't be nil unless you define them as optional. In this case, the Swift compiler will require to handle all the usages where the value can be nil.

=====================================================

We can use ?? operator to provide a default value whenever value is not available, or we can use if let expression to extract the optional value and use it inside the inner scope.

#### Conclusion
Swift programming language evolves very fast. The core team is constantly considering the feedback that developers provide and add new features to the Swift language. To write more idiomatic code, you should also learn about pattern matching, protocols, generics, and functional programming in Swift.