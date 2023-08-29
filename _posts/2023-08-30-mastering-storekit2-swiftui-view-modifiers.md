---
title: Mastering StoreKit 2. SwiftUI view modifiers.
layout: post
category: StoreKit
---

We talked a lot about StoreKit 2 in this series of posts. This week, we will finalize the series by covering the set of view modifiers StoreKit 2 provides us to use in SwiftUI views.

{% include friends.html %}

As I said before, StoreKit views handle loading and purchasing in-app purchases completely. But sometimes, you must be aware of the current step in the flow to react. For example, you might need to dismiss the paywall whenever the user finishes the purchase or show a loading indicator while waiting for the App Store server result.

```swift
struct PaywallView: View {
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        SubscriptionStoreView(groupID: "598392E1")
            .onInAppPurchaseStart { product in
                print(product.displayName)
            }
            .onInAppPurchaseCompletion { product, result in
                dismiss()
            }
    }
}
```

StoreKit 2 provides us with *onInAppPurchaseStart* and *onInAppPurchaseCompletion* view modifiers to handle the start and completion of any purchase in the view hierarchy using StoreKit views.
Both view modifiers accept a closure to perform and have product and purchase result as parameters.

Another view modifier that simplifies our job is the *subscriptionStatusTask* modifier. It allows us to observe the status of a particular subscription group constantly. It updates you whenever the status of the subscription changes.

```swift
struct ContentView: View {
    @State private var paywallShown = false
    @State private var isPro = false
    
    var body: some View {
        RootView()
            .sheet(isPresented: $paywallShown) {
                PaywallView()
            }
            .subscriptionStatusTask(for: "598392E1") { taskState in
                if let value = taskState.value {
                    isPro = !value
                        .filter { $0.state != .revoked && $0.state != .expired }
                        .isEmpty
                } else {
                    isPro = false
                }
            }
    }
}
```

As you can see in the example above, we use the *subscriptionStatusTask* view modifier to observe the status of the particular subscription group. As you may have noticed, we provide a closure handling the subscription status changes, and it takes an array of the subscription statuses as the parameter. It uses an array instead of a single value because the user might purchase a subscription but also has access to the subscription by family sharing.

The *subscriptionStatusTask* view modifier works only with subscriptions. StoreKit 2 provides the *currentEntitlementTask* view modifier for consumable and non-consumable in-app purchases. It works similarly to the *subscriptionStatusTask* view modifier but instead gives you an optional transaction of the particular product.

```swift
struct ContentView: View {
    @State private var paywallShown = false
    @State private var isPro = false
    
    var body: some View {
        RootView()
            .sheet(isPresented: $paywallShown) {
                PaywallView()
            }
            .currentEntitlementTask(for: "1232") { taskState in
                if let verification = taskState.transaction,
                   let transaction = try? verification.payloadValue {
                    isPro = transaction.revocationDate == nil
                } else {
                    isPro = false
                }
            }
    }
}
```

As you can see, the *currentEntitlementTask* view modifier allows us to provide a closure accepting an optional transaction wrapped into the *VerificationResult* type. It is optional because there might be no transaction for the particular product. Whenever the user purchases or revokes the product, StoreKit 2 runs the attached closure.

Another view modifier StoreKit 2 provides us is the *storeProductsTask* modifier. It allows us to load the list of products and observe them. It runs the provided closure whenever any product in the collection changes.

```swift
struct ContentView: View {
    @State private var paywallShown = false
    
    var body: some View {
        RootView()
            .sheet(isPresented: $paywallShown) {
                PaywallView()
            }
            .storeProductTask(for: "12763") { taskState in
                print(taskState.product?.displayName)
            }
    }
}
```

Today, we learned how to query StoreKit 2 from SwiftUI views using brand-new view modifiers.
