---
title: Redux-like state container in SwiftUI. Connectors.
layout: post
category: Architecture
image: /public/store.png
---

During the last year, I totally understand the power of a single source of truth and a state container that holds the whole app state in a single place. I've used this approach in a couple of my apps and continue to use it in new projects.

> If you are not familiar with the concept of a single source of truth, take a look at my dedicated series of ["Redux-like state container in SwiftUI. Basics."](/2019/09/18/redux-like-state-container-in-swiftui/) posts.

I'm rewriting my ShowBot app in SwiftUI using the single state container approach. I want to talk mainly about watched episodes history screen. This is how it looks now. Let's try to build this screen.

![showbot](/public/showbot.jpg)

==========================code=======================

The main issue here is the formatting logic that lives inside the view. We can't verify it using unit tests. Another problem is SwiftUI previews. We have to provide the entire store with the whole app state to render a single screen.

We can improve the case a little bit by using derived stores that provide only the app state's needed part. But we still need to keep the formatting logic somewhere outside of the view.

> To learn more about derived stores, take a look at my ["Redux-like state container in SwiftUI. Best practices."]() post.

Let me introduce another component that lives in between the whole app store and the dedicated view. The primary responsibility of this component is the transformation of the app state to the view state. I call it Connector, and it is Redux inspired component.

==========================code=======================

As you can see, Connector is a simple protocol that defines two functions. The first one transforms the whole app state into the view state, and the second one converts view actions into app actions. Let's refactor our view by introducing view state and view actions.

==========================code=======================

We create an entirely different model for our view that holds the only needed data. The view state here is a direct mapping of the view representation and its model. The view action enum is the only action that available for this particular view. You eliminate the accidents where you call unrelated actions.

==========================code=======================

Another benefit here is the super simple view. It doesn't do anything. The view displays the data from and sends actions. You can quickly write as many SwiftUI previews as you need to cover all the different cases like loading, empty, etc.

========================Preview=======================

It's time to create the particular connector type, which we will use to bind the app state to the view state.

======================connector=======================

As you can see in the example above, WatchedHistoryConnector is a simple value type that we can quickly test using unit testing. Now, we can take a look at how we can use our connector types. Usually, I have container or flow views that connect views to the store.

======================container=======================

> To learn more about Container Views, take a look at [“Introducing Container views in SwiftUI”](/2019/07/31/introducing-container-views-in-swiftui/) post.

This week I've shared the approach that I use in my latest project. I love how it works and the possibility to improve my test coverage by introducing a simple value type. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!