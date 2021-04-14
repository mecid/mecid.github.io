---
title: Localization in SwiftUI
layout: post
category: Accessibility
---

This week I want to talk about another crucial feature of any app, which is *Localization*. Every user expects that your app correctly uses environment features like the right-to-left layout or uses system locale to format dates or currencies. Another vital thing here is translations, and this week, we will learn which tools SwiftUI provides to add in our apps as many languages as we can.

{% include friends.html %}

#### LocalizedStringKey
*LocalizedStringKey* is a special struct which is provided by the SwiftUI framework. It conforms *ExpressibleByStringLiteral* protocol, which allows us to create this struct by using a *String* value. *Text* component, on the other hand, has an overload that accepts *LocalizedStringKey* instead of *String*. It allows us to use our localization keys in a very transparent way. Let's take a look at a quick example. 

```swift
let goal: LocalizedStringKey = "goal"
let text = Text(goal)
```

*LocalizedStringKey* looks for a *"goal"* key in translation files, and as soon as it finds provided translation for a key *"goal"* it will replace the key with a correct translated string. In the case where there is no provided translation, it will use the key as a dummy string value.

To learn how to create translation files, take a look at the ["Internationalization and Localization"](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html#//apple_ref/doc/uid/10000171i-CH5-SW1) guide provided by Apple.

#### String interpolation
We already know how to localize static text in our apps. But what about the dynamic text, where we need to inject some data like names or some numeric values. Fortunately, *LocalizedStringKey* conforms *ExpressibleByStringInterpolation* protocol, which allows us to use *String* interpolation to pass data into our string values. Let's take a look at a small example.

```swift
let name = "Majid"
Text("myNameIs \(name)")
```

In the example above, we create a *Text* component with the *"myNameIs \(name)"* string value. As you already know, the Text component has an init method overload, which accepts *LocalizedStringKey* and *LocalizedStringKey* conforms *ExpressibleByStringInterpolation*, and **this is all the magic behind the localization in SwiftUI**. This piece of code will look for the *"myNameIs %@"* key in your translation files. Let's take a look at how our translation file should look to make it work.

```swift
"myNameIs %@" = "My name is %@.";
```

**I have to mention that it doesn't work in preview canvas. You should run it on simulator.**

*String* interpolation is a compelling feature of Swift language. By using *String* interpolation, we can also pass stuff like formatters or format specifiers, which can provide additional presentation logic. Let's take a look at a few code examples.

```swift
let date = Date()
let formatter = DateFormatter()
formatter.dateStyle = .medium
formatter.timeStyle = .none
Text("birthday \(date, formatter: formatter)")
```

In the code sample above, we pass a *DateFormatter*, which converts a *Date* representation into a proper localized string value. This code will run a localized string lookup and then apply the formatter. Here is how the translation file should look.

```swift
"birthday %@" = "My birthday is %@.";
```

Let's take a look at another quick sample where we inject integer into our translation.

```swift
let age = 28
Text("age \(age)")

"age %lld" = "I'm %lld years old";
```

#### Conclusion
Localization is an essential aspect in the world of user experience. Today we learned how easily we could localize our apps with the help of *LocalizedStringKey* struct provided by SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 

