---
title: What is new in SwiftUI after WWDC 24
layout: post
image: /public/wwdc24.webp
category: Meta
---

WWDC 24 is here, and we have a lot to cover. Every year, SwiftUI matures by introducing more features to catch up with UIKit. This year is no exception. Let's dive into the new features that the SwiftUI framework introduces.

#### View collections
SwiftUI introduced the new overloads for Group and ForEach views, allowing us to create custom containers like List or TabView.

=====================================================

As you can see in the example above, we use the Group view with the new initializer, which allows us to access the views of the content view passed via @ViewBuilder closure. SwiftUI introduces new Subview and SubviewsCollection types, providing proxy access to real views.

#### New tab bar experience
Using the new Tab type, the new customizable tab bar experience with fluid transition into a sidebar is available in SwiftUI.

=====================================================

As you can see in the example above, we use the new Tab type to define our tabs. We also use the tabViewStyle view modifier on the instance of the TabSection to group and move the particular section of tabs to the sidebar.

#### Hero animations
SwiftUI introduced matchedTransitionSource and navigationTransition, which we can use in pair on any instance of the NavigationLink type.

=====================================================

It enables us to create smooth transitions between views using the same identifier and namespace while navigating from one view to another inside NavigationStack.

#### Scroll position
The new ScrollPosition type, in pair with the scrollPosition view modifier, allows us to read the precise position of a ScrollView instance. We can also use it to programmatically scroll to the particular point of the scrolling content.

=====================================================

#### Entry macro
The new Entry macro allows us to quickly introduce environment values, focused values, etc, without boilerplate. Let's look at how we define environment values before the Entry macro.

=====================================================

Now, we can minimize our code by using the Entry macro.

=====================================================

#### Others
The next iteration of the SwiftUI framework includes many new APIs, such as window pushing, text selection observation in the TextField and TextEditor views, search focus monitoring, custom text rendering, new MeshGradient type, and much more that I can't cover in a single post.
