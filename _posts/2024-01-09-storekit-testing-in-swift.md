---
title: StoreKit testing in Swift
layout: post
---

The second iteration of the StoreKit framework was the most significant change in my apps during the last few years. The recent version of the StoreKit framework has fully adopted Swift language features like async/await. This week, we will talk about the StoreKitTest framework, which is not a part of StoreKit 2 but is tightly coupled with it.

The StoreKitTest framework allows us to write tests for in-app product purchasing, refunding, and restoring features. You can cover almost every aspect of the in-app purchase with tests using the StoreKitTest framework. Before starting, you should create a StoreKit Configuration File.

=====================================================

The StoreKitTest framework provides us with the SKTestSession type. Using an instance of the SKTestSession type, we can purchase in-app products, manage transactions, refund and expire subscriptions, etc.

Let's start by creating a test case for our StoreKit-related features. I usually have a type called SettingsStore, which defines user configuration and handles in-app purchases. We will cover the in-app purchase management part of the SettingsStore with tests by using the StoreKitTest framework.

=====================================================

As you can see in the example above, we initialize an instance of the SKTestSession type. Then, we call the clearTransactions function to remove all the transactions we may have stored from the previous launches. We also turn off dialogs to automate the purchase confirmation flow easily.

Now, we can use our SettingsStore type to purchase products and process subscription status. The SKTestSession type also allows us to buy a product that simulates the purchase outside the app. For example, it might be a purchased product with family sharing enabled.

=====================================================

As you can see in the example above, we use the buyProduct function on an instance of the SKTestSession type to simulate a purchase. We can also use the expireSubscription function of the SKTestSession type to expire ongoing subscriptions and verify how our app processes this data.

=====================================================

The SKTestSession type also allows us to simulate product refunds using the refundTransaction function. Another exciting option is to test how the app reacts to transaction updates. 

For example, you can use the askToBuyEnabled property to enable the ask-to-buy feature and then use the approveAskToBuyTransaction or declineAskToBuyTransaction functions to approve or decline purchases. In this case, the transaction should change from pending to successful.
