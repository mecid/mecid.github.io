---
title: Window management in SwiftUI
layout: post
---

One of the significant additions to the current iteration of the SwiftUI framework was window management APIs. We can open a separate window using the new environment APIs and create a menu bar app using the new scene APIs. This week we will learn how to use new window management APIs in SwiftUI.

#### Multiple windows support
You can always check whenever the platform you are running supports multiple window environments using the supportsMultipleWindows environment value.

=====================================================

#### Single window
SwiftUI provides a new scene type to define a single window on macOS. 

=====================================================

As you can see in the example above, we use the new Window type to define a single window assigned to a particular identifier. Now we can use the openWindow environment value to open a window with the dedicated identifier.

=====================================================

#### Window group
Another addition to the scene API is the new overload of the WindowGroup scene type allowing us to register a window group displaying items by identifier.

=====================================================

In the example above, we register a window group for a particular identifier type. Now we can use the same environment value to open a window by providing an item identifier.

=====================================================

#### Window styling
The SwiftUI framework provides a few ways of styling windows using view modifiers. First, we can display or hide the title of the window by applying the windowStyle view modifier. It accepts hiddenTitleBar or titleBar values. This view modifier should be used on the window itself. 

Another option is to use the presentedWindowStyle view modifier on the view hierarchy, which will set the particular style for windows created by this view hierarchy.

#### Menubar window
Menu bar apps are very popular on macOS, and now we have a native way to build them in SwiftUI. There is a new scene type called MenuBarExtra. It appears in the system menu bar and presents its content in a popover-like window.

=====================================================

You can control the visibility of the menu bar icon via another overload accepting a boolean binding.

=====================================================

Menu bar apps can render their content as a menu or inside a dedicated window. You can control this behavior using the menuBarExtraStyle view modifier.

=====================================================

#### Conclusion
Today we learned how to use new APIs to manage windows in SwiftUI. The declarative approach we used to see for views is working for high-level concepts like windows now. Feel free to use it to build iOS, iPadOS, and macOS apps in a single codebase.
