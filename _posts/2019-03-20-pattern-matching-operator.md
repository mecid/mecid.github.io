---
title: Deep dive into Pattern matching with ~= operator 
layout: post
category: Swift Language Features
---

[In one of my previous posts, we talked about the Pattern Matching feature of Swift language](/2019/02/06/pattern-matching-with-case-let). We discussed how we could use "case let" keyword in our daily development to find patterns in Enums, Turples, and Optionals. But today we are going to talk about particular Pattern Matching operator which hides all of this magic behind it.

{% include friends.html %}

Pattern Matching is the act of checking a given sequence of tokens for the presence of the constituents of some pattern. Let's take a look at a simple string matching operation.

```swift
let message = "Hello World!"

switch message {
case "Hello": print("hello")
case "World": print("world")
case "Hello World!": print("Hello World!")
default: break
}
```

As you can understand this code will print "Hello World!" message in the console. In most of the cases Pattern Matching work as equality check, except Ranges, where it refers to the "contains" method of Range type.

So, the question is "How it is really working?". Behind the Pattern Matching operation, Swift uses ~= operator, which is overloaded for most of the standard types. While using Pattern Matching, Swift is looking for ~= operator for the current types. Here is an example of how ~= operator looks for String type.

```swift
func ~= (pattern: String, value: String) -> Bool {
    return pattern == value
}
```

The good news here is that we can easily overload ~= operator to change this behavior. For example, in the code listing below we change the implementation to custom one, where we instead of equality checking match for containment and now you will see the "Hello" message in the console.

```swift
func ~= (pattern: String, value: String) -> Bool {
    return value.contains(pattern)
}

let message = "Hello World!"

switch message {
case "Hello": print("hello")
case "World": print("world")
case "Hello World!": print("Hello World!")
default: break
}
```

There is no magic behind the Pattern Matching. It is just a simple ~= function. We can easily define it for our types and use them in switch statements. Let's do that.

```swift
struct User {
    let firstName: String
    let secondName: String
    let age: Int
}

extension User {
    static func ~= (range: ClosedRange<Int>, user: User) -> Bool {
        return range.contains(user.age)
    }
}

let user = User(firstName: "Majid", secondName: "Jabrayilov", age: 27)

switch user {
case 21...30: print("The user age is between 21 and 30")
case 31...40: print("The user age is between 31 and 40")
default: break
}
```

Here we have straightforward User struct which contains the name, second name and age fields. I want to be able to use User struct instances in switch statements for matching the age range of my users. Please take a look at the order of parameters in ~= function. The first one describes the case value, where the second one is the value used after the switch keyword. Console output, in this case, is "The user age is between 20 and 30".

Another good option for overloading Pattern Matching operator can be regular expressions. I want to match the text to multiple regular expression patterns. Let's dive into the code.

```swift
struct Regex {
    let pattern: String

    static let email = Regex(pattern: "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}")
    static let phone = Regex(pattern: "([+]?1+[-]?)?+([(]?+([0-9]{3})?+[)]?)?+[-]?+[0-9]{3}+[-]?+[0-9]{4}")
}

extension Regex {
    static func ~=(regex: Regex, text: String) -> Bool {
        return text.range(of: regex.pattern, options: .regularExpression) != nil
    }
}
```

Here we have Regex struct which has only one field, and that is the pattern string. We also implement email and phone static constants with predefined regular expressions. Next, we overload ~= operator, in this case, it matches text to our Regex struct by using "range of" method of string type. That's all we need to use our Regex type for Pattern Matching. Here is the usage example.

```swift
let email = "cmecid@gmail.com"

switch email {
case Regex.email: print("email")
case Regex.phone: print("phone")
default: print("default")
}
```

Today we discussed how actually is working Pattern Matching in Swift, how easily we can override the logic for standard types and how we can add the ability for Pattern Matching to custom types. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!

