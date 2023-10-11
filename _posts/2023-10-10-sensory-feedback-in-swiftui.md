---
title: Sensory feedback in SwiftUI
layout: post
category: Interactions
image: /public/haptic.png
---

SwiftUI introduced the new *sensoryFeedback* view modifier, allowing us to play haptic feedback on all Apple platforms. This week, we will learn how to use the *sensoryFeedback* modifier to give haptic feedback on different actions in our apps.

{% include friends.html %}

All we need to play haptic feedback in a SwiftUI view is to attach the *sensoryFeedback* view modifier with two parameters. The first defines a feedback style, and the second is a trigger value.

```swift
struct ContentView: View {
    @State private var store = Store()
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.self) { result in
                Text(result)
            }
            .searchable(text: $store.query)
            .sensoryFeedback(.success, trigger: store.results)
        }
    }
}
```

As you can see in the example above, we use the *sensoryFeedback* view modifier with *success* style. We also define the *results* property of the store as a trigger. It means SwiftUI will play the success-styled haptic feedback whenever store results change.

SwiftUI provides a bunch of predefined feedback styles like *success*, *warning*, *error*, *selection*, *increase*, *decrease*, *start*, *stop*, *alignment*, *levelChange*, *impact*, etc.

```swift
struct ContentView: View {
    @State private var store = Store()
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.self) { result in
                Text(result)
            }
            .searchable(text: $store.query)
            .sensoryFeedback(
                .impact(weight: .heavy, intensity: 0.9),
                trigger: store.results
            )
        }
    }
}
```

As you can see, the *impact* style allows us to tune the *weight* and *intensity* of the feedback. Remember that it is always better to use the predefined style and customize the haptic feedback for super custom cases.

```swift
struct ContentView: View {
    @State private var store = Store()
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.self) { result in
                Text(result)
            }
            .searchable(text: $store.query)
            .sensoryFeedback(trigger: store.results) { oldValue, newValue in
                return newValue.isEmpty ? .error : .success
            }
        }
    }
}
```

Another variant of the *sensoryFeedback* view modifier allows us to choose a particular feedback style depending on the trigger value. Here, we place success feedback whenever our store contains results and play error feedback whenever results are empty.

```swift
struct ContentView: View {
    @State private var store = Store()
    
    var body: some View {
        NavigationStack {
            List(store.results, id: \.self) { result in
                Text(result)
            }
            .searchable(text: $store.query)
            .sensoryFeedback(.success, trigger: store.results) { oldValue, newValue in
                return newValue.isEmpty
            }
        }
    }
}
```

SwiftUI also provides an option to define a condition on trigger value, deciding whenever it should play a predefined feedback style.

This week, we learned how to use the new *sensoryFeedback* view modifier to play haptic feedback in our SwiftUI views. Remember that you shouldn't overwhelm the user with haptics. Use them to improve accessibility and significant task results.
