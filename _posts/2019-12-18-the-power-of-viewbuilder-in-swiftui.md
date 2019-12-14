---
title: The power of @ViewBuilder in SwiftUI
layout: post
image: /public/viewbuilder.png
---

Last week we started a series of posts about developing interactive components using *SwiftUI*, where we talked about building the bottom sheet. We need to understand the power of *@ViewBuilder* before moving to the next post about building another interactive view. That's why this week, we will talk about *@ViewBuilder* and its benefits while developing custom views.

#### @FunctionBuilder
*@ViewBuilder* is one of the possible function builders. The function builders feature of *Swift* is described in [Swift Evolution Proposal](https://github.com/apple/swift-evolution/blob/9992cf3c11c2d5e0ea20bee98657d93902d5b174/proposals/XXXX-function-builders.md). The main goal of function builders is providing *DSL* like syntax. Let's take a look at a very quick example of *@ViewBuilder* usage.

===============================================

We need to go down one layer to understand how *@ViewBuilder* works.

===============================================

Here is the declaration of *HStack* view, as you can see the content closure inside the init method marked with *@ViewBuilder*. It means that expression inside that closure needs to be handled by *@ViewBuilder*. The swift compiler will try to find the static buildBlock method declared in *@ViewBuilder* struct that has two views as parameters. Let's take a look at *@ViewBuilder* struct declaration to find that method.

===============================================

As you can see, *@ViewBuilder* has a static *buildBlock* method that accepts two views, combine them and return *TupleView*. It also has other declarations of *buildBlock* method, which takes from one to ten child view, and all of them combine child view into a *TupleView*. That's why *@ViewBuilder* can accept only ten views inside the closure.

> To learn how to avoid ten views limitation, take a look at my ["View Composition in SwiftUI"](/2019/10/30/view-composition-in-swiftui/) post.

#### TupleView
===============================================

*TupleView* is a view created from a swift tuple of view values. *TupleView* doesn't have any logic inside. It just holds the views. *TupleView* completely transparent and behaves like its parent view. It means when you put it inside the *HStack*, *TupleView* places the views from the tuple in a horizontal direction.

#### Using @ViewBuilder
Now we know all the needed things to build our own custom view container, which uses *@ViewBuilder*. Assume that our app needs a notification view. The notification view should have a consistent design and appear from the top of the screen, but content can be various. It is a perfect use case for *@ViewBuilder*. Let's see how we can utilize it.

===============================================

As you can see, we use *@ViewBuilder* to mark our content closure. It gives us the opportunity to use *NotificationView* in the same way as *VStack* or *HStack*. Here is the example of using *NotificationView*.

===============================================

We also used the ability to build custom views via *@ViewBuilder* during the last post, where we made the *BottomSheetView* in *SwiftUI*. 

#### Conclusion
This week we talked about the benefits of function builders and used *@ViewBuilder* as a concrete example. *@ViewBuilder* allows us to build super reusable *SwiftUI* views by separating its presentation logic and content. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 