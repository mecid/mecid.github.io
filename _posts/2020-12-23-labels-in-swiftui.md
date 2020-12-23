---
title: Labels in SwiftUI
layout: post
image: /public/swiftui.png
category: Mastering SwiftUI views
---

We often underestimate the power of simple things. The same feelings I had during the first usage of Label view in SwiftUI. It looks straightforward, but it hides the many use cases where it works great. Today we will talk about the Label view and its customization capabilities.

#### Basics
Label is a standard component for user interface items, consisting of an icon with a title. One of the most common and recognizable user interface components is the combination of an icon and a label. Let's take a look at how easily we can use it in code.

```swift
struct ContentView: View {
    var body: some View {
        // SF Symbol
        Label("Heart Rate", systemImage: "heart.fill")
        
        // Image from app bundle
        Label("ECG", image: "ecg")
    }
}
```

As you can see, we create a label by passing two parameters to the initializer. The first one is a title that we can pass as a plain string or LocalizedStringKey. The second one is an image, a system image from the SF Symbols collection, or your custom image from the app bundle.

Another initializer overload allows you to define a label using two ViewBuilder closures to build the Label view.

```swift
struct ContentView: View {
    var body: some View {
        Label {
            Text("Hello")
        } icon: {
            Image(systemName: "heart")
        }
    }
}
```

The main thing that I love about the Label view is how it deals with accessibility. It combines both image and title in a single accessibility element and uses the title as an accessibility label for the whole view. This is usually what we want to achieve while using a stack with an image and text describing that image.

#### Styling
SwiftUI provides us LabelStyle protocol to implement different styling options for our labels. There are already two style options coming with SwiftUI out of the box. You might be already familiar with style protocols in SwiftUI, as I already covered it on my blog a few times.

SwiftUI gives us DefaultLabelStyle, IconOnlyLabelStyle and TitleOnlyLabelStyle style options to use out of the box. The default one is DefaultLabelStyle that shows both title and image. IconOnlyLabelStyle shows only the image, and TitleOnlyLabelStyle shows the title only.

```swift
struct ContentView: View {
    var body: some View {
        Label("Heart Rate", systemImage: "heart.fill")
            .labelStyle(IconOnlyLabelStyle())
        Label("ECG", image: "ecg")
            .labelStyle(TitleOnlyLabelStyle())
    }
}
```

The great thing about style protocols is that you can create your own implementation and place the items the way you need. Let's try to build another style that will place the icon and title in a horizontal or vertical stack depending on the current accessibility category.

```swift
struct AccessibleLabelStyle: LabelStyle {
    @Environment(\.sizeCategory) var sizeCategory

    @ViewBuilder
    func makeBody(configuration: Configuration) -> some View {
        if sizeCategory.isAccessibilityCategory {
            VStack {
                configuration.icon
                configuration.title
            }
        } else {
            HStack {
                configuration.icon
                configuration.title
            }
        }
    }
}

struct ContentView: View {
    var body: some View {
        Button(action: {}) {
            Label("Heart Rate", systemImage: "heart.fill")
        }.labelStyle(AccessibleLabelStyle())
    }
}
```

As you can see in the example above, we create a custom label style called AccessibleLabelStyle. It uses the environment to read the size category and layout items appropriately. We use labelStyle modifier to set and share the style for labels using the environment.

#### Conclusion
Today we learned about another great SwiftUI view that provides us a lot of ways for customization and styling. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!