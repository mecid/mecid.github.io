---
title: Functional core, Imperative shell in Swift
layout: post
---

We love value types because they provide us with safety and predictability, allowing us to reason about the code we write. But we still need objects to hold and mutate our app's shared state. This week, we will discuss modeling our app's logic by leveraging the value and reference semantics.

Value types are great, and one of the reasons for that is instance isolation. Whenever you pass a value type as a parameter or assign it to another variable, you get a new copy with the very same data. You always have a warranty that there are no side effects across the app while mutating the local copy of the value type.

====================================================

Even mutating functions in your value types create a new copy and reassign it to the current variable. That's why you can call mutating functions only on instances defined with the var keyword. It means you can easily replace one sample of value type with another, and it will work the same way. This feature of value types shines when you write unit tests. Pure functions like these are easy to test because you only need to verify the function's output. There are no side effects.

Isolated value types work great for defining your app's state and pure actions around that piece of data. But we still need to share mutable states between different app screens and make side effects like networking and IO operations. Reference types are great for data sharing. Whenever you assign an instance of a class to a new variable, it shares the reference to the same object.

We can use objects to store and share the state represented by a value type. Objects are not pure. That's why it is an excellent place for side effects. The idea is to encapsulate all the app logic using value types with pure functions and only use objects to store value types and provide side effects.

====================================================

As you can see in the example above, we have an object holding the state represented by a value type and providing very few functions to make side effects possible. We encapsulated all the logic in the value type that we can easily verify in unit tests. And we can still quickly write integration tests for the object by mocking only side effects to check its behavior.

====================================================

The idea of functional core and imperative shell fits very well into the world of mobile development with Swift language, which provides so many language features allowing us to write expressible and safe code in a very few keystrokes. In the next week, we will talk about this approach in the context of unidirectional flow. 
