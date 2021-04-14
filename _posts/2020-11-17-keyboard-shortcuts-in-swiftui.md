---
title: Keyboard shortcuts in SwiftUI
layout: post
category: Interactions
image: /public/ipad.jpg
---

This year Apple released the new App Lifecycle API for SwiftUI, which brings tons of new modifiers to replace *AppDelegate* callbacks. I have already covered most of them in previous posts. This week, we will discuss the new *keyboardShortcut* modifier, which allows us to assign a shortcut to any interacting view.

{% include friends.html %}

SwiftUI provides us the *keyboardShortcut* modifier that we can attach to any view in the view hierarchy and define a keyboard shortcut. Pressing the defined keyboard shortcut is the equivalent to direct interaction with the view to perform its primary action. Let's take a look at the example.

> To learn more about new App Lifecycle API, take a look at ["Managing app in SwiftUI"](/2020/08/19/managing-app-in-swiftui/) post.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Button("Print message") {
            print("Hello World!")
        }.keyboardShortcut("p", modifiers: [.command, .shift])
    }
}
```

As you can see in the example above, we assign a keyboard shortcut to the button. We define it by using *keyboardShortcut* modifier and passing the "p" key equivalent and a list of modifier keys. A modifier key is an instance of *EventModifiers* struct that conforms to *OptionSet* protocol and defines keys like shift, command, control, option, etc. You can ignore the key modifier parameter, and in this case, SwiftUI will use the command modifier by default.

> To learn more about *OptionSet* in Swift, look at my ["Inclusive enums with OptionSet"](/2019/04/10/inclusive-enums-with-optionset/) post.

This first parameter of *keyboardShortcut* modifier should be an instance of *KeyEquivalent* struct. *KeyEquivalent* struct conforms to *ExpressibleByExtendedGraphemeClusterLiteral* protocol, which allows us to create an instance of the *KeyEquivalent* using a string literal containing only one character. *KeyEquivalent* also defines a few key symbols like arrows, escape, delete, home, etc.

Remember that you have to press the "Capture Keyboard" button in the simulator chrome to test keyboard shortcuts on a simulator.

```swift
import SwiftUI

struct ContentView: View {
    @State private var isEnabled = false

    var body: some View {
        Toggle(isOn: $isEnabled) {
            Text(String(isEnabled))
        }.keyboardShortcut("t")
    }
}
```

I have to mention that *keyboardShortcut* is a view modifier, and you can apply it to any SwiftUI view. In the example above, we define a keyboard shortcut on the toggle view. By pressing this shortcut, we interact with the toggle and switch its value.

> To learn more about view modifiers, take a look at my ["ViewModifiers in SwiftUI"](/2019/08/07/viewmodifiers-in-swiftui/) post.

You can also attach the *keyboardShortcut* modifier to any container view like *VStack* or *HStack*. In this case, the shortcut will apply to the first interactive child in the container hierarchy.

```swift
import SwiftUI

struct ContentView: View {
    @State private var isEnabled = false

    var body: some View {
        VStack {
            Button("Print message") {
                print("Hello World!")
            }
            
            Button("Delete message") {
                print("Message deleted.")
            }
        }.keyboardShortcut("p")
    }
}
```

#### Conclusion
This week we learned about implementing keyboard shortcuts in your SwiftUI apps. Keyboard shortcuts can improve your app user experience in a great way. It is effortless to implement in SwiftUI, and I believe you will not waste your time and add it as soon as possible. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!