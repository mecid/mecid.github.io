---
title: Building async button in SwiftUI
layout: post
category: Building custom views
image: /public/swiftui.png
---

Swift Concurrency became a vital part of my development stack. I leverage the power of the new Swift Concurrency features like async/await and task groups almost everywhere. SwiftUI Button type doesn't support Swift Concurrency out of the box, but it is flexible enough to allow us to build a button type supporting Swift Concurrency.

{% include friends.html %}

Almost every interaction starting a task in our apps is displayed as a button. A considerable part of this task should be non-blocking for other user interface parts. Let's start with a simple example demonstrating how to start an async task when the user presses a button.

```swift
struct ExampleView: View {
    @State private var counter = 0
    
    var body: some View {
        VStack {
            Text(counter, format: .number)
            
            Button {
                Task {
                    await heavyUpdate()
                }
            } label: {
                Text("Increment")
            }
        }
    }
    
    private func heavyUpdate() async {
        do {
            print("update started")
            try await Task.sleep(for: .seconds(3))
            counter += 1
            print("update finished")
        } catch {
            print("update cancelled")
        }
    }
}
```

As you can see in the example above, we define a button that starts a task on every press. The *heavyUpdate* function simulates the long-running task by sleeping for some time. You can press the button as many times as you need, and it will create numerous tasks. Usually, you need to disable the button while the action is in progress.

```swift
struct ExampleView: View {
    @State private var isRunning = false
    @State private var counter = 0
    
    var body: some View {
        VStack {
            Text(counter, format: .number)
            
            Button {
                isRunning = true
                Task {
                    await heavyUpdate()
                    isRunning = false
                }
            } label: {
                Text("Increment")
            }
            .disabled(isRunning)
        }
    }
    
    private func heavyUpdate() async {
        do {
            print("update started")
            try await Task.sleep(for: .seconds(3))
            counter += 1
            print("update finished")
        } catch {
            print("update cancelled")
        }
    }
}
```

Now, we define the *isRunning* property, which allows us to track the state of the async task. When the *isRunning* property changes to true, we disable the button. Let's extract the button's logic into the dedicated view.

```swift
struct AsyncButton<Label: View>: View {
    let action: () async -> Void
    let label: Label
    
    @State private var isRunning = false
    
    init(
        action: @escaping () async -> Void,
        @ViewBuilder label: () -> Label
    ) {
        self.action = action
        self.label = label()
    }
    
    var body: some View {
        Button {
            isRunning = true
            Task {
                await action()
                isRunning = false
            }
        } label: {
            label
        }
        .disabled(isRunning)
    }
}
```

As you can see in the example above, we have extracted our button's logic into a separate *AsyncButton* type. The SwiftUI framework's environment feature allows us to style any instance of the *AsyncButton* type the same way we style plain buttons.

```swift
struct AsyncButtonExampleView: View {
    @State private var counter = 0
    
    var body: some View {
        VStack {
            Text(counter, format: .number)
            
            AsyncButton {
                do {
                    try await Task.sleep(for: .seconds(3))
                    counter += 1
                } catch {
                    // handle cancelation...
                }
            } label: {
                Text("Increment")
            }
            .controlSize(.large)
            .buttonStyle(.borderedProminent)
        }
    }
}
```

> To learn more about styling buttons in SwiftUI, take a look at my dedicated ["The many faces of button in SwiftUI"](/2021/06/30/the-many-faces-of-button-in-swiftui/) post.

The final touch we need to add to our *AsyncButton* type is cancelation support. We need to be able to cancel the running task. I will use the trigger value, a commonly used pattern in the SwiftUI framework, to achieve this. The idea is straightforward. You only need an equatable value to observe and react to its change.

```swift
struct AsyncButton<Label: View, Trigger: Equatable>: View {
    var cancellation: Trigger
    let action: () async -> Void
    let label: Label
    
    @State private var task: Task<Void, Never>?
    @State private var isRunning = false
    
    init(
        cancellation: Trigger = false,
        action: @escaping () async -> Void,
        @ViewBuilder label: () -> Label
    ) {
        self.cancellation = cancellation
        self.action = action
        self.label = label()
    }
    
    var body: some View {
        Button {
            isRunning = true
            task = Task {
                await action()
                isRunning = false
            }
        } label: {
            label
        }
        .disabled(isRunning)
        .onChange(of: cancellation) {
            task?.cancel()
        }
    }
}
```

As you can see, we have introduced the *trigger* property and used the *onChange* view modifier to observe it. As soon as the trigger property changes, we cancel the button's ongoing task. Let's look at how to use the trigger pattern in a simple example.

```swift
struct AsyncButtonExampleView: View {
    @State private var counter = 0
    @State private var trigger = false
    
    var body: some View {
        VStack {
            Text(counter, format: .number)
            
            AsyncButton(cancellation: trigger) {
                do {
                    try await Task.sleep(for: .seconds(3))
                    counter += 1
                } catch {
                    
                }
            } label: {
                Text("Increment")
            }
            .controlSize(.large)
            .buttonStyle(.borderedProminent)
            
            Button {
                trigger.toggle()
            } label: {
                Text("Cancel")
            }
        }
    }
}
```

The simple toggling of a boolean value is enough to run the *onChange* view modifier and cancel the task. This approach is used widely across SwiftUI. For example, the same pattern is used in the sensory feedback and scroll view APIs.

Today, we learned how to build a custom button type that supports the Swift Concurrency feature. We were also introduced to the new trigger pattern, which is a declarative way of doing imperative things. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

