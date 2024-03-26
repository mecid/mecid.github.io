---
title: Building async button in SwiftUI
layout: post
---

Swift Concurrency became a vital part of my development stack. I leverage the power of the new Swift Concurrency features like async/await and task groups almost everywhere. SwiftUI Button type doesn't support Swift Concurrency out of the box, but it is flexible enough to allow us to build a button type supporting Swift Concurrency.

Almost every interaction starting a task in our apps is displayed as a button. A considerable part of this task should be non-blocking for other user interface parts. Let's start with a simple example demonstrating how to start an async task when the user presses a button.

=====================================================

As you can see in the example above, we define a button that starts a task on every press. The heavyUpdate function simulates the long-running task by sleeping for some time. You can press the button as many times as you need, and it will create numerous tasks. Usually, you need to disable the button while the action is in progress.

=====================================================

Now, we define the isRunning property, which allows us to track the state of the async task. When the isRunning property changes to true, we disable the button. Let's extract the button's logic into the dedicated view.

=====================================================

As you can see in the example above, we have extracted our button's logic into a separate AsyncButton type. The SwiftUI framework's environment feature allows us to style any instance of the AsyncButton type the same way we style plain buttons.

=====================================================

The final touch we need to add to our AsyncButton type is cancelation support. We need to be able to cancel the running task. I will use the trigger value, a commonly used pattern in the SwiftUI framework, to achieve this. The idea is straightforward. You only need an equatable value to observe and react to its change.

=====================================================

As you can see, we have introduced the trigger property and used the onChange view modifier to observe it. As soon as the trigger property changes, we cancel the button's ongoing task. Let's look at how to use the trigger pattern in a simple example.

=====================================================

The simple toggling of a boolean value is enough to run the onChange view modifier and cancel the task. This approach is used widely across SwiftUI. For example, the same pattern is used in the sensory feedback and scroll view APIs.

Today, we learned how to build a custom button type that supports the Swift Concurrency feature. We were also introduced to the new trigger pattern, which is a declarative way of doing imperative things.
