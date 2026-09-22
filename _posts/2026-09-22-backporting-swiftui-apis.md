---
title: Backporting SwiftUI APIs
layout: post
---

Usually, we don’t see many new APIs in intermediate releases like 27.1, but this time is different. Apple introduced several new iPhone Duo-related APIs in the 27.1 release. This week, we will learn how to use all of these new APIs without bumping your project's target version.

{% include friends.html %}

Let’s start with the new *toolbarVerticalBehavior* view modifier. It allows us to disable the vertical toolbar and keep toolbar items in the standard horizontal top or bottom bars. It might be useful in some cases while adopting the iPhone Duo layout.

Let’s start by preparing infrastructure for backporting the *toolbarVerticalBehavior* view modifier. 

```swift
import SwiftUI

public struct Backport<Content> {
    let content: Content
}

extension View {
    public var backport: Backport<Self> {
        Backport(content: self)
    }
}
```

As you can see in the example above, we introduced the *Backport* type with a generic constraint. We also add an extension to SwiftUI’s *View* protocol to get the instance of the *Backport* type with a wrapped view. This trick allows us to namespace the APIs we will backport.

```swift
extension Backport where Content: View {
    public enum ToolbarVerticalBehavior {
        case automatic
        case disabled
    }

    @ContentBuilder public func toolbarVerticalBehavior(_ behaviour: ToolbarVerticalBehavior) -> some View {
        if #available(anyAppleOS 27.1, *) {
            switch behaviour {
            case .automatic:
                content.toolbarVerticalBehavior(.automatic)
            case .disabled:
                content.toolbarVerticalBehavior(.disabled)
            }
        } else {
            content
        }
    }
}
```

Here we introduced the *toolbarVerticalBehavior* function on the *Backport* type that handles availability checks and calls the new APIs if available or returns a plain view when it is not available. 

In this particular case, we don’t need to do anything on previous versions of the platform because the *toolbarVerticalBehavior* view modifier makes sense only in iPhone Duo. But in any other cases, you can provide your own implementation when the target condition fails.

```swift
struct ExampleView: View {
    var body: some View {
        Text("Hello")
            .toolbar {
                ToolbarItem {
                    Button("Action", systemImage: "checkmark") {
                        // do some action here
                    }
                }
            }
            .backport.toolbarVerticalBehavior(.disabled)
    }
}
```

As you can see in the example above, we use the .backport namespace before using the dedicated *toolbarVerticalBehavior* extension. What I don’t like about this approach is the need for namespacing our backported APIs. It might be useful as it collapses all backports into a single type and allows you to easily find usages of backported APIs in the future to remove them when your app target allows.

```swift
@available(iOS, introduced: 26.0, deprecated: 27.1, obsoleted: 28.0, message: "Get rid of ported version")
public enum PortedToolbarVerticalBehavior {
    case automatic
    case disabled
}

extension View {
    @available(iOS, introduced: 26.0, deprecated: 27.1, obsoleted: 28.0, message: "Get rid of ported version")
    @ViewBuilder func portedToolbarVerticalBehavior(_ behavior: PortedToolbarVerticalBehavior) -> some View {
        if #available(anyAppleOS 27.1, *) {
            switch behavior {
            case .automatic:
                toolbarVerticalBehavior(.automatic)
            case .disabled:
                toolbarVerticalBehavior(.disabled)
            }
        } else {
            self
        }
    }
}

```

Instead, let’s use the availability annotation that will make a compiler warning or error when the app is ready for using the new APIs. Here, we use a ported prefix instead of a dedicated namespace type in pair with an availability annotation, allowing us to be notified and easily find all the obsoleted functions in the future.

Backporting new SwiftUI APIs allows us to adopt the latest platform features without immediately raising the deployment target of the entire app. Depending on the API, a backport can simply hide an availability check or provide a custom implementation that mimics the new behavior on older versions.  I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
