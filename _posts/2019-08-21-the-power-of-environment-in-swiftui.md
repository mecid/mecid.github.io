---
title: The power of Environment in SwiftUI
layout: post
category: Data Flow
---

*Environment* is one of the unique features of SwiftUI which we didn't have before in *UIKit*. Today I would like to show you all the benefits of using *Environment* in your apps.

{% include friends.html %}

#### Environment
Let's start with describing the idea of *Environment*. We already discussed it previously in ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/), but I want to start with basics. When you create and start your very first *View* in SwiftUI, the framework generates *Environment* for it. SwiftUI creates it automatically, and we don't need to do something.

SwiftUI uses *Environment* to pass system-wide settings like *ContentSizeCategory, LayoutDirection, ColorScheme, etc*. *Environment* also contains app-specific stuff like *UndoManager* and *NSManagedObjectContext*. Full list of the passed values you can find in [*EnvironmentValues* struct documentation](https://developer.apple.com/documentation/swiftui/environmentvalues). Let's take a look at an example where we access *Environment* values.

```swift
struct ButtonsView: View {
    @Environment(\.sizeCategory) var sizeCategory

    var body: some View {
        Group {
            if sizeCategory == .accessibilityExtraExtraExtraLarge {
                VStack {
                    buttons
                }
            } else {
                HStack {
                    buttons
                }
            }
        }
    }
}
```

By using @*Environment property wrapper*, we can read and subscribe on changes for the selected value. Here we have *ButtonsView* that reads Dynamic Type value from *Environment* and put buttons in *VStack* or *HStack* depending on the size category value. User can change Dynamic Type value in the system settings, and as soon as it happens, SwiftUI will recreate *ButtonsView* to respect the changes.

Now let's see how we can modify *Environment* values. In SwiftUI we don't have separation like *Controllers* or *Views*. Everything is a *View*, and because of that, we can easily modify *Environment* for an entire view hierarchy of the app by adding *environment modifier* to the root view.

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        window = UIWindow(windowScene: scene as! UIWindowScene)
        window?.rootViewController = UIHostingController(
            rootView: RootView()
                .environment(\.multilineTextAlignment, .center)
                .environment(\.lineLimit, nil)
                .environment(\.lineSpacing, 8)
        )

        window?.makeKeyAndVisible()
    }
```

In the example above, we made all the text in the app center-aligned with line spacing 8pt without any line limit.

#### Environment inheritance
Every view inside SwiftUI inherits *Environment* from its parent view by default. But remember that you can override any values you want while creating the child view by attaching *Environment* modifier.

```swift
struct RootView {
    var body: some View {
        PlayerView()
            .environment(\.layoutDirection, .leftToRight)
    }
}

struct PlayerView: View {
    var body: some View {
        HStack {
            Button("previous") {

            }
            Button("play") {

            }
            Button("next") {

            }
        }
    }
}
```

We don't need to change the order of the buttons on right-to-left locales like Arabic. That's why *RootView* set layout direction to *leftToRight* on *PlayerView*. It's important to understand that this modification will apply only to *PlayerView* and all its child views.

#### View specific Environment values
We already covered how SwiftUI pass system-wide settings via *Environment*, but this is not the end. SwiftUI uses *Environment* to inject visible view specific values like *isEnabled, editMode, presentationMode, horizontalSizeClass, verticalSizeClass, etc.*

```swift
struct ModalView: View {
    @Environment(\.presentationMode) var presentation

    var body: some View {
        Button("dismiss") {
            self.presentation.value.dismiss()
        }
    }
}
```

Here we use view-specific environment values to dismiss presented modal view.

#### Custom Environment keys
Now we know that SwiftUI provides us plenty of system-wide and view-specific values via the environment. However, I have to mention that we can create a custom environment key and push any value we want into the environment. Let's take a look at how we can insert custom values into the environment.

```swift
import SwiftUI

struct ItemsPerPageKey: EnvironmentKey {
    static var defaultValue: Int = 10
}

extension EnvironmentValues {
    var itemsPerPage: Int {
        get { self[ItemsPerPageKey.self] }
        set { self[ItemsPerPageKey.self] = newValue }
    }
}

struct RelatedProductsView: View {
    @Environment(\.itemsPerPage) var count

    let products: [Product]

    var body: some View {
        ForEach(products[..<count], id: \.id) { product in
            Text(product.title)
        }
    }
}
```

In the example above, we create a custom key that represents a count of items that can be presented on a screen. We also implement a view that uses that environment value. You can easily pass that value to any view you want by using the *environment modifier*.

#### Dependency Injection via Environment
Another great use-case for *Environment* is Dependency Injection. Every view has its copy of the parent's *Environment*, and we can use it to add all *ObservableObjects* related to the current view.

```swift
struct CalendarView : View {
    @EnvironmentObject var store: CalendarStore

    var body: some View {
        NavigationView {
            List {
                ForEach(self.store.sleeps) { sleep in
                    NavigationLink(
                        destination: SleepDetailsView()
                            .environmentObject(SleepStore(sleep: sleep))
                    ) {
                        CalendarRow(sleep: sleep)
                    }
                }
            }
        }.navigationBarTitle("calendar")
    }
}
```

In the example above, we use *environmentObject* modifier to pass an instance of *SleepStore* object. *SleepStore* should conform to *ObservableObject* protocol, which is used by SwiftUI to recreate the view during data changes.

> To learn more about *ObservableObject* check another post ["Making real-world app with SwiftUI"](/2019/06/05/swiftui-making-real-world-app/).

The significant benefit of using *Environment* and not passing *ObservableObject* via the *init* method of the view is the internal SwiftUI storage. SwiftUI stores *Environment* in the special framework memory outside the view. It gives an implicit access to view-specific *Environment* for all child views.

#### Conclusion
As much as I use SwiftUI, I enjoy the concept of *Environment*. As you can see, it allows us to configure our app's view hierarchy and make nice Dependency Injection out of the box. I hope you will love *Environment* feature of SwiftUI. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 