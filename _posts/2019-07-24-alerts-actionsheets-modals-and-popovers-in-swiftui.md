---
title: Alerts, Action Sheets, Modals and Popovers in SwiftUI
layout: post
category: Mastering SwiftUI views
---

Last week we talked about [Navigation in SwiftUI](/2019/07/17/navigation-in-swiftui/). This week I want to continue the topic with *Modals*, *Alerts*, *Action Sheets*, and *Popovers*. SwiftUI views have a dedicated modifiers for presenting this kind stuff. Let's take a look at how we can use modifiers to display *Modals*, *Alerts*, *Action Sheets*, and *Popovers*.

{% include friends.html %}

#### Alerts and Action Sheets
Both *Alerts* and *Action Sheets* use the similar two ways of presenting it to the user. Let's start with a simpler one. We have to describe a *Boolean* binding which can be observed by SwiftUI, and as soon as *Boolean* is true, SwiftUI presents the *Action Sheet* or *Alert*.

```swift
struct MasterView: View {
    @State private var showActionSheet = false

    var body: some View {
        VStack {
            Button("Show action sheet") {
                self.showActionSheet = true
            }
        }.actionSheet(isPresented: $showActionSheet) {
            ActionSheet(
                title: Text("Actions"),
                message: Text("Available actions"),
                buttons: [
                    .cancel { print(self.showActionSheet) },
                    .default(Text("Action")),
                    .destructive(Text("Delete"))
                ]
            )
        }
    }
}
```
As you can see in the example above to present an action sheet, we use *actionSheet* modifier bound to a *Boolean* value and a closure which creates an action sheet. Alternatively, to display an alert, we need to use *alert* modifier instead.

The interesting fact here is that SwiftUI resets the binding to initial value after *Alert* or *Action Sheet* dismiss. To learn more about *Property Wrappers* available in SwiftUI, take a look at ["Understanding Property Wrappers in SwiftUI" post](/2019/06/12/understanding-property-wrappers-in-swiftui/).

It is a straightforward approach to present *Alerts* or *Action Sheets*. But sometimes it is not enough, because we need some data to show in *Alert* or *Action Sheet*. For this case, we have another overload of *alert* and *actionSheet* modifiers, which uses *Optional Identifiable binding* instead of *Boolean binding*.

```swift
struct Message: Identifiable {
    let id = UUID()
    let text: String
}

struct MasterView: View {
    @State private var message: Message? = nil

    var body: some View {
        VStack {
            Button("Show alert") {
                self.message = Message(text: "Hi!")
            }
        }.alert(item: $message) { message in
            Alert(
                title: Text(message.text),
                dismissButton: .cancel()
            )
        }
    }
}
```

As soon as *message* is not *nil* SwiftUI call a closure with *message* as a parameter. You can create your *Alert* based on data passed into the closure. 

#### Modals
To present modals, SwiftUI provides the special view modifier called *sheet*. *Sheet* modifier is very similar to *alert* and *actionSheet*, it uses *Boolean* or *Optional Identifiable* binding to understand when to present a modal. It also needs a closure which returns a content view for a modal. Besides that, *sheet* modifier has an optional *onDismiss* closure parameter, SwiftUI calls this closure after modal dismiss. Like with alerts, SwiftUI will reset binding to the initial value after modal dismiss.

```swift
import SwiftUI
import Combine

struct MasterView: View {
    @State private var showModal = false

    var body: some View {
        VStack {
            Button("Show modal") {
                self.showModal = true
            }
        }.sheet(isPresented: $showModal, onDismiss: {
            print(self.showModal)
        }) {
            ModalView(message: "This is Modal view")
        }
    }
}

struct ModalView: View {
    @Environment(\.presentationMode) var presentation
    let message: String

    var body: some View {
        VStack {
            Text(message)
            Button("Dismiss") {
                self.presentation.value.dismiss()
            }
        }
    }
}
```

*presentationMode* is an *Environment* binding to the current *PresentationMode* of this view. We can use it to programmatically dismiss the *Modal*. To learn more about *Property Wrappers provided by SwiftUI and Environment values*, you can check my ["Understanding Property Wrappers in SwiftUI" post](/2019/06/12/understanding-property-wrappers-in-swiftui/).

You can also use *fullScreenCover* view modifier to present full screen modals. It works the same way as *sheet* modifier.

#### Popovers
Using Popovers in SwiftUI is very similar to *Alers* and *Action Sheets*. *Popover* modifier also has two overloads for *Boolean* and *Optional Identifiable* bindings. Another additional parameter in *popover modifier* is *arrowEdge*, by providing *Edge* value you can draw an arrow in a specified direction. Here is the example of *Popover* modifier usage.

```swift
struct MasterView: View {
    @State private var showPopover: Bool = false

    var body: some View {
        VStack {
            Button("Show popover") {
                self.showPopover = true
            }.popover(
                isPresented: self.$showPopover,
                arrowEdge: .bottom
            ) { Text("Popover") }
        }
    }
}
```

#### Conclusion
As you can see, SwiftUI provides a pretty easy way of presenting context-related views like *Alerts*, *Action Sheets*, *Modals*, and *Popovers* by using *bindings*. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  

