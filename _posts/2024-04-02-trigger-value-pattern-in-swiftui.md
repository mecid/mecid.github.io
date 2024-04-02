---
title: Trigger value pattern in SwiftUI
layout: post
---

The recent version of the SwiftUI framework introduces a trigger value pattern across its APIs. Trigger value allows us to attach a view modifier that runs its action whenever the trigger value changes. You can find this pattern while using sensory feedback or launching keyframe animation in SwiftUI. This week, we will learn how to build custom view modifiers using trigger value patterns.

As I said before, the trigger value pattern became very popular in the recent version of the SwiftUI framework. Let's look at the example showing the usage of the sensoryFeedback view modifier in SwiftUI.

=====================================================

In the example above, we attach the sensoryFeedback view modifier to run haptic feedback on the device. Haptic feedback doesn't run on view appearance. It only runs whenever the trigger value changes. Another example is the scrollIndicatorsFlash view modifier.

=====================================================

The scrollIndicatorsFlash view modifier allows us to flash the indicator of the scroll view whenever the messages property changes. The idea behind this pattern is pretty simple. We observe an equatable value and run the action whenever it changes.

Let's implement a similar view modifier playing a sound whenever the trigger value changes.

=====================================================

As you can see in the example above, we implement a view modifier type that defines a sound property and trigger value. In the body of the view modifier, we observe the trigger value and play the sound using an instance of the AVAudioPlayer type on every change of the trigger value.

=====================================================

Now, we have the playSound view modifier, allowing us to reuse the functionality across our codebase. In other words, the trigger value pattern will enable us to build reusable view modifiers to run data-driven actions in SwiftUI.

Let's look at another example of the trigger value pattern used to implement the AsyncButton type.

=====================================================

In the example above, we use the trigger value pattern to provide cancelation functionality in the AsyncButton type.

=====================================================

As you can see, we use the trigger value pattern to cancel the disabled button's ongoing task and make it active again.

Today, we learned how to build custom APIs by introducing the trigger value pattern in our codebase. The trigger value pattern is widespread across the SwiftUI framework, and I'm sure more APIs will use this pattern soon.
