---
title: Mastering ContentUnavailableView in SwiftUI
layout: post
---

SwiftUI introduces the new ContentUnavailableView type, allowing us to display empty, error states or any other state where content is unavailable in our apps. This week, we will learn how to use the ContentUnavailableView to guide users through empty states of your app.

Let's start with the very first example showing the basic usage of the ContentUnavailableView view.

=====================================================

As you can see in the example above, we define the ContentUnavailableView as overlay for the list of products. Whenever the product list is empty we display the ContentUnavailableView with title and image. Another variant of the ContentUnavailableView view allows us also define the description text of the current state.

=====================================================

The ContentUnavailableView allows us also display action buttons below the description. That's why there is another variant of the ContentUnavailableView initializer allowing us to define every piece of the view using ViewBuilder closures.

=====================================================

As you can see in the example above, we use a set of closures to define label, description and actions. This initializer allows us to fully customize the look and feel of an instance of the ContentUnavailableView type.

SwiftUI provides us a ready-to-use predefined instance of the ContentUnavailableView type that we can use in search screens.

=====================================================

Whenever you have a search screen displaying search results, you can use the search instance of the ContentUnavailableView type. It is localized by the framework and traverse the view hierarchy to find a search bar and extract its text to display inside the view.

Remember that you should place the searchable view modifier under the overlay if you want to extract the text from the search bar otherwise it doesn't personalize the message.

=====================================================

You can also provide query into the description manually by using the search function of the ContentUnavailableView type with a single parameter.

Today, we learned how to use the ContentUnavailableView type in SwiftUI to display empty states in a user-friendly manner.
