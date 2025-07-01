---
title: Glassifying toolbars in SwiftUI
layout: post
image: /public/glassy-toolbar-2.png
category: Mastering SwiftUI views
---

Liquid Glass is the new design language Apple using across all of its platforms. The look and feel of tabs were the major change that we covered last week. This week we will focus on another significant change related to toolbars.

{% include friends.html %}

Toolbars changed significantly in Liquid Glass. They have a new glassy background, they can be split into groups, etc. Fortunately, similar to the tabs API, the toolbar API doesn’t change a lot and takes the Liquid Glass look and feel without any code change. But you still can use the new APIs to fine-tune behaviour. 

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                Text("Item 1")
                Text("Item 2")
                Text("Item 3")
            }
            .navigationTitle("Items")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel", systemImage: "xmark") {
                        
                    }
                }
                
                ToolbarItem(placement: .confirmationAction) {
                    Button("Done", systemImage: "checkmark") {
                        
                    }
                }
            }
        }
    }
}
```

As you can see in the example above, we use the same *toolbar* view modifier in pair with the *ToolbarItem* type. The new design language moves away from text-based toolbar items to symbol-based. So, remember to use buttons with both images and text labels.

![glassy-toolbar](/public/glassy-toolbar-1.png)

Whenever you support platforms before Liquid Glass, you may need to keep the old text-based toolbar items. You can easily achieve that by using the *labelStyle* view modifier.

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                Text("Item 1")
                Text("Item 2")
                Text("Item 3")
            }
            .navigationTitle("Items")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel", systemImage: "xmark") {
                        
                    }
                    .labelStyle(.toolbar)
                }
                
                ToolbarItem(placement: .confirmationAction) {
                    Button("Done", systemImage: "checkmark") {
                        
                    }
                    .labelStyle(.toolbar)
                }
            }
        }
    }
}
```

As you can see, we use the *labelStyle* view modifier with *toolbar* style. The *toolbar* style doesn’t come as the part of SwiftUI framework; instead, we create it manually.

```swift
struct ToolbarLabelStyle: LabelStyle {
    func makeBody(configuration: Configuration) -> some View {
        if #available(iOS 26, *) {
            Label(configuration)
        } else {
            Label(configuration)
                .labelStyle(.titleOnly)
        }
    }
}

@available(iOS, introduced: 18, obsoleted: 26, message: "Remove this property in iOS 26")
extension LabelStyle where Self == ToolbarLabelStyle {
    static var toolbar: Self { .init() }
}
```

It is pretty easy to define a custom label style. We check the availability of the Liquid Glass and apply necessary styling. We also mark the style as obsolete in iOS 26, which will be an error when you change the target of your project to iOS 26, so it becomes unnecessary and you can remove it.

*ToolbarItemPlacement* become really important while adopting Liquid Glass, because it controls not only placement but also the way it looks. For example, we use the *confirmationAction* placement, and it applies the *glassProminent* button style.

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                Text("Item 1")
                Text("Item 2")
                Text("Item 3")
            }
            .navigationTitle("Items")
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel", systemImage: "xmark") {
                        
                    }
                    .labelStyle(.toolbar)
                    .tint(.red)
                }
                
                ToolbarItem(placement: .confirmationAction) {
                    Button("Done", systemImage: "checkmark") {
                        
                    }
                    .labelStyle(.toolbar)
                    .badge(3)
                }
            }
        }
    }
}
```

Liquid Glass allows us to tint toolbar items using the *tint* view modifier and badge them using the *badge* view modifier. But use it wisely, it is not something you are going to use often, instead rely on toolbar placement API.

The new Liquid Glass provides us the new toolbar grouping functionality. For example, you can group a bunch of secondary actions together by splitting them from primary action.

```swift
struct ToolsToolbar: ToolbarContent {
    var body: some ToolbarContent {
        ToolbarItem(placement: .cancellationAction) {
            Button("Cancel", systemImage: "xmark") {}
        }
        
        ToolbarItemGroup(placement: .primaryAction) {
            Button("Draw", systemImage: "pencil") {}
            Button("Erase", systemImage: "eraser") {}
        }
        
        ToolbarSpacer(.flexible)
        
        ToolbarItem(placement: .confirmationAction) {
            Button("Save", systemImage: "checkmark") {}
        }
    }
}
```

SwiftUI introduced the new *ToolbarSpacer* type allowing us to split toolbar items and place the space between them. You can apply *fixed* or *flexible* spacing between toolbar items.

![glassy-toolbar](/public/glassy-toolbar-2.png)

 Today, we learned how to make our toolbars supporting the new Liquid Glass design language. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
