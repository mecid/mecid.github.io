---
title: Mastering List in SwiftUI
layout: post
category: Mastering SwiftUI views
---

The list is the crucial view for many apps. I can't imagine an app that doesn't use a list view anywhere in the view hierarchy. During WWDC21, list view became even more powerful and brought us all the needed features of UITableView. This week, we will learn how to use the list view in SwiftUI and master its features.

#### Basics
List view is straightforward but very powerful. You can use it similarly to other SwiftUI views. To create a list view in SwiftUI, you should initiate the List struct with a view builder closure that defines the content of the list.

=====================================================

Usually, we use the list view to display an array of similar items. To achieve this behavior with SwiftUI, we should use another version of list view's initializer to map every item in the collection to its view representation.

=====================================================

As you can see in the example above, we construct the list by providing a messages array and the keypath defining the message's ID. SwiftUI uses the ID to differentiate the items and animate changes like insert, remove and reorder. Remember that you should provide a keypath to the stable ID property. For example, it might be a stored property from your database.

#### Sections
Sometimes we need to display the content of the list view in different sections. We can make it happen using the Section view in SwiftUI.

=====================================================

As you can see here, we use the ForEach view to iterate over collections of items and map them to particular views. It also needs an ID to differentiate the items in the collection.

#### Recursive
I didn't face this case too often, but we have to display recursive data structures like trees in our apps from time to time. You can easily do that with the list view in SwiftUI. There is a particular list initializer that accepts a keypath for the recursive field of your data structure. It will use the keypath to traverse and display your data recursively.

=====================================================

As you can see in the example above, we define the Tree struct with the children field, which is another array of trees. Then, we initiate the list view and provide a keypath to the recursive field.

#### Selection
List view provides you an opportunity to select items only in the edit mode. However, we usually want to mark several items to delete or move them together. Therefore, to enable the selection interface, we have to move the view to the edit mode and provide a selection binding to the list view.

=====================================================

We add the edit button in the toolbar. Edit button toggles the edit mode for the current scope and enables the editing interface. You can allow both single and multi-selection modes. It depends on the type of selection binding you provide. SwiftUI enables multi selection mode when you give a binding to the set, or it uses single selection binding when you pass the binding to the single ID item.

=====================================================

Here we provide a binding to the single item ID. That's why SwiftUI enables the single selection mode on the list.

#### Swipe actions
We can attach leading and trailing swipe actions to any items in the list view. We can quickly achieve that by using the new swipeActions view modifier.

=====================================================

In the example above, we add the opportunity to remove the items from the list using a trailing edge swipe. We can provide multiple actions on every edge. Here is another example with two swipeable actions on the leading edge with different button colors.

=====================================================

#### List styling options
SwiftUI provides a few different styles for the list view. It includes plain, sidebar, inset, grouped, inset, and insetGrouped styles. By default, SwiftUI uses insetGrouped style, but you can change it to any style type you need using the listStyle view modifier. Keep in mind that the listStyle view modifier uses an environment to propagate the selected style, and it will affect all the list views down in the view hierarchy. 

=====================================================

SwiftUI also provides us a set of view modifier which allows us to change the tint color of separators, list items or completely hide them from the list.

=====================================================

#### Conclusion
This week we learned how to use the most crucial SwiftUI view. So I'm happy to cover the list view in my blog finally.
