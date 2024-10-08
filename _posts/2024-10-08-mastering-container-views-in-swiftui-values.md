---
title: Mastering container views in SwiftUI. Values.
layout: post
---

In the series' final post about container views in SwiftUI, I will discuss container values and how SwiftUI allows us to propagate data through the container view logic. This week, we will learn how to declaratively define and pass container values.

Container values are similar to environment values, allowing us to pass data inside through the container view instead of the environment and access it later inside the container view.

=====================================================

As you can see in the example above, we decompose the first section from the content view and place it outside to make it prominent. In this case, we hard-code the section by its index, which must always be the first section.

We can make it more dynamic using container values. What if we could mark a particular section featured, and the container view would find that section and place it at the top.

====================================================

As you can see, we use the @Entry macro to define a container value with the default value. All we need to do is to create an extension for the ContainerValues type and specify a property with the @Entry macro. Now, we can use it to mark any section or view as featured.

=====================================================

We use the containerValue view modifier to set the container value using the keypath. We can also write an extension to simplify the usage.

=====================================================

We know how to define a container value with a default state and change it using the containerValue view modifier. The next step is to read the value from the container view and decide on view recomposition.

=====================================================

Both the Subview and SectionConfiguration types provide the containerValues property, which allows us to read any defined container value.

=====================================================

As you can see in the example above, we use the containerValues property on instances of the SectionConfiguration type to filter featured and non-featured sections.

We use the isFeatured container value to recompose the view hierarchy as needed. Container values propagate through the view hierarchy similar to environment values. Any view inside the featured section will have the isFeatured value of true.

Today, we learned how to use container values to propagate data through container view and recompose view hierarchies using that information.
