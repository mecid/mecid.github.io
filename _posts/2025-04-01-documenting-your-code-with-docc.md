---
title: Documenting your code with DocC
layout: post
---

Today I would like to talk about documenting Swift code using the DocC. Documenting your code becomes even more important in the era of modularized apps. Whenever different parts of your app live in multiple Swift Packages, it becomes crucial to provide proper documentation.

I’m not here to persuade you to document your code, but I’d like to share a quote from a software engineering book that significantly influenced my approach as a software engineer. 

> Many developers put off writing documentation until the end of the development process, after coding and unit testing are complete. This is one of the surest ways to produce poor quality documentation. The best time to write comments is at the beginning of the process, as you write the code. Writing the comments first makes documentation part of the design process. Not only does this produce better documentation, but it also produces better designs

Nowadays, Apple provides us with the documentation compiler called DocC. DocC converts Markdown-based text into rich documentation for Swift frameworks and packages. Today, we will learn the basics of DocC, allowing us to provide proper documentation for our code.

DocC syntax is pretty simple but yet powerful. You don’t need to create additional files or use additional software; DocC is a part of the Swift compiler. The The DocC compiler infers the documentation from your source code. Instead of putting double slashes for comments, use triple slashes for documentation. That’s it.

```swift
extension Array {
    /// Returns a random element based on the day of the month,
    /// which is calculated to determine the random index.
    func stableRandom(using date: Date = .now) throws -> Element {
        let day = Calendar.current.component(.day, from: date)
        
        if day < count {
            return self[day]
        } else {
            let index = day % count
            return self[index]
        }
    }
}
```

As you can see in the example above, we use triple slashes to add some documentation to the function. Now, you can use Option+Click in your Xcode editor to see the documentation of the function. That’s so easy.

Whenever you need to add more details on the implementation of the function, you can add more paragraphs, and they will be displayed in the discussions section of the help dialog. Let’s move forward by adding more information about its parameters and return type.

```swift
extension Array {
    /// Returns a random element based on the day of the month,
    ///  which is calculated to determine the random index.
    ///
    /// The stableRandom function consistently returns the same element throughout
    /// the entire date. However, its value changes as soon as the date changes.
    func stableRandom(using date: Date = .now) throws -> Element {
        let day = Calendar.current.component(.day, from: date)
        
        if day < count {
            return self[day]
        } else {
            let index = day % count
            return self[index]
        }
    }
}
```

You can add info about parameters using - Parameters: keyword. Both dash and colon are required and used as separation indicators. On a new line, you can start writing the name of the parameter after another dash with indentation; the colon after the name is required to separate the name of its description.

```swift
extension Array {
    /// Returns a random element based on the day of the month,
    ///  which is calculated to determine the random index.
    ///
    /// The stableRandom function consistently returns the same element throughout
    /// the entire date. However, its value changes as soon as the date changes.
    ///
    /// - Parameters:
    ///     - date: The date used in the randomization algorithm.
    func stableRandom(using date: Date = .now) throws -> Element {
        let day = Calendar.current.component(.day, from: date)
        
        if day < count {
            return self[day]
        } else {
            let index = day % count
            return self[index]
        }
    }
}
```

Returns and Throws keywords enable us to specify the description for returning values and throwing errors. The syntax for these keywords is the same: they begin with a dash, end with a colon, and then contain the description.

```swift
extension Array {
    /// Returns a random element based on the day of the month,
    ///  which is calculated to determine the random index.
    ///
    /// The stableRandom function consistently returns the same element throughout
    /// the entire date. However, its value changes as soon as the date changes.
    ///
    /// - Parameters:
    ///     - date: The date used in the randomization algorithm.
    ///
    /// - Returns: A random element of the array.
    ///
    /// - Throws: May throw an error whenever can't randomize the date
    func stableRandom(using date: Date = .now) throws -> Element {
        let day = Calendar.current.component(.day, from: date)
        
        if day < count {
            return self[day]
        } else {
            let index = day % count
            return self[index]
        }
    }
}
```

Whenever you need to refer to another type in the documentation, use double backticks. It will allow you to link to the particular documentation entity.

```swift
extension Array {
    /// Returns a random element based on the day of the month,
    ///  which is calculated to determine the random index.
    ///
    /// The stableRandom function consistently returns the same element throughout
    /// the entire date. However, its value changes as soon as the date changes.
    ///
    /// - Parameters:
    ///     - date: The date used in the randomization algorithm.
    ///
    /// - Returns: A random element of the array.
    ///
    /// - Throws: May throw the ``RandomError.random`` error
    func stableRandom(using date: Date = .now) throws -> Element {
        let day = Calendar.current.component(.day, from: date)
        
        if day < count {
            return self[day]
        } else {
            let index = day % count
            return self[index]
        }
    }
}
```
