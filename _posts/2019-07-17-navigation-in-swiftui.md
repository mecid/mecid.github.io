---
title: Navigation in SwiftUI
post: layout
---

This week I want to talk about *Navigation in SwiftUI*. *SwiftUI* provides a declarative way of implementing navigation in your apps. Today we will cover different navigation flows available in *SwiftUI* like *Master-Detail* and *Presenting Modals*.

#### Master-Detail flow
Assume that you are working on app which shows a list of some items and you want to move to details screen as soon as the user selects any item. For this type of navigation, *SwiftUI* provides *NavigationView* and *NavigationLink* components. Let's check how we can use them.

```swift
struct MasterView: View {
    private let messages = [
        "Hello", "How are you?"
    ]

    var body: some View {
        NavigationView {
            List(messages.identified(by: \.self)) { message in
                NavigationLink(destination: DetailsView(message: message)) {
                    Text(message)
                }
            }.navigationBarTitle("Messages")
        }
    }
}

struct DetailsView: View {
    let message: String

    var body: some View {
        VStack {
            Text(message)
                .font(.largeTitle)
        }
    }
}
```

Here we have a list of messages, to make navigation possible we embed our *List* into a *NavigationView*, it adds standard *NavigationBar* on top of our *List*. We can also set text as a title of *NavigationBar* by adding *navigationBarTitle* modifier to a *List*. Please make sure that you add the *navigationBarTitle* modifier to a *List* component, and not to a *NavigationView* because multiple views can share same *NavigationView* and every view can have a different title. **Hidden gem: You can embed two views into NavigationView to achieve Split on iPadOS and MacOS**

Next, we embed List rows into *NavigationLink*, while creating *NavigationLink*, we have to provide a destination view. *SwiftUI* presents a destination view when the user presses the *List row*. By wrapping *List row* into a *NavigationLink*, *SwiftUI* adds trailing arrow to the view which indicates that there is a details screen next to the view. And this is where the real power of declarative programming comes. *List row* starts appearing in another way only by embedding it into *NavigationLink*. To learn more about environment-based appearance in *SwiftUI*, you can check out ["Building forms with SwiftUI" post](/2019/06/19/building-forms-with-swiftui/).

#### Modals
Let's change our navigation a little bit. Instead of showing our view as a child in the navigation hierarchy, I want to present it as a modal using new iOS 13 cart interface. To make it possible, all we need to do is embedding *List row* into *PresentationLink* instead of *NavigationLink*.

```swift
struct MasterView: View {
    private let messages = [
        "Hello", "How are you?"
    ]

    var body: some View {
        NavigationView {
            List(messages.identified(by: \.self)) { message in
                PresentationLink(destination: DetailsView(message: message)) {
                    Text(message)
                }
            }.navigationBarTitle("Messages")
        }
    }
}
```

Now, the user gets an excellent card interface which we have in iOS 13. You can dismiss it by swiping down. But what if you want to add a button which dismisses the modal? To do that we have to use *Environment Property Wrapper*.

```swift
struct DetailsView: View {
    @Environment(\.isPresented) var isPresented
    let message: String

    var body: some View {
        VStack {
            Text(message)
                .font(.largeTitle)
            Button("Close") {
                self.isPresented?.value = false
            }
        }
    }
}
```

*isPresented* is an *Environment* binding to whether *self* is part of a hierarchy that is currently being presented. In other words, *SwiftUI* uses this binding to show/hide presented views. By setting the value of this binding to false, *SwiftUI* dismisses the modal. To learn more about *Property Wrappers provided by SwiftUI and Environment values*, you can check my ["Understanding Property Wrappers in SwiftUI" post](/2019/06/12/understanding-property-wrappers-in-swiftui/).

#### Conclusion
As you can see, there are no manual calls to push or present any views. All you need to do is wrapping your view into *NavigationLink* or *PresentationLink*, and as soon as the user presses it, *SwiftUI* moves it to next destination, and this is the really excellent declarative approach. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!