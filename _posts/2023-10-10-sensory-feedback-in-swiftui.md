---
title: Sensory feedback in SwiftUI
layout: post
---

SwiftUI introduced the new sensoryFeedback view modifier, allowing us to play haptic feedback on all Apple platforms. This week, we will learn how to use the sensoryFeedback modifier to give haptic feedback on different actions in our apps.

All we need to play haptic feedback in a SwiftUI view is to attach the sensoryFeedback view modifier with two parameters. The first defines a feedback style, and the second is a trigger value.

=====================================================

As you can see in the example above, we use the sensoryFeedback view modifier with success style. We also define the results property of the store as a trigger. It means SwiftUI will play the success-styled haptic feedback whenever store results change.

SwiftUI provides a bunch of predefined feedback styles like success, warning, error, selection, increase, decrease, start, stop, alignment, levelChange, impact, etc.

=====================================================

As you can see, the impact style allows us to tune the weight and intensity of the feedback. Remember that it is always better to use the predefined style and customize the haptic feedback for super custom cases.

=====================================================

Another variant of the sensoryFeedback view modifier allows us to choose a particular feedback style depending on the trigger value. Here, we place success feedback whenever our store contains results and play error feedback whenever results are empty.

=====================================================

SwiftUI also provides an option to define a condition on trigger value, deciding whenever it should play a predefined feedback style.

This week, we learned how to use the new sensoryFeedback view modifier to play haptic feedback in our SwiftUI views. Remember that you shouldn't overwhelm the user with haptics. Use them to improve accessibility and significant task results.
