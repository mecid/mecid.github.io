---
title: Alerts, ActionSheets, Modals and Popovers in SwiftUI
layout: post
---

Last week we talked about [Navigation in SwiftUI](/2019/07/17/navigation-in-swiftui/). This week I want to continue the topic with *Modals*, *Alerts*,  *ActionSheets*, and *Popovers*. *SwiftUI* views have a dedicated modifier for presenting this kind stuff called *presentation*. Let's take a look at how we can use *presentation* modifier to display *Alerts*, *ActionSheets*, and *Modals*.

#### Alerts and ActionSheets
Both *Alerts* and *ActionSheets* use the same two ways of presenting it to the user. Let's start with a simpler one. We have to describe a *Boolean* binding which can be observed by *SwiftUI*, and as soon as *Boolean* is true, *SwiftUI* presents the *ActionSheet* or *Alert*.

```swift
struct MasterView: View {
    @State private var showActionSheet = false

    var body: some View {
        VStack {
            Button("Show action sheet") {
                self.showActionSheet = true
            }
        }
            .presentation($showActionSheet) {
                ActionSheet(
                    title: Text("Actions"),
                    message: Text("Available actions"),
                    buttons: [
                        .cancel { self.showActionSheet = false },
                        .default(Text("Action")),
                        .destructive(Text("Delete"))
                    ]
                )
            }
    }
}
```

It is a straightforward approach to present *Alerts* or *ActionSheets*. But sometimes it is not enough, because we need some data to show in *Alert* or *ActionSheet*. For this case, we have another overload of *presentation* modifier, which uses *Optional Identifiable binding* instead of Boolean binding.

```swift
struct Message: Identifiable {
    let id: UUID
    let content: String
}

struct MasterView: View {
    @State private var showMessageAlert: Message? = nil

    var body: some View {
        VStack {
            Button("Show alert") {
                self.showMessageAlert = Message(id: UUID(), content: "Hi!")
            }
        }
            .presentation($showMessageAlert) { message in
                Alert(
                    title: Text(message.content),
                    dismissButton: .cancel()
                )
            }
    }
}
```

As soon as *Identifiable* is not *nil* *SwiftUI* call a closure with *Identifiable* as a parameter. You can create your *Alert* or *ActionSheet* based on data passed into the closure. 

#### Modals
To present a modal, you have to pass an instance of *Modal* struct to *presentation* modifier and to hide you can pass a *nil* value to the same modifier.

```swift
struct MasterView: View {
    @State private var modal: Modal? = nil

    var body: some View {
        VStack {
            Button("Show modal") {
                self.modal = Modal(Text("Modal")) {
                    self.modal = nil
                }
            }
        }
            .presentation(modal)
    }
}
```

In the example above, I use *@State Property Wrapper*, as soon as this property changes *SwiftUI* rebuilds view with the new value. To learn more about *Property Wrappers* available in *SwiftUI*, take a look at ["Understanding Property Wrappers in SwiftUI" post](/2019/06/12/understanding-property-wrappers-in-swiftui/). I create a modal by using the provided init method and passing there the view which represents the modal and closure which *SwiftUI* runs after dismiss. Another way of presenting *Modals* is *PresentationLink* component. We covered it in the previous [post for more information please check it](/2019/07/17/navigation-in-swiftui/).

#### Popovers
Using Popovers in *SwiftUI* very similar to *Alers* and *ActionSheets*, the only difference here is the usage of *popover* modifier instead of the *presentation*. Popover modifier also has two overloads for *Boolean* and *Optional Identifiable* bindings. Another additional parameter in popover modifier is *arrowEdge*, by *Edge* value you can draw an arrow in a specified direction. Here is the example of *Popover* modifier usage.

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
As you can see, *SwiftUI* provides a pretty easy way of presenting context-related views like *Alerts*, *ActionSheets*, *Modals*, and *Popovers* by using *bindings*. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  

