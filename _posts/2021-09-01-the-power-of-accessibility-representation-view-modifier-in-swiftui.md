---
title: The power of accessibilityRepresentation view modifier in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

The SwiftUI Release 3 has a lot of improvements in the area of accessibility. It provides all the missing APIs like accessibility rotors, but it also gives us new ways of doing complex things efficiently. This week we will talk about the *accessibilityRepresentation* view modifier that allows us to replace accessibility elements of one view with another.

Assume we are working on a super custom checkmark button that toggles its state only after a long press. Let's take a look at the code we could write to implement this view.

```swift
import SwiftUI

struct LongPressCheckmark: View {
    @Binding var isSelected: Bool

    var body: some View {
        Image(systemName: isSelected ? "checkmark.rectangle" : "rectangle")
            .onLongPressGesture { isSelected.toggle() }
    }
}
```

The code above looks simple. We have a binding to a boolean value indicating the selection of the checkmark. We also have an image presenting a checkmark or empty rectangle depending on the value of the binding. I added a long-press gesture to the image that toggles the boolean binding.

The main downside of the code above is the accessibility support. VoiceOver recognizes the image and doesn't provide information about the selection state, possible actions, etc. We can improve the accessibility support manually by using the set of provided accessibility view modifiers in SwiftUI.

```swift
import SwiftUI

struct LongPressCheckmark: View {
    @Binding var isSelected: Bool

    var body: some View {
        Image(systemName: isSelected ? "checkmark.rectangle" : "rectangle")
            .onLongPressGesture { isSelected.toggle() }
            .accessibilityRemoveTraits(.isImage)
            .accessibilityAddTraits(.isButton)
            .accessibilityAddTraits(isSelected ? .isSelected : [])
            .accessibilityLabel(Text("Checkmark"))
            .accessibilityHint("You can toggle the checkmark")
            .accessibilityAction { isSelected.toggle() }
    }
}
```

> To learn about the basics of accessibility in SwiftUI, take a look at my ["Accessibility in SwiftUI"](/2019/09/10/accessibility-in-swiftui/) post.

Here we add accessibility modifiers to provide information about the current state of the checkmark, an accessibility action to toggle the state, and the accessibility label with the hint. We have much more accessibility-related lines of code than button logic. Fortunately, SwiftUI provides us a way to simplify the code above by using the *accessibilityRepresentation* view modifier.

```swift
import SwiftUI

struct LongPressCheckmark: View {
    @Binding var isSelected: Bool

    var body: some View {
        Image(systemName: isSelected ? "checkmark.rectangle" : "rectangle")
            .onLongPressGesture { isSelected.toggle() }
            .accessibilityRepresentation {
                Toggle(isOn: $isSelected) {
                    Text("Checkmark")
                }
            }
    }
}
```

As you can see, we replace all the accessibility-related lines of code with the single *accessibilityRepresentation* view modifier. *accessibilityRepresentation* view modifier replaces the accessibility element of the current view with the accessibility information of the view you provide in the closure. SwiftUI doesn't render the view you provide in the closure. SwiftUI uses it only for generating accessibility information.

Whenever you build a custom view that has logic similar to the view in SwiftUI, you can use the *accessibilityRepresentation* view modifier to generate accessibility behavior for your custom view automatically from that SwiftUI view.

In the previous example, we used the standard *Toggle* to extract its accessibility information. I should mention that the *accessibilityRepresentation* view modifier works both with plain views and complex view hierarchies. In the following example, we will build a custom bar chart view.

```swift
struct Bar: Identifiable {
    let id: UUID
    let value: Double
    let label: String
}

struct BarChartView: View {
    let bars: [Bar]

    var body: some View {
        Canvas {
            // custom drawing here
        }.accessibilityRepresentation {
            HStack {
                ForEach(bars) { bar in
                    Rectangle()
                        .accessibilityLabel(bar.label)
                        .accessibilityValue("\(bar.value)")
                }
            }
        }
    }
}
```

Here we use the new *Canvas* view for custom drawing. *Canvas* view doesn't support accessibility out of the box. Fortunately, we can use the *accessibilityRepresentation* view modifier to generate the accessibility information for our chart from the view hierarchy with *HStack*.

The new *accessibilityRepresentation* view modifier drastically simplifies the accessibility support for custom views. Remember that accessibility isn't a feature or a "nice to have". It's a necessity. So let's make your app accessible for everyone. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

