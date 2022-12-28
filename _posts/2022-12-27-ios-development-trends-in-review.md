---
title: iOS development trends in review
layout: post
---

2022 has come to an end, and it is a perfect time for retrospective analysis. Today I want to review trends in iOS development over the past year that I notice while building my own apps or consulting others.

SwiftUI
SwiftUI has a lot of criticism about its functionality compared to more mature UIKit. During the last few years, Apple invested a lot of time into the SwiftUI framework and improved it in a few ways. SwiftUI became a robust framework allowing you to build great apps declaratively.  

This year we finally got the new data-driven Navigation API allowing us to move the user through the app using a value-oriented approach. The new Navigation API is also simplifying deep linking and state restoration.

Another great addition was the Layout protocol which provided us with a very low-level API to build custom layouts. Now we can create reusable layout types with the same API as VStack and HStack.

Nowadays, many system-oriented features of the apps require using the SwiftUI framework. The SwiftUI framework is the only way to build home screen, lock screen, live activity widgets, etc.

Unidirectional flow
Unidirectional flow means that all the data in the application follows the same pattern, making the logic of your app more predictable and easier to understand. The unidirectional flow pattern works great in conjunction with the idea of a single source of truth.

The single state for the whole app makes it easier to debug and inspect. The single source of truth eliminates tons of bugs produced by creating multiple duplicates of the same piece of state across the app.
Modularization
Mobile apps become bigger and bigger by providing a complete set of business features. Teams become larger, and the number of developers working on the product increases. This process often creates problems like increased compile time, merge conflicts, and violates the separation of concerns in the codebase.

Fortunately, modularization solves all of these problems. Swift Package Manager became the heart of this approach. It allows us to extract every feature into a separate Swift module and reduce the compile time of your project. Teams can produce a module per app feature and run it as a separate app to improve the value delivery.
Testing
As I said before, apps have become the primary source of income for many types of businesses around the world. We as developers, should take care of the stability and consistency of the apps. It is tough to achieve a good level of crash and bug-free sessions of the app without good test coverage.

UI tests become essential for validating crucial user flows in our apps. UI tests are very slow, and we should try to cover as much logic as we can with unit tests and use UI tests only for important user flows like checkout or add to cart.

Usually, UI tests require many more lines of code than unit tests, and keeping the test codebase as clean as possible is vital. I saw that many developers treated differently to testing code. But it is important to design testing code as we do with the feature code. A few patterns allow us to keep UI tests in excellent shape. 
Accessibility
Last but not least is accessibility. Accessibility isn't a feature or a "nice to have." It's a necessity. First, I would like to mention that Apple has done a great job with the Accessibility framework. Most of the things are handled by the system without our action.

Usually, your app is accessible by default. You only need to test your app with Voice Over and try to improve the user experience by grouping or hiding particular elements on the screen. You can also provide some shortcuts for the actions in your app for accessible users.

Don't hesitate to add a few lines of code to improve your app's user experience significantly.

