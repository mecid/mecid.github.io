---
title: Mastering NavigationStack in SwiftUI. Navigator Pattern.
layout: post
category: Mastering SwiftUI views
---

SwiftUI is the declarative data-driven framework allowing us to build complex user interfaces by defining the data rendering on the screen. Navigation was the main pain point of the framework from the very first day. Fortunately, things have changed since WWDC 22, and SwiftUI provides the new data-driven Navigation API. This week we will learn how to use the new Navigation API to build complex user flows.

#### Basics
First, I must mention that the old NavigationView is deprecated, and we should use the new NavigationStack instead. Let's take a look at a quick example.

=====================================================

In the example above, we define a simple master-detail flow. We place NavigationStack at the root of our view hierarchy. Next, we define a list of messages where every message provides a link to the details screen of the particular message. As you can see, we still use the old NavigationLink type here, and it works great for this use case.

The NavigationLink type adds new data-driven capabilities. A brand new initializer allows us to create a link bound to some value. Here is the previous example refactored using the new data-driven Navigation API.

=====================================================

We use the new value-based navigation links to route the user through the app. Look at how we associate every item on the list with a particular value. Next, we define a destination view for a specific value using the navigationDestination view modifier. In the current example, we have only one type of destination, but you can have as many as you need by applying multiple navigationDestination view modifiers.

=====================================================

Remember that we also have another version of the value-based NavigationLink initializer, allowing us to provide a custom label for the link. We use this version of the initializer to give the headline font for category links.

#### Placing rules
You should be careful about placing the navigationDestination view modifier in the view hierarchy. There are three rules for placing the navigationDestination view modifier:

1. The top-level navigationDestination view modifier will always override the lowest one for the same type.
2. The navigationDestination view modifier should be inside the NavigationStack.
3. Don't place navigationDestination view modifier on the child of lazy container like List, ScrollView, LazyVStack, etc.

#### Navigator pattern
I love to keep my feature's navigation flow in a single place. That's why I usually implement the Navigator pattern allowing me to handle my navigation in a type-safe way. It is effortless to implement the Navigator pattern with the new data-driven Navigation API in SwiftUI. First, we should create an enum type defining all our app/feature/module routes.

=====================================================

Next, we should use Route enum with the new value-based navigation links.

=====================================================

Finally, we can place the single navigationDestination view modifier and handle all the cases of our Route type.

=====================================================

#### Conclusion
I'm thrilled to use the new data-driven Navigation API in SwiftUI. If you follow my blog, you might know that I have been waiting for it long. I will continue to cover the new data-driven Navigation API in detail, and next week we will talk about deep linking.
