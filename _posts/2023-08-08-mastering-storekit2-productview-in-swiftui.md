---
title: Mastering StoreKit 2. ProductView and StoreView.
layout: post
---

We started a series of posts about StoreKit 2 last week. We learned the basics of StoreKit 2 and how easily we can monetize our apps. This week, we will continue the topic by learning about StoreKit views in SwiftUI. The StoreKit 2 introduces SwiftUI views, allowing us to quickly build paywalls or digital product shop screens.

StoreKit 2 provides the ProductView type allowing us to display and handle in-app purchases for a single Product easily. All you need to do is to place an instance of the ProductView with the particular product id in the view hierarchy.

=====================================================

As you can see in the example above, we can place an instance of the ProductView in a single line of code, but this line hides the complexity of loading and purchasing products from the App Store. StoreKit provides us with a set of styles to display a product in different appearances. We can use the productViewStyle view modifier to attach a particular style. StoreKit uses the automatic style by default, but we can use large, regular, or compact styles.

=====================================================

You can also enable the presentation of the promoted image if uploaded to the App Store servers. The ProductView's initializer has another parameter called prefersPromotionalIcon. You can use it to display the promoted icon.

=====================================================

You might need to draw a custom SwiftUI view instead of promoted image for a product, and it is possible by using another overload of the ProductView initializer.

=====================================================

As you can see in the example above, you can handle different loading states of promotional image and replace it with your own SwiftUI view if needed.

Another option is to build a completely custom product presentation by introducing a custom type of style. In this case, we must create a type conforming to the ProductViewStyle protocol.

=====================================================

As you can see in the example above, we define the CustomProductStyle type conforming to the ProductViewStyle protocol. The ProductViewStyle protocol has the only requirement, which is the makeBody function. The makeBody function has the parameter of type ProductViewStyleConfiguration, providing us with all the needed information about the product, loading state, and purchase functionality. 

We can use different product styles to build the layout we need. Let's make a paywall screen displaying subscription options among the lifetime in-app purchases.

=====================================================

As you can see, we use different style options to display the list of available in-app purchases just in a few lines of code. And the UI we build supports purchasing and observes refunds automatically. But StoreKit views in SwiftUI don't stop here. It provides us with the StoreView view displaying the collection of products in different layouts depending on the available space.

=====================================================

The StoreView displays the layout for displaying a collection of products. For example, on tvOS StoreView uses a grid to show the products. You can still customize the look and feel of the products displayed in the StoreView by applying the productViewStyle view modifier to the StoreView.

=====================================================

The StoreView provides a set of buttons like restore transactions or paywall cancellation buttons out of the box. You can manipulate the visibility of a particular button by using the storeButton view modifier.

Today we learned about SwiftUI views that StoreKit 2 provides us to build paywalls in our apps.
