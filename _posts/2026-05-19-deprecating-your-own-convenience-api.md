---
title: Deprecating your own convenience API
layout: post
---

Almost after every major update of iOS, we got new APIs that we use on the most recent platform but can’t use on the previous one. Usually, I solve this kind of thing by introducing my own convenience code that runs new APIs on the available versions and my custom implementation on old platform versions.

Usually, my apps support two of the most recent platform versions, and it is easy to maintain. But now, we have iOS 26 and iOS 18, and they differ a lot in many small details. For example, in iOS 26, toolbar items are displayed as SF Symbols or any other similar image, but on iOS 18, we used to display text-based buttons. It is easy to solve, but it creates code that will be dead in a year when the new iOS version arrives.

So, you should keep in mind every custom type or function you build to cover functionality on older platforms, because you might need to delete them in a year as soon as you bump the minimal platform version. Or, you can make the compiler remind you about that code. This week, we will talk about a way to make the compiler help us in identifying dead code in our codebase.

=======================================================

Let’s take a look at a simple example above. We set up two toolbar items. We want to achieve a symbol-only look and feel on iOS 26 and a text-only look and feel on iOS 18. For these particular cases, I’ve introduced the ToolbarLabelStyle type.

=======================================================

As you can see in the example above, we easily solve this by introducing ToolbarLabelStyle type. It checks the availability of the platform and applies the correct styling for our labels. The code looks and feels very natural, but it will become dead code when I bump the target version to iOS 26. How to find this type of code in my codebase? It might be in so many places where I use similar solutions.

=======================================================

Fortunately, we can use availability annotations. We annotate our toolbar property with the availability annotation, which deprecates and obsoletes the usage of the toolbar property. You are curious about what it means? 

As soon as you bump the target of your app to iOS 26, all the usage of the toolbar property will be marked as warnings by the compiler with the message we put in the annotation. Whenever you bump the target to iOS 27 (in the future), the compiler will produce an error because this code is already obsolete.

=======================================================

You can be more aggressive and instead of deprecating your code, make it obsolete for iOS 26. In this case, you will get compiler errors instead of warnings. Sometimes, it might be blocking you from some work, so I highly encourage you to deprecate first, then obsolete. But you should always take care of compiler warnings to not accumulate a technical debt.
