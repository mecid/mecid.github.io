---
title: Mastering StoreKit 2
layout: post
---

StoreKit provides us with an opportunity to make revenue from our apps. It allows us to set up the purchase flow for in-app purchases and subscriptions. StoreKit 2 introduces a modern Swift-based API to build type-safe in-app purchases. This week we will start the series of posts about StoreKit 2.

First, we must configure our project by adding in-app purchases in the projects' "Signing & Capabilities" tab. Next, we should create a StoreKit configuration file to test in-app purchases without a network connection with the App Store. Go to File -> New -> File and choose "StoreKit Configuration File". 

You can create a local-only configuration file and populate it with test subscriptions and in-app purchases. Another option is to fetch the list of subscriptions and in-app purchases from the App Store Connect by enabling the "Sync this file with an app in App Store Connect" checkbox.

The final step is to run your app with the predefined StoreKit configuration file. You need to edit the scheme of your project, and in the options tab of the run section, choose your StoreKit configuration file. Now you have a fully-configured project allowing us to test in-app purchases in the Xcode.

Let's start building our paywall feature by introducing the Store type to handle all the logic related to in-app purchases.

=====================================================

As you can see in the example above, we define the Store type fetching and storing the list of products we will display on the paywall screen. The StoreKit 2 framework provides the Product type encapsulating all the logic around an in-app purchase. The Product type has a static function called products that we can use to fetch the list of products by providing a collection of identifiers.

=====================================================

We use our Store type to fetch and display the list of available in-app purchases. An instance of the Product type contains all information we need to show, like the title, description, and price of an in-app purchase. 

The Product type also has the purchase function that we can use to start an in-app purchase flow for a particular product. It returns an instance of the PurchaseResult enum defining three cases: success, pending, and userCancelled.

Whenever the purchase result is in the success state, it provides an associated value of type Transaction defining the successful transaction. StoreKit wraps the transaction in the VerificationResult type allowing us to validate that transaction is correctly signed and comes from the App Store.

As soon as you retrieve the transaction, you should unlock the feature user purchased and call the finish function on the particular transaction. Remember, you should always finish the transaction only after unlocking the purchased feature.

The purchase becomes pending whenever the ask to buy is enabled. In this case, the transaction arrives later, only after approval by the parent. You should observe the Transaction.updates stream to handle this kind of transaction. We must start monitoring this stream as soon as the app launches to never miss a transaction.
