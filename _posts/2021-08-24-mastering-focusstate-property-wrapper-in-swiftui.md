---
title: Mastering FocusState property wrapper in SwiftUI
layout: post
---

SwiftUI became very powerful during the last WWDC. We gained many new features, and one of them was a brand new FocusState property wrapper. FocusState property wrapper allows us to read and write the current focus position in the view hierarchy. This week we will learn how to manage focus in SwiftUI apps.

{% include friends.html %}

SwiftUI provides a new FocusState property wrapper that allows us to focus on a particular view or check if that view is already focused. The usage is effortless. Let's see how we can use it.

=====================================================

As you can see in the example above, we need to define a boolean variable using the FocusState property wrapper. We also have to bind its value to the focus state of a particular view using the focused view modifier. SwiftUI sets the boolean value of the view to true as soon as the user focuses on it. It also changes it to false as soon as the view loses focus.

You can define as many FocusState variables as needed to cover your focus management logic. SwiftUI takes care of them by keeping in sync the focused view with its binding.

=====================================================

In the example above, we have two variables bound to email and password text fields. SwiftUI can manage them together and keep them in sync with the user interface. Remember that you can change the value to false programmatically to hide the keyboard or set the value to true to move the focus to a particular view.

=====================================================

Defining many FocusState properties in complex view hierarchies can become cumbersome. Fortunately, FocusState works not only with boolean values but also with any Hashable type. It means that we can model the focused state using an enum type conforming to Hashable protocol. Let's take a look at the example.

=====================================================

Here we have the Field enum that conforms to Hashable and defines all the focusable views we manage. Now, we can use this enum to bind the focus state of different views to various enum cases.

=====================================================

As you can see, we use another version of the focused view modifier to bind a view to a concrete case of the Field enum. SwiftUI updates the value of the FocusState property whenever the user focuses on any of the bound views. Remember that we should make our FocusState property optional to use combined with Hashable enum because there might be no focused view at the moment.

Today we learned how to use the FocusState property wrapper to manage focus in our views. Remember that FocusState allows us both to read and change the focused view programmatically. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

