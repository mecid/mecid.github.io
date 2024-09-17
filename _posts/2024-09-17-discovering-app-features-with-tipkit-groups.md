---
title: Discovering app features with TipKit. Groups.
layout: post
---

A year ago, Apple released the TipKit framework, which has a bizarre title. TipKit became a framework, making app features much easier for users to discover. This week, we will talk about an enhancement that Apple introduced to improve tip-appearing logic called TipGroups.

Maintaining order becomes difficult whenever you have more than one tip on a screen because TipKit tries to display everything at once. Let's take a look at the example of three tips displayed on a screen.

=====================================================

As you can see, we use the popoverTip view modifier three times to display the collection of tips on the screen. This way, we can't control timing, and all the tips appear as soon as possible. The SwiftUI framework introduced the TipGroup type, allowing us to group a set of tips and display them in order one by one.

=====================================================

As you can see, we define a state property of type TipGroup. The TipGroup allows us to set the priority and provide a collection of tips. In our case, we use ordered priority, which means the tip group will follow the order of the array we provide, and the add tip will only appear if the user invalidates the welcome tip.

Another priority choice that the TipKit framework provides us is the firstAvailable option. In this case, the TipKit framework doesn't keep the order inside the group and displays the first available tip.

=====================================================

Remember that you should use the currentTip on an instance of the TipGroup type to access the active tip. The currentTip property automatically calculates the available tip. You can still invalidate tips programmatically whenever a user discovers the feature, and it will update the currentTip property automatically.

=====================================================

In the previous examples, we have attached the tip group to the same view. What if we share a tip group across different views and display tips by keeping the order we define in the group? It might be a little tricky, but it is still possible.

=====================================================

As you can see in the example above, we verify that the current tip has the expected identifier. Otherwise, we ignore it by providing nil value.

Today, we learned how to improve the app's discoverability without disturbing the user with a bunch of tips. By using the TipGroups type, we can provide tips step by step in a calm manner.
