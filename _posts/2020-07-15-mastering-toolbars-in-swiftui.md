---
title: Mastering toolbars in SwiftUI
layout: post
image: /public/watchOS.png
category: Mastering SwiftUI views
---

Toolbar API is another excellent addition to SwiftUI this year. Usually, we use toolbars to provide available actions. Did you remember the case where you have a button outside of the navigation bar or bottom bar? This week we will learn all about the new Toolbar API.

{% include friends.html %}

In the previous version of SwiftUI, we could place buttons in the navigation bar by using *navigationBarItems* modifier. Let's take a quick look at how it works.

```swift
import SwiftUI

struct ContentView: View {
    let messages: [String]

    var body: some View {
        List(messages, id: \.self) { message in
            Text(message)
        }
        .navigationTitle("Messages")
        .navigationBarItems(trailing: EditButton())
    }
}
```

The example above looks good, but there is one big issue. It works only with the navigation bar. For instance, watchOS apps use *NavigationView* to provide a navigation stack, but there is no navigation bar to deliver actions. We have to use 3D touch menus on the watchOS, but then we face another problem. SwitUI doesn't provide us API to use 3D touch menus.

Fortunately, Apple released the unified Toolbar API that works on all Apple platforms. Toolbar API is another example of a declarative API that adapts to the environment and looks different on different devices. For example, it uses a navigation bar on iOS, but on watchOS, it inserts a button into the top of the screen. Let's take a look at how easily we can use it.

```swift
import SwiftUI

struct ContentView: View {
    let messages: [String]

    var body: some View {
        List(messages, id: \.self) { message in
            Text(message)
        }
        .navigationTitle("Messages")
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                Button("New") {}
            }

            ToolbarItem(placement: .bottomBar) {
                Button("Filter") {}
            }
        }
    }
}
```

As you can see in the example above, SwiftUI provides us the *toolbar* modifier that we can use to build toolbar items. The *toolbar* modifier accepts the *ToolbarContentBuilder* closure, which is very similar to *ViewBuilder* function builder, but instead of views, it uses *ToolbarItems*.

#### ToolbarItem
We use *ToolbarItem* struct to declare an action. *ToolbarItem* has two required parameters. The first one is placement, which is the instance of *ToolbarItemPlacement* struct. The second one is *ViewBuilder* closure that SwiftUI uses to build the view representation of your action.

SwiftUI hides all the magic of toolbars behind *ToolbarItemPlacement* struct. SwiftUI can put your toolbar item in different places, depending on the value of the placement parameter. There are multiple placement opportunities. Let's talk about the essential options.

1. *automatic* - The item is placed in the default section that varies depending on the current platform.
2. *primaryAction* - The item represents a primary action. Usually, SwiftUI places this item in the navigation bar on iOS or on top of other views on watchOS.

There are placement options that we can use only in toolbars presented by a modal view.
1. *confirmationAction* - The item represents a confirmation action for a modal interface. You can use it in your sheets to confirm saving action.
2. *cancellationAction* - The item represents a cancellation action for a modal interface.
3. *destructiveAction* - The item represents a destructive action for a modal interface. You can use it in your modal screens that delete some data.

There are also a bunch of platform-specific placement options.
0. *principal* - The item is placed in the main item section. For example, you can use it to customize the navigation bar's title view on iOS.
1. *bottomBar* - The item is placed in the bottom toolbar. It is available only on iOS.
2. *keyboard* - The item is placed in the keyboard section. On iOS, keyboard items are above the software keyboard when present. On macOS, keyboard items will be placed inside the Touch Bar.
3. *navigationBarLeading* - The item is placed in the leading area of the navigation bar. It is available only on iOS and macOS.
4. *navigationBarTrailing* - The item is placed in the trailing area of the navigation bar. It is available only on iOS and macOS.

![watchOS-toolbar](/public/watchOS.png)

#### ToolbarContent
We can also create a dedicated type defining our toolbar that we can reuse across the app. SwiftUI provides us the *ToolbarContent* protocol that allows us to create a struct representing a reusable toolbar. Let's take a look at how we can use it.

```swift
import SwiftUI

struct ItemsToolbar: ToolbarContent {
    let add: () -> Void
    let sort: () -> Void

    var body: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            Button("Add", action: add)
        }

        ToolbarItem(placement: .bottomBar) {
            Button("Sort", action: sort)
        }
    }
}
```

As you can see in the example above, we create *ItemsToolbar* struct that conforms to *ToolbarContent* protocol. We customize it by providing closures for adding new items and sorting. Now we can reuse it across the app whenever we need to present a list of actions for an items list.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            NavigationView {
                Text("Hello World!")
                    .toolbar {
                        ItemsToolbar {
                            // add new item here
                        } sort: {
                            // sort items
                        }

                    }
            }
        }
    }
}
```

#### ToolbarItemGroup
Sometimes we might have several toolbar items in the same placement. Creating a *ToolbarItem* for every single button in the toolbar can be very repetitive. That's why SwiftUI provides us another type called *ToolbarItemGroup*. *ToolbarItemGroup* allows us to fix toolbar items in the specific placement. Let's take a look at a very quick example.

```swift
import SwiftUI

struct ItemsToolbar: ToolbarContent {
    let add: () -> Void
    let sort: () -> Void
    let filter: () -> Void

    var body: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            Button("Add", action: add)
        }

        ToolbarItemGroup(placement: .bottomBar) {
            Button("Sort", action: sort)
            Button("Filter", action: filter)
        }
    }
}
```

#### Conclusion
Today we learned how to use the new Toolbar API to present actions in our apps in a unified way on different platforms. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
