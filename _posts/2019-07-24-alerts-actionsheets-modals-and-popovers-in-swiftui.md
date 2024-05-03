---
title: Alerts, Action Sheets, Modals and Popovers in SwiftUI
layout: post
category: Mastering SwiftUI views
---

SwiftUI has a set of dedicated modifiers for presenting sheets, alerts, action sheets, and popovers. Let's take a look at how we can use them to display views in different ways.

{% include friends.html %}

#### Alerts and Action Sheets
Both alerts and action sheets use the similar ways of presenting it to the user. Let's start with a simpler one. We have to describe a boolean binding which can be observed by SwiftUI, and as soon as boolean becomes true, SwiftUI presents the action sheet or alert.

```swift
struct EditorView: View {
    @Binding var message: String
    @State private var dialogShown = false
    
    var body: some View {
        Form {
            TextField("Message", text: $message)
            Button("Save") {
                dialogShown = true
            }
            .actionSheet(isPresented: $dialogShown) {
                ActionSheet(
                    title: Text("Are you sure?"),
                    buttons: [
                        .default(Text("Save")) { print("Saving...") },
                        .cancel()
                    ]
                )
            }
        }
    }
}
```
As you can see in the example above to present an action sheet, we use *actionSheet* modifier bound to a boolean value and a closure which creates an action sheet. Alternatively, to display an alert, we have to use *alert* modifier instead.

> *ActionSheet* is deprecated now, use confirmation dialogs instead. To learn more, take a look at my ["Confirmation dialogs in SwiftUI"](/2021/07/28/confirmation-dialogs-in-swiftui/) post.

The interesting fact here is that SwiftUI resets the binding to initial value after alert or action sheet dismissal.

It is a straightforward approach to present alerts or action sheets. But sometimes it is not enough, because we need some data to show in alert. For this case, we have another versions of the *alert* view modifier, which use an optional **presenting** parameter.

```swift
struct EditorView: View {
    @Binding var message: String
    @State private var dialogShown = false
    
    var body: some View {
        Form {
            TextField("Message", text: $message)
            Button("Save") {
                dialogShown = true
            }
            .alert(
                "Are you sure?",
                isPresented: $dialogShown,
                presenting: message
            ) { message in
                Button("Save") {
                    print("Saving...")
                }
                
                Button("Cancel") {
                    print("Cancel...")
                }
            } message: { message in
                Text("Do you want to save: \(message)?")
            }
        }
    }
}
```

As you can see in the example above, the *alert* view modifier provides additional parameter *presenting* allowing us to pass the value we want to display in the alert message and actions builder. 

#### Sheets
To present modals, SwiftUI provides the special view modifier called *sheet*. *Sheet* view modifier is very similar to *alert* and *actionSheet*, it uses boolean or optional identifiable binding to understand when to present a sheet. It also needs a closure which returns a content view for a sheet.

Besides that, *sheet* view modifier has an optional *onDismiss* closure parameter, SwiftUI calls this closure after modal dismiss. Like with alerts, SwiftUI will reset binding to the initial value after modal dismissal.

```swift
struct MasterView: View {
    @State private var showModal = false
    
    var body: some View {
        VStack {
            Button("Show modal") {
                self.showModal = true
            }
        }
        .sheet(isPresented: $showModal) {
            print(showModal)
        } content: {
            ModalView(message: "This is Modal view")
        }
    }
}

struct ModalView: View {
    @Environment(\.dismiss) private var dismiss
    let message: String
    
    var body: some View {
        VStack {
            Text(message)
            Button("Dismiss") {
                dismiss()
            }
        }
    }
}
```
You can also use *fullScreenCover* view modifier to present full screen modals. It works the same way as the *sheet* view modifier. The *dismiss* property is an environment value that allows us to dismiss the current presentated view. We can use it to programmatically dismiss the sheet. 

#### Popovers
Using popovers in SwiftUI is very similar to alerts and action sheets. *Popover* modifier also has two overloads for boolean and optional identifiable bindings. Another additional parameter in the *popover* view modifier is *arrowEdge*, by providing *Edge* value you can draw an arrow in a specified direction. Here is the example of the *popover* view modifier usage.

```swift
struct MasterView: View {
    @State private var showPopover: Bool = false
    
    var body: some View {
        VStack {
            Button("Show popover") {
                self.showPopover = true
            }
            .popover(
                isPresented: $showPopover,
                arrowEdge: .bottom
            ) {
                Text("Popover")
            }
        }
    }
}
```

#### Conclusion
As you can see, SwiftUI provides a pretty easy way of presenting context-related views like alerts, action sheets, sheets, and popoversp by using bindings. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  
