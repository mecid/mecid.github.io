---
title: TextField in SwiftUI
layout: post
image: /public/textfield.png
---
This week I want to talk to you about a *TextField* component in *SwiftUI*.  It might look like an elementary tutorial, but *TextField* has pretty exciting features like out of the box formatting that we don't have in *UIKit*. Let's start with the basics of the *TextField* component.

#### Basics
As you might know, we can use *TextField* for user input. All we need to do is creating a *TextField* and assigning it to any binding of a *String* value. Let's take a look at a very quick example.

```swift
struct ContentView: View {
    @State private var text = ""

    var body: some View {
        TextField("type something...", text: $text)
    }
}
```

In the example above, we create a *TextField* with a placeholder and bind it to a state variable. Pretty easy, right? *TextField* also provides callbacks that we might need to handle during user input.

```swift
struct ContentView: View {
    @State private var text = ""

    var body: some View {
        TextField(
            "type something...",
            text: $text,
            onEditingChanged: { _ in print("changed") },
            onCommit: { print("commit") }
        )
    }
}
```

As you can see in the example above, *TextField* allows us to handle *onEditingChanged* and *onCommit* events. Let's learn what the difference between them. *TextField* calls *onEditingChanged* closure whenever the user starts or finishes editing text. It also passes a boolean value that describes a starting or finishing event. Whenever a user performs an action like pressing the return key, *TextField* calls *onCommit* closure.

#### TextField formatters
OK, we learned basics, and now we can move to a more interesting feature of *TextFields*. You might be noticed that *TextField* has a few overloads that accept *Formatter*. These overloads allow us to bind a *TextField* to raw data and present it after converting to a string by using selected *Formatter* instance. Let's take a look at a quick example that will help us to understand how it works.

```swift
extension NumberFormatter {
    static var currency: NumberFormatter {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        return formatter
    }
}

struct ContentView: View {
    @State private var price = 99

    var body: some View {
        TextField(
            "type something...",
            value: $price,
            formatter: NumberFormatter.currency
        )
    }
}
```

*TextField* uses provided *Formatter* while converting between the string that user edits and the underlying raw value. In case when *Formatter* is unable to perform a conversion, the value will not be modified. Try to type some letters that *Formatter* is not able to convert.

#### Styling
*SwiftUI* provides us a few styles and the *textFieldStyle* modifier that we can use to apply styles to our *TextFields* in the app. *textFieldStyle* modifier uses the environment to pass the style to every view inside the environment. It looks very similar to the *buttonStyle* modifier that discussed in the previous post.

```swift
struct ContentView: View {
    @State private var text = ""
    @State private var price = 99

    var body: some View {
        TextField("type something...", text: $text)
            .textFieldStyle(RoundedBorderTextFieldStyle())
    }
}
```

*SwiftUI* has a *TextFieldStyle* protocol that we can use to provide styling to our *TextFields*. Let's take a look at how we can use it.

```swift
struct SuperCustomTextFieldStyle: TextFieldStyle {
    func _body(configuration: TextField<_Label>) -> some View {
        configuration
            .padding()
            .border(Color.accentColor)
    }
}

struct ContentView: View {
    @State private var text = ""

    var body: some View {
        TextField("type something...", text: $text)
            .textFieldStyle(SuperCustomTextFieldStyle())
    }
}
```

It looks like there is some compiler magic behind this protocol because it requires *_body*, which is not a part of *TextFieldStyle*. 

#### Conclusion
This week we learned the things behind the *TextField* component. It provides us an interesting raw data formatting feature that we don't have in *UIKit*. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!