---
title: Text field enhancements in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/text-field.png
---

From the very first release of the SwiftUI framework, text fields were a weak point of the framework. Over the years, Apple introduced a few enhancements to text fields to improve the developer experience. This week, we will talk about the improvements that SwiftUI introduced for text fields.

{% include friends.html %}

One of the most common issues of the *TextField* type in SwiftUI was the ability to increase the size of the view whenever text doesn’t fit on a single line. There were a bunch of tricks to achieve this behaviour, but lately SwiftUI introduced the *axis* parameter for the *TextField* type, allowing us to scroll the text field whenever its content doesn’t fit available space.

```swift
struct ContentView: View {
    @State private var text = ""
    
    var body: some View {
        TextField("Very long text", text: $text, axis: .vertical)
    }
}
```

As you can see in the example above, we use the *axis* parameter while defining the text field to allow the view to enlarge to fit its content. Whenever it doesn’t fit the content, it becomes scrollable in the axis you provide.

> To learn more about the basics of text fields in SwiftUI, take a look at my ["TextField in SwiftUI"](/2020/02/26/textfield-in-swiftui/) post.

Another common issue related to text fields in SwiftUI is the opportunity to programmatically read and write the text selection. SwiftUI introduced another parameter called *selection*, allowing us to bind the text field selection to a property of type *TextSelection*.

```swift
struct ContentView: View {
    @State private var text = ""
    @State private var selection: TextSelection?
    
    var body: some View {
        TextField("Very long text here", text: $text, selection: $selection)
    }
}
```

As soon as the user selects the text, SwiftUI activates the binding and provides an instance of the *TextSelection* type. You can also provide an instance of the *TextSelection* type via binding to programmatically select the text. I’m happy to share that the same selection API is also available for the *TextEditor* type.

My favorite enhancement related to the text fields in SwiftUI is the suggestions API. Unfortunately, it is available only on macOS, but I hope to see this API available on all possible Apple platforms during next year.

```swift
struct ContentView: View {
    @State private var text = ""
    
    var body: some View {
        TextField("Very long text here", text: $text)
            .textInputSuggestions {
                Label("Suggestion 1", systemImage: "01.circle")
                    .textInputCompletion("text1")
                Label("Suggestion 2", systemImage: "02.circle")
                    .textInputCompletion("text2")
                Label("Suggestion 3", systemImage: "03.circle")
                    .textInputCompletion("text3")
            }
    }
}
```

SwiftUI introduced the *textInputSuggestions* view modifier, allowing us to provide text suggestions for any text field. In the *ViewBuilder* closure, we can provide views that will appear under the text field. Whenever the user taps any of the provided suggestions, SwiftUI replaces the text in the text field with the text you provide via the *textInputCompletion* view modifier.

The last thing I was to talk about is the Writing Tools features that Apple introduced across all text fields on Apple platforms. SwiftUI supports Writing Tools automatically for any text field and allows you to control its availability by using the *writingToolsBehavior* view modifier.

```swift
struct ContentView: View {
    @State private var text = ""
    
    var body: some View {
        TextField("Very long text here", text: $text)
            .writingToolsBehavior(.limited)
    }
}
```

This week, we discussed the new APIs introduced by the SwiftUI framework to enhance the developer experience with text fields. I hope you find the post enjoyable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
