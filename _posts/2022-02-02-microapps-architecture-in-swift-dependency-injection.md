---
title: Microapps architecture in Swift. Dependency Injection.
layout: post
---

We covered a lot of things related to microapps architecture in Swift during the last month. I would love to finalize the series of posts by touching another essential edge of the approach: Dependency Injection. This week we will learn how to inject the dependencies into feature modules to improve testability and control the environment.

As we told before, we should build our feature modules as completely isolated apps. That's why we call them microapps. Every microapp can have its architecture or state management approach depending on the feature complexity. You can use MVVM in one module or unidirectional flow in another module.

Feature modules shouldn't implement low-level functionality like networking or caching. Feature modules should define the dependencies needed, and the coordinator or container fulfills them. Let's take a look at some examples. 

Assume that we are working on a search feature module. We need to make an API request to search for the items matching our query. We also need to fetch recent queries to show query history. Let's start with defining the view model for our search view.

=====================================================

As you can see in the example above, we create the SearchViewModel that defines the Dependencies type. The Dependencies struct list all the low-level pieces that we need to implement the functionality of our view model. Now we can move on to fulfill the logic we need in our SearchView.

=====================================================

This approach allows us to expose only necessary low-level logic to our view model. The SearchViewModel type defines its own set of dependencies and requires them to compile.

In another case, the SearchService type might implement different search endpoint functions, and we can pass the instance inside the SearchViewModel. The downside of this approach is that the SearchViewModel type will have access to all the parts of the SearchService type, even if it doesn't need them.

Another benefit of the approach we describe in this post is the opportunity to easily mock dependencies to write unit tests and run previews in Xcode.

=====================================================

Now let's talk about where to store the whole app dependencies. Usually, we have a container that initializes and keeps all the app's services. We can store it in the AppDelegate or inside the root view of a SwiftUI app.

=====================================================

As you can see, I also added the extension with calculated property to easily extract dependencies for the search feature module. You can have this kind of property as many as you need to fulfill the requirements of all the app's feature modules.

Today we learned how to inject the low-level functionality inside feature modules without exposing information about the concrete implementation.
