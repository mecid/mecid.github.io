---
title: Custom accessibility content in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

SwiftUI Release 3 brought a lot of new accessibility APIs, which we can use to improve user experience drastically in an effortless way. This week I want to talk about another new API that allows us to provide customized accessibility content using the new *accessibilityCustomContent* view modifier in SwiftUI.

Let's start with a simple example that defines the *User* struct and a view presenting an instance of the *User* struct.

```swift
import SwiftUI

struct User: Decodable {
    let name: String
    let email: String
    let address: String
    let age: Int
}

struct UserView: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading) {
            Text(user.name)
                .font(.headline)
            Text(user.address)
                .font(.subheadline)
                .foregroundColor(.secondary)
            Text(user.email)
                .foregroundColor(.secondary)
            Text("Age: \(user.age)")
                .foregroundColor(.secondary)
        }
    }
}
```

SwiftUI provides us with excellent accessibility support out of the box. You don't need to do anything to make your *UserView* accessible. Every piece of text inside the *UserView* is accessible for assistive technologies like VoiceOver and Switch Control. It might sound good, but it can overwhelm VoiceOver users with a lot of data. Let's improve accessibility support a little bit but adding a few accessibility modifiers to our *UserView*.

```swift
struct UserView: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading) {
            Text(user.name)
                .font(.headline)
            Text(user.address)
                .font(.subheadline)
                .foregroundColor(.secondary)
            Text(user.email)
                .foregroundColor(.secondary)
            Text("Age: \(user.age)")
                .foregroundColor(.secondary)
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(user.name)
    }
}
```

As you can see in the example above, we use accessibility modifiers to ignore the accessibility content of the children to make the stack itself an accessibility element. We also add the accessibility label to the stack but still miss the other parts. We want to make all the data accessible. Usually, we use different fonts and colors to prioritize text visually, but how can we achieve the same impact for assistive technologies?

> To learn about the basics of accessibility in SwiftUI, take a look at my ["Accessibility in SwiftUI"](/2019/09/10/accessibility-in-swiftui/) post.

Fortunately, SwiftUI provides a way to generate customized accessibility content with different importance using the brand new *accessibilityCustomContent* view modifier. Let's take a look at how we can use it.

```swift
struct UserView: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading) {
            Text(user.name)
                .font(.headline)
            Text(user.address)
                .font(.subheadline)
                .foregroundColor(.secondary)
            Text(user.email)
                .foregroundColor(.secondary)
            Text("Age: \(user.age)")
                .foregroundColor(.secondary)
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(user.name)
        .accessibilityCustomContent("Age", "\(user.age)")
        .accessibilityCustomContent("Email", user.email, importance: .high)
        .accessibilityCustomContent("Address", user.address, importance: .default)
    }
}
```

Here we add a bunch of *accessibilityCustomContent* view modifiers to define custom accessibility content with various priorities. The *accessibilityCustomContent* view modifier has three parameters:

1. The first one is the localized label for your custom content that VoiceOver uses to announce.
2. The second one is the localized label or string value presenting custom content.
3. The third one is the importance level of your custom content. It can be *default* or *high*. VoiceOver reads content with high importance immediately, while the content with default importance is spoken only in verbose mode when the user uses vertical swipes to access more data.

The *accessibilityCustomContent* view modifier allows us to prioritize data for assistive technologies. For example, VoiceOver reads the data with *high* importance immediately and enables the user to access data with *default* importance as needed using vertical swipes.

You can use as many *accessibilityCustomContent* view modifiers as needed to present a massive subset of your data. Remember that you can also replace and override data or importance by introducing *accessibilityCustomContent* view modifiers with the same label.

An excellent way to keep your custom accessibility content labels consistent across the large codebase is using the *AccessibilityCustomContentKey* type. You can use an instance of it as the first parameter of the *accessibilityCustomContent* view modifier.

```swift
extension AccessibilityCustomContentKey {
    static let age = AccessibilityCustomContentKey("Age")
    static let email = AccessibilityCustomContentKey("Email")
    static let address = AccessibilityCustomContentKey("Address")
}

struct UserView: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading) {
            Text(user.name)
                .font(.headline)
            Text(user.address)
                .font(.subheadline)
                .foregroundColor(.secondary)
            Text(user.email)
                .foregroundColor(.secondary)
            Text("Age: \(user.age)")
                .foregroundColor(.secondary)
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(user.name)
        .accessibilityCustomContent(.age, "\(user.age)")
        .accessibilityCustomContent(.email, user.email, importance: .high)
        .accessibilityCustomContent(.address, user.address, importance: .default)
    }
}
```

In the example above, we define a few shortcuts for our custom accessibility content keys and use them in conjunction with the *accessibilityCustomContent* view modifier.

Today we learned how to use the *accessibilityCustomContent* view modifier to make our apps more accessible by prioritizing our data for assistive technologies and allowing user access as more details as needed. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
