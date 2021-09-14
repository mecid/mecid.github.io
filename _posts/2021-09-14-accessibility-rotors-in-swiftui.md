---
title: Accessibility rotors in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

SwiftUI Release 3 contains many new APIs that we can utilize to improve accessibility in our apps, and one of them is the new *accessibilityRotor* view modifier. This week we will learn how to use the *accessibilityRotor* view modifier to provide custom VoiceOver navigation using rotors.

#### Basics
Usually, we navigate through the app using the left and right swipes while VoiceOver is on. But sometimes, we need a custom path for navigation through a specific collection of elements. A particular group of elements is called a custom rotor. You can create as many as you need custom rotors in your app.

To use the rotor, rotate two fingers on your iOS device's screen as if you're turning a dial. VoiceOver will say the first rotor option. Keep rotating your fingers to hear more options. Lift your fingers to choose an option. Then flick your finger up or down on the screen to navigate through it.

#### SwiftUI custom rotors API
Assume that we are working on a screen that provides information about health trends. There is a long list of different trends, including positive, negative, and constant results. Negative trends are the things the user should focus on improving. That's why we should build a custom rotor to navigate only through negative trends. Let's start by introducing the *Trend* model and the *TrendsView*.

```swift
import SwiftUI

struct Trend: Identifiable {
    let id = UUID()
    let message: String
    let isPositive: Bool
}

struct TrendView: View {
    let trend: Trend

    var body: some View {
        HStack {
            Image(systemName: trend.isPositive ? "chevron.up" : "chevron.down")
                .accessibilityElement()
                .accessibilityLabel(trend.isPositive ? "positive" : "negative")
            Text(trend.message)
        }.accessibilityElement(children: .combine)
    }
}
```

The code above is super simple. Please look at how we use the *accessibilityElement* view modifier to make *TrendView* accessible and combine all the children's information. 

> To learn about the basics of accessibility in SwiftUI, take a look at my ["Accessibility in SwiftUI"](/2019/09/10/accessibility-in-swiftui/) post.

```swift
struct TrendsView: View {
    let trends: [Trend]

    var body: some View {
        List {
            ForEach(trends, id: \.id) { trend in
                TrendView(trend: trend)
            }
        }
        .accessibilityRotor("Negative trends") {
            ForEach(trends, id: \.id) { trend in
                if !trend.isPositive {
                    AccessibilityRotorEntry(trend.message, id: trend.id)
                }
            }
        }
    }
}
```

Here we have the *TrendsView* that displays the list of trends. We use the *accessibilityRotor* view modifier to create a custom rotor called "Negative trends". As soon as the user focuses on the list, VoiceOver suggests accessing the negative trends rotor.

The first parameter of *accessibilityRotor* view modifier is a label. You can use *String, LocalizedStringKey, or Text* view as a label. VoiceOver uses the title to inform a user about a custom rotor.

The second parameter is the *AccessibilityRotorContentBuilder* closure. The *AccessibilityRotorContentBuilder* function builder is very similar to *ViewBuilder*, but instead of building views, it creates *AccessibilityRotorContent*.

 *AccessibilityRotorContent* is also identical to *View* protocol, but it describes the content of the rotor. SwiftUI types like *ForEach* and *Group* conform both to *View* and *AccessibilityRotorContent* protocols. That's why we can use them both inside *ViewBuilder* and *AccessibilityRotorContentBuilder* closures.

The last piece is the *AccessibilityRotorEntry* conforming to *AccessibilityRotorContent* and allowing us to use it inside a *ForEach* or *Group*. We use it to create a rotor entry and bind it to a SwiftUI view using an ID. This is the point where all the magic takes place. We have two *ForEach* instances, and SwiftUI is smart enough to match the IDs inside *ForEach* and bind rotor entries to the views with the same IDs.

#### accessibilityRotorEntry view modifier
The approach above works like a charm, but we can gain more control over binding rotor entry to a view using the *accessibilityRotorEntry* view modifier.

```swift
struct TrendsView: View {
    let trends: [Trend]

    @Namespace private var customRotorNamespace

    var body: some View {
        List {
            ForEach(trends, id: \.id) { trend in
                VStack {
                    TrendView(trend: trend)
                        .accessibilityRotorEntry(id: trend.id, in: customRotorNamespace)
                    Text(trend.notes)
                }
            }
        }
        .accessibilityRotor("Negative trends") {
            ForEach(trends, id: \.id) { trend in
                if !trend.isPositive {
                    AccessibilityRotorEntry(trend.message, trend.id, in: customRotorNamespace) 
                }
            }
        }
    }
}
```

In this case, we use the *accessibilityRotorEntry* view modifier to bind a rotor entry id to a view explicitly. It allows us to ignore IDs from *ForEach* and can be very helpful in situations where we don't have ForEach or don't need to include the whole child of *ForEach* into the rotor, like in our example.

#### Preparing rotor entries
As a bonus, the *AccessibilityRotorEntry* type allows us to provide a closure to run when the user navigates to a particular rotor entry. For example, we can scroll to the specific list item if it is not visible when navigating to that item using rotors.

```swift
struct TrendsView: View {
    let trends: [Trend]

    @Namespace private var customRotorNamespace

    var body: some View {
        ScrollViewReader { scrollView in
            List {
                ForEach(trends, id: \.id) { trend in
                    TrendView(trend: trend)
                        .accessibilityRotorEntry(id: trend.id, in: customRotorNamespace)
                        .id(trend.id)
                }
            }
            .accessibilityRotor("Negative trends") {
                ForEach(trends, id: \.id) { trend in
                    if !trend.isPositive {
                        AccessibilityRotorEntry(trend.message, trend.id, in: customRotorNamespace) {
                            scrollView.scrollTo(trend.id)
                        }
                    }
                }
            }
        }
    }
}
```

#### TextEditor Rotors
The new accessibility rotors API is robust and allows us to navigate even through text in *TextEditor*. We can mark particular parts of our text content and navigate between them inside the *TextEditor*.

```swift
struct ContentEditor: View {
    @Binding var content: Content

    var body: some View {
        TextEditor(text: $content.text)
            .accessibilityRotor("Emails", textRanges: content.emailRanges)
            .accessibilityRotor("Links", textRanges: content.linkRanges)
    }
}
```

#### Conclusion
SwiftUI Release 3 has done great work in the area of accessibility. Now we can use all these new APIs to build super accessible apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
