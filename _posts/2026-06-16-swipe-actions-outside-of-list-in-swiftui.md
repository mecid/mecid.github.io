---
title: Swipe actions outside of List in SwiftUI 
layout: post
---

Swipe actions were a primary reason for using List in SwiftUI. As you may recall, I’ve mentioned several times that a scroll view paired with lazy stacks is the preferred approach in most scenarios, except when swipe actions are required. 

The swipeActionsContainer view modifier allows us to use swipe actions without List. This week, we will learn how to use swipeActionsContainer view modifier to attach swipe actions inside scroll view, lazy stacks, and even custom layouts.
 =======================================================

As you can see in the example above, you can attach the swipeActions view modifier to the items of a List view to configure swipe actions. The swipeActions view modifier has been working only inside a List container. Fortunately, things have changed and now we can use the swipe actions with any container like a scroll view, lazy stacks, or even our custom layouts.

=======================================================

Here is another example where we attach the swipeActions view modifier to the items of the scroll view. All you need to make it work outside of List is enabling swipeActions by modifying the view using the swipeActionsContainer view modifier. 

The swipeActionsContainer view modifier is crucial and works like a swipe actions enabler. It also controls a few things. For example, it enables only one row’s swipe actions to be revealed at a time. It tracks the scrolling events and dismisses any open actions. It also dismisses actions while tapping outside the active row.

List doesn’t need swipeActionsContainer view modifier because it does that job automatically, but anything else than list needs the swipeActionsContainer view modifier attached.

=======================================================

Here is another example where we attach the swipeActions to the custom layout. I’m sure there is no reason to use flow layout with swipe actions, but you should know that you can use it with any custom layout as soon as you modify it with the swipeActionsContainer view modifier.

This is a small but very important improvement because it removes one more reason to reach for List when we don’t actually need it. We can keep full control over layout, styling, and performance while still providing native swipe interactions where they make sense.
