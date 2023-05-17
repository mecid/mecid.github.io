---
title: API availability in Swift
layout: post
---

WWDC is coming pretty soon, and we are going to use a bunch of new APIs. But how to use new APIs available only for the latest version of iOS? This week we will learn about availability conditions in Swift.

Swift provides #available attributes allowing us to check the availability of the function. Here is a quick example.

=====================================================

As you can see in the example above, we use the #available attribute to check if the current platform supports the APIs from iOS 16. We also can use the #available attribute to check if the platform is unavailable.

=====================================================

We learned how to use the #available attribute to check the platform version. But how the API developer should define functions and types to provide the availability information?

=====================================================

Swift allows us to use the @available attribute to provide information about the platform that function or type needs to run. We can separate different platforms by using commas.

=====================================================

We also can define a function as deprecated or obsoleted by using the @available attribute. In the example above, we define a function as introduced in iOS 14, deprecated in iOS 15, and obsoleted in iOS 16.
Swift compiler generates the warning whenever you build the project with iOS 15 as the target. When you try to build the project with the target version iOS 16, it will generate a compiler error. This is the main difference between deprecated and obsoleted parameters.

=====================================================

You can also use the Swift compiler's message parameter to show an error or warning. The introduced parameter defines the minimal platform version needed to run the particular function.

=====================================================

Another variant of the @available attribute allows us to mark the function or type as renamed. In this case, Xcode allows the user to press the warning message and show the fix button that replaces the old name with the new one.

Today we learned not only how to use the new APIs in the legacy codebase but also how to be a good citizen on the platform and define the functions and types with the platform availability information, which is crucial when you work on a framework or library.
