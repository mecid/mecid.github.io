---
title: iOS development trends in review
layout: post
category: Meta
image: /public/debug-preview.jpeg
---

2022 has come to an end, and it is a perfect time for retrospective analysis. Today I want to review trends in iOS development over the past year that I notice while building my own apps or consulting others.

{% include friends.html %}

#### SwiftUI
SwiftUI has a lot of criticism about its functionality compared to more mature UIKit. During the last few years, Apple invested a lot of time into the SwiftUI framework and improved it in a few ways. SwiftUI became a robust framework allowing you to build great apps declaratively.  

This year we finally got the new data-driven Navigation API allowing us to move the user through the app using a value-oriented approach. The new Navigation API is also simplifying deep linking and state restoration.

Another great addition was the *Layout* protocol which provided us with a very low-level API to build custom layouts. Now we can create reusable layout types with the same API as *VStack* and *HStack*.

Nowadays, many system-oriented features of the apps require using the SwiftUI framework. The SwiftUI framework is the only way to build home screen, lock screen, live activity widgets, etc.

* [Tracking geometry changes in SwiftUI](/2024/08/13/tracking-geometry-changes-in-swiftui/)
* [Mastering MapKit in SwiftUI. Basics.](/2023/11/28/mastering-mapkit-in-swiftui-basics/)
* [Building custom layout in SwiftUI. Basics.](/2022/11/16/building-custom-layout-in-swiftui-basics/)
* [Mastering NavigationStack in SwiftUI. Navigator Pattern.](/2022/06/15/mastering-navigationstack-in-swiftui-navigator-pattern/)
* [Mastering NavigationStack in SwiftUI. Deep Linking.](/2022/06/21/mastering-navigationstack-in-swiftui-deep-linking/)

#### Unidirectional flow
Unidirectional flow means that all the data in the application follows the same pattern, making the logic of your app more predictable and easier to understand. The unidirectional flow pattern works great in conjunction with the idea of a single source of truth.

The single state for the whole app makes it easier to debug and inspect. The single source of truth eliminates tons of bugs produced by creating multiple duplicates of the same piece of state across the app.

* [Unidirectional flow in Swift](/2023/07/11/unidirectional-flow-in-swift/)
* [Redux-like state container in SwiftUI. Basics.](/2019/09/18/redux-like-state-container-in-swiftui/)
* [Functional core Imperative shell in Swift. Unidirectional Flow.](/2022/03/16/functional-core-imperative-shell-in-swift-unidirectional-flow/)

#### Modularization
Mobile apps become bigger and bigger by providing a complete set of business features. Teams become larger, and the number of developers working on the product increases. This process often creates problems like increased compile time, merge conflicts, and violates the separation of concerns in the codebase.

Fortunately, modularization solves all of these problems. Swift Package Manager became the heart of this approach. It allows us to extract every feature into a separate Swift module and reduce the compile time of your project. Teams can produce a module per app feature and run it as a separate app to improve the value delivery.

* [Microapps architecture in Swift. SPM basics.](/2022/01/12/microapps-architecture-in-swift-spm-basics/)
* [Microapps architecture in Swift. Feature modules.](/2022/01/19/microapps-architecture-in-swift-feature-modules/)
* [Microapps architecture in Swift. Dependency Injection.](/2022/02/02/microapps-architecture-in-swift-dependency-injection/)
* [Microapps architecture in Swift. Resources and localization.](/2022/01/26/microapps-architecture-in-swift-resources-and-localization/)

#### Testing
As I said before, apps have become the primary source of income for many types of businesses around the world. We as developers, should take care of the stability and consistency of the apps. It is tough to achieve a good level of crash and bug-free sessions of the app without good test coverage.

UI tests become essential for validating crucial user flows in our apps. UI tests are very slow, and we should try to cover as much logic as we can with unit tests and use UI tests only for important user flows like checkout or add to cart.

Usually, UI tests require much more effort than unit tests, and keeping the test codebase as clean as possible is vital. I saw that many developers treated differently to testing code. But it is important to design testing code the same way we do with the feature code. A few patterns allow us to keep UI tests in excellent shape. 

* [UI Testing in Swift with XCTest framework](/2021/03/18/ui-testing-in-swift-with-xctest-framework/)
* [UI Testing using Page Object pattern in Swift](/2021/03/24/ui-testing-using-page-object-pattern-in-swift/)
* [Introducing Swift Testing. Basics.](/2024/10/22/introducing-swift-testing-basics/)
* [Introducing Swift Testing. Lifecycle.](/2024/10/29/introducing-swift-testing-lifecycle/)
* [Introducing Swift Testing. Traits.](/2024/11/05/introducing-swift-testing-traits/)
* [Introducing Swift Testing. Parameterized Tests.](/2024/11/12/introducing-swift-testing-parameterized-tests/)

#### Accessibility
Last but not least is accessibility. Accessibility isn't a feature or a "nice to have." It's a necessity. First, I would like to mention that Apple has done a great job with the Accessibility framework. Most of the things are handled by the system without our action.

Usually, your app is accessible by default. You only need to test your app with Voice Over and try to improve the user experience by grouping or hiding particular elements on the screen. You can also provide some shortcuts for the actions in your app for accessible users.

Don't hesitate to add a few lines of code to improve your app's user experience significantly.

* [Make your app accessible for everyone](/2018/07/09/make-your-app-accessible-for-everyone/)
* [Accessibility in SwiftUI](/2019/09/10/accessibility-in-swiftui/)
* [Dynamic Type in SwiftUI](/2019/10/09/dynamic-type-in-swiftui/)
* [Custom accessibility content in SwiftUI](/2021/10/06/custom-accessibility-content-in-swiftui/)


#### Conclusion
I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Merry Christmas and Happy New Year!
