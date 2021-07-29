---
title: Dynamic Type in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

This week I want to talk to you about Dynamic Type support in SwiftUI. I think there is no way to create an excellent user experience without Dynamic Type support in your apps. SwiftUI provides Dynamic Type out of the box for any text representation and simplifies our job. But we still need to do some work, so let's talk about it.

{% include friends.html %}

#### Dynamic Type basics
The Dynamic Type feature allows users to choose the size of textual content displayed on the screen. It helps users who need larger text for better readability. It also accommodates those who can read a smaller text, allowing more information to appear on the screen. Apps that support *Dynamic Type also* provide a more consistent reading experience.

You don't need to do anything to support Dynamic Type in your SwiftUI views, because by default, all the components representing text are multiline. Apple's [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/ios/overview/themes/) have a special section about *Typography*, which provides common text styles. These text styles describe font configuration for different types of text content like *title, headline, body, subhead, caption, footnote*. The styles are shared between all the apps. Try to use these predefined text styles as much as possible. Here is a small example of how to use *HIG* defined text styles in SwiftUI.

```swift
struct PostView: View {
    let post: Post

    var body: some View {
        VStack(alignment: .leading) {
            Image(post.image)

            Text(post.title)
                .font(.headline)

            Text(post.time)
                .font(.subheadline)

            Text(post.body)
                .font(.body)
        }
    }
}
```

To learn how to adapt custom fonts to Dynamic Type take a look at Paul Hudson's ["How to use Dynamic Type with a custom font"](https://www.hackingwithswift.com/quick-start/swiftui/how-to-use-dynamic-type-with-a-custom-font) post.

#### Content size category
In the previous paragraph, I said that SwiftUI supports Dynamic Type out of the box, and that's true. But to support Dynamic Type, we need to keep in mind that every text can be multiline even when it has just two words. It all depends on user-defined font size, which can be extra-extra-large. SwiftUI provides a special environment value describing the user-defined size category. Let's take a look at how we can use it.

```swift
import SwiftUI

struct ContentView: View {
    @Environment(\.sizeCategory) var sizeCategory

    var buttons: some View {
        Group {
            Button("Action 1") {}
            Button("Action 2") {}
        }
    }

    var body: some View {
        Group {
            if sizeCategory.isAccessibilityCategory {
                VStack { buttons }
            } else {
                HStack { buttons }
            }
        }
    }
}
```

By using *sizeCategory* value of the environment, we can read the defined font size and decide how to render our content. By using the environment, our app will subscribe to the system settings, and as soon as the user changes the font size, our view will reload.

> To learn more about environment feature, take a look at ["The power of Environment in SwiftUI"](/2019/08/21/the-power-of-environment-in-swiftui/) post.

Let's go ahead and create an extension for *Group* component, which embeds it into a horizontal or vertical stack depending on the user-defined size category.

```swift
import SwiftUI

fileprivate struct EmbedInStack: ViewModifier {
    @Environment(\.sizeCategory) var sizeCategory

    func body(content: Content) -> some View {
        Group {
            if sizeCategory > ContentSizeCategory.medium {
                VStack { content }
            } else {
                HStack { content }
            }
        }
    }
}

extension Group where Content: View {
    func embedInStack() -> some View {
        modifier(EmbedInStack())
    }
}
```

In the example above, we use *ViewModifier*, which wraps the *Group* of views into a stack. One of the benefits of *ViewModifiers* is the ability to have a state or subscribe to an environment value.

> To learn more about *ViewModifiers*, take a look at ["ViewModifiers in SwiftUI"](/2019/08/07/viewmodifiers-in-swiftui/) post.

#### ScrollViews
The possibility of a short text to be multiline brings another requirement. We need to embed our root views into the scroll view to allow the user to scroll the content when it doesn't fit the screen. It quickly turns into boilerplate, that's why I've created a special extension to reuse this functionality.

```swift
import SwiftUI

extension View {
    func embedInScrollView(alignment: Alignment = .center) -> some View {
        GeometryReader { geometry in
            ScrollView {
                self.frame(
                    minWidth: geometry.size.width,
                    minHeight: geometry.size.height,
                    maxHeight: .infinity,
                    alignment: alignment
                )
            }
        }
    }
}
```

#### ScaledMetric
During WWDC20 Apple released a new property wrapper called *ScaledMetric*. *ScaledMetric* allows you to scale a *BinaryFloatingValue* according to the size category chosen by the user. Let's take a look at the quick example.

```swift
struct ContentView: View {
    @ScaledMetric(relativeTo: .body) var spacing: CGFloat = 8

    var body: some View {
        VStack(spacing: spacing) {
            ForEach(0...10, id: \.self) { number in
                Text(String(number))
            }
        }
    }
}
```

Here we have the vertical spacing value which is scaled using the selected size category.

> To learn more about new property wrappers in SwiftUI, take a look at ["New property wrappers in SwiftUI"](/2020/06/29/new-property-wrappers-in-swiftui/) post.

#### Conclusion
Dynamic Type is a super important feature, and every app should support it. SwiftUI does much stuff out of the box to support Dynamic Type, but it requires some boilerplate. Today we learned how to reduce it by creating special view extensions. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 

