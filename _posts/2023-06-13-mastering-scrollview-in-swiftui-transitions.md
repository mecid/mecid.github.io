---
title: Mastering ScrollView in SwiftUI. Transitions
layout: post
---

The fifth iteration of the SwiftUI framework brings a lot of new APIs related to ScrollView, making it much more powerful than before. This week will begin the series of posts about new abilities of the ScrollView in SwiftUI, and we will start with scroll transitions.

The brand new scrollTransition view modifier allows us to observe the transition of the view while the user scrolls content. It will enable us to understand whenever a view is in the viewport of the ScrollView and allows us to apply visual effects like scaling, rotating, etc. Let's take a look at a quick example.

=====================================================

As you can see in the example above, we use the new scrollTransition view modifier to accept a closure with two parameters. The first is the view without any effects, and the second is an instance of the ScrollTransitionPhase type.

The ScrollTransitionPhase type defines a state of a view transition in the viewport of an instance of the ScrollView. The ScrollTransitionPhase type is an enum with three cases: topLeading, bottomTrailing, and identity. The ScrollTransitionPhase enum provides the isIdentity property allowing us to check whenever the view finished its transition.

Usually, you display the view in the identity phase without any effects. The SwiftUI framework animates all the changes you apply during the transition. In our example, I use the opacity view modifier to change the view's alpha during the transition.

The ScrollTransitionPhase enum provides another property called value. It ranges from -1 to 1 and defines the numeric phase of transition where -1 means topLeading and 1 means bottomTrailing. We can use it to scale our visual effects while a view transition from topLeading to bottomTrailing.

=====================================================

As you can see in the example above, we use the value property on the ScrollTransitionPhase type to scale our view while transitioning from one phase to another.

The scrollTransition view modifier allows us to tune the animation we want to use while interpolating the transition.

=====================================================

Here we use the bouncy animation to interpolate between transition phases. You can use a few options to configure the transition: .interactive, animated with the particular animation you provide, and identity to disable animation.

=====================================================

We also can use different configurations for topLeading and bottomTrailing transitions. I use the identity configuration to disable transition effects in this direction.

Today we learned how to use the new scrollTransition view modifier to animate viewport transitions in the ScrollView.



