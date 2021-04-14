---
title: Navigation in SwiftUI
post: layout
category: Mastering SwiftUI views
---

This week I want to talk about *Navigation in SwiftUI*. SwiftUI provides both declarative and imperative ways of implementing navigation in your apps. Today we will cover two ways of *Master-Detail* navigation available in SwiftUI. We will learn how use *NavigationLink*, and how to do programmatic navigation in SwiftUI.

{% include friends.html %}

#### Master-Detail flow
Assume that you are working on app which shows a list of some items and you want to move to details screen as soon as the user selects any item. For this type of navigation, SwiftUI provides *NavigationView* and *NavigationLink* components. Let's check how we can use them.

```swift
struct MasterView: View {
    private let messages = [
        "Hello", "How are you?"
    ]

    var body: some View {
        NavigationView {
            List(messages, id: \.self) { message in
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

Here we have a list of messages, to make navigation possible we embed our *List* into a *NavigationView*, it adds standard *NavigationBar* on top of our *List*. We can also set text as a title of *NavigationBar* by adding *navigationBarTitle* modifier to a *List*. Please make sure that you add the *navigationBarTitle* modifier to a *List* component, and not to a *NavigationView* because multiple views can share same *NavigationView* and every view can have a different title. 

**Hidden gem: You can embed two views into NavigationView to achieve Split on iPadOS and MacOS**

Next, we embed List rows into *NavigationLink*, while creating *NavigationLink*, we have to provide a destination view. SwiftUI presents a destination view when the user presses the *List row*. By wrapping *List row* into a *NavigationLink*, SwiftUI adds trailing arrow to the view which indicates that there is a details screen next to the view. And this is where the real power of declarative programming comes. *List row* starts appearing in another way only by embedding it into *NavigationLink*. To learn more about environment-based appearance in SwiftUI, you can check out ["Building forms with SwiftUI" post](/2019/06/19/building-forms-with-swiftui/).

#### Programmatic navigation

Sometimes we need to check multiple conditions and after that push some view. For this kind of situations, *NavigationLink* provides a different way of initialization, which binds it to some value, and as soon as you set the tagged value to the binding, it pushes the view. Let's take a look at the code sample.

```swift
import SwiftUI

struct Master: View {
    @State var selection: Int? = nil
    
    var body: some View {
        NavigationView {
            VStack {
                NavigationLink(destination: Details(), tag: 1, selection: $selection) {
                    Button("Press me") {
                        self.selection = 1
                    }
                }
            }
        }
    }
}

struct Details: View {
    @Environment(\.presentationMode) var presentation

    var body: some View {
        Group {
            Text("Details")
            Button("pop back") {
                self.presentation.wrappedValue.dismiss()
            }
        }
    }
}
```

As you can see in the example above, we create *NavigationLink* by passing destination view and two additional parameters *Hashable* tag and binding to the *Hashable*. As soon as the value of binding is equal to tag, *NavigationView* pushes *NavigationLink*. You can programmatically pop back by using *dismiss* method on *EnvironmentValue* called *presentationMode*.

#### Conclusion
As you can see, SwiftUI provides both imperative and declarative ways of pushing views into navigation stack by using *NavigationLink*. Feel free to use a way of navigation which fits best to your app requirements. Follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!