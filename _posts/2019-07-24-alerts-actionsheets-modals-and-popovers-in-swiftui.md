---
title: Alerts, Action Sheets, Modals and Popovers in SwiftUI
layout: post
category: Mastering SwiftUI views
---

Last week we talked about [Navigation in SwiftUI](/2019/07/17/navigation-in-swiftui/). This week I want to continue the topic with sheets, alerts, action sheets, and popovers. SwiftUI has a set of dedicated modifiers for presenting this kind of stuff. Let's take a look at how we can use different view modifiers to display sheets, alerts, action sheets, and popovers.

{% include friends.html %}

#### Alerts and Action Sheets
Both alerts and action sheets use the similar ways of presenting it to the user. Let's start with a simpler one. We have to describe a boolean binding which can be observed by SwiftUI, and as soon as boolean becomes true, SwiftUI presents the action sheet or alert.

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
As you can see in the example above to present an action sheet, we use *actionSheet* modifier bound to a boolean value and a closure which creates an action sheet. Alternatively, to display an alert, we have to use *alert* modifier instead.

The interesting fact here is that SwiftUI resets the binding to initial value after *Alert* or *Action Sheet* dismissal.

> To learn more about *Property Wrappers* available in SwiftUI, take a look at ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/) post.

It is a straightforward approach to present alerts or action sheets. But sometimes it is not enough, because we need some data to show in an alert or action sheet. For this case, we have another versions of *alert* and *actionSheet* modifiers, which use an optional identifiable binding instead of boolean binding.

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

As soon as *message* is not *nil* SwiftUI call a closure with *message* as a parameter. You can create your alert based on the data passed into the closure. 

#### Sheets
To present modals, SwiftUI provides the special view modifier called *sheet*. *Sheet* view modifier is very similar to *alert* and *actionSheet*, it uses boolean or optional identifiable binding to understand when to present a sheet. It also needs a closure which returns a content view for a sheet. Besides that, *sheet* view modifier has an optional *onDismiss* closure parameter, SwiftUI calls this closure after modal dismiss. Like with alerts, SwiftUI will reset binding to the initial value after modal dismissal.

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
You can also use *fullScreenCover* view modifier to present full screen modals. It works the same way as *sheet* modifier. *presentationMode* is an environment value that represents the current presentation mode of the view. We can use it to programmatically dismiss the sheet. 

#### Popovers
Using popovers in SwiftUI is very similar to alerts and action sheets. *Popover* modifier also has two overloads for boolean and optional identifiable bindings. Another additional parameter in the *popover* view modifier is *arrowEdge*, by providing *Edge* value you can draw an arrow in a specified direction. Here is the example of the *popover* view modifier usage.

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
As you can see, SwiftUI provides a pretty easy way of presenting context-related views like alerts, action sheets, sheets, and popoversp by using bindings. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  

