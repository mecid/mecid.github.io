---
title: Trigger value pattern in SwiftUI
layout: post
image: /public/swiftui.png
---

The recent version of the SwiftUI framework introduces a trigger value pattern across its APIs. Trigger value allows us to attach a view modifier that runs its action whenever the trigger value changes. You can find this pattern while using sensory feedback or launching keyframe animation in SwiftUI. This week, we will learn how to build custom view modifiers using trigger value pattern.

{% include friends.html %}

As I said before, the trigger value pattern became very popular in the recent version of the SwiftUI framework. Let's look at the example showing the usage of the *sensoryFeedback* view modifier in SwiftUI.

```swift
struct TriggerValueExample: View {
    let messages: [String]
    
    var body: some View {
        List(messages, id: \.self) { message in
            Text(verbatim: message)
        }
        .sensoryFeedback(.impact, trigger: messages)
    }
}
```

In the example above, we attach the *sensoryFeedback* view modifier to run haptic feedback on the device. Haptic feedback doesn't run on view appearance. It only runs whenever the trigger value changes. Another example is the *scrollIndicatorsFlash* view modifier.

```swift
struct TriggerValueExample: View {
    let messages: [String]
    
    var body: some View {
        List(messages, id: \.self) { message in
            Text(verbatim: message)
        }
        .scrollIndicatorsFlash(trigger: messages)
    }
}
```

The *scrollIndicatorsFlash* view modifier allows us to flash the indicator of the scroll view whenever the messages property changes. The idea behind this pattern is pretty simple. We observe an equatable value and run the action whenever it changes.

Let's implement a similar view modifier playing a sound whenever the trigger value changes.

```swift
struct PlaySoundViewModifier<Trigger: Equatable>: ViewModifier {
    let sound: URL
    let trigger: Trigger

    func body(content: Content) -> some View {
        content
            .onChange(of: trigger) {
                if let player = try? AVAudioPlayer(contentsOf: sound) {
                    player.play()
                }
            }
    }
}

extension View {
    func playSound(_ sound: URL, trigger: some Equatable) -> some View {
        self.modifier(PlaySoundViewModifier(sound: sound, trigger: trigger))
    }
}
```

As you can see in the example above, we implement a view modifier type that defines a sound property and trigger value. In the body of the view modifier, we observe the trigger value and play the sound using an instance of the *AVAudioPlayer* type on every change of the trigger value.

```swift
struct SoundFeedbackExample: View {
    let messages: [String]
    
    var body: some View {
        List(messages, id: \.self) { message in
            Text(verbatim: message)
        }
        .playSound(
            Bundle.main.url(forResource: "sound", withExtension: "wav")!,
            trigger: messages
        )
    }
}
```

Now, we have the *playSound* view modifier, allowing us to reuse the functionality across our codebase. In other words, the trigger value pattern will enable us to build reusable view modifiers to run data-driven actions in SwiftUI.

Let's look at another example of the trigger value pattern used to implement the *AsyncButton* type.

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

In the example above, we use the trigger value pattern to provide cancelation functionality in the *AsyncButton* type.

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

As you can see, we use the trigger value pattern to cancel the disabled button's ongoing task and make it active again.

Today, we learned how to build custom APIs by introducing the trigger value pattern in our codebase. The trigger value pattern is widespread across the SwiftUI framework, and I'm sure more APIs will use this pattern soon. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

