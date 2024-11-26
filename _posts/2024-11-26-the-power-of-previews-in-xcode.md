---
title: The power of previews in Xcode
layout: post
category: Mastering SwiftUI views
image: /public/debug-preview.jpeg
---

Previews in Xcode become more powerful every year. Previews in Xcode are not about SwiftUI; you can use them even with UIKit. This week, we will talk about enhancing *Previewable* and *PreviewModifier* types, allowing us to build reusable preview environments.

{% include friends.html %}

Let’s take a look at the situation where you have a SwiftUI view with a binding. To run a preview for views with binding requires additional boilerplate where you have to wrap the view with another view defining a state for your binding and then pass it to the previewing view.

```swift
struct SleepDetailsView: View {
    let snapshot: SleepSnapshot
    @Binding var goal: TimeInterval
    
    var body: some View {
        // ...
    }
}

struct SleepDetailsPreview: View {
    @State var goal: TimeInterval = 8 * 3600
    
    var body: some View {
        SleepDetailsView(snapshot: SleepSnapshot(), goal: $goal)
    }
}

#Preview {
    SleepDetailsPreview()
        .preferredColorScheme(.dark)
}
```

As you can see in the example above, we create the *SleepDetailsPreview* type only for the purpose of previewing the *SleepDetailsView* in Xcode. It simply clutters our codebase with yet another view type. Fortunately, Xcode 16 introduced the *Previewable* macro type.

The *Previewable* macro type allows us to inline the state definition into the *Preview* macro. It is specially designed to use inside of the *Preview* macro and use outside of *Preview* makes a compiler error.

```swift
#Preview {
    @Previewable @State var goal: TimeInterval = 8 * 3600
    
    SleepDetailsView(snapshot: SleepSnapshot(), goal: $goal)
        .preferredColorScheme(.dark)
}
```

As you can see, we inline the *State* property into the *Preview* macro by adding the *Previewable* macro. It allows us to significantly reduce the code we need to run a preview in Xcode.

Another great addition to Xcode Previews is the *PreviewModifier* protocol. The *PreviewModifier* type allows us to create reusable preview environments that we can share across multiple previews. 

For example, we can create the *MockDataPreviewModifier* type that populates the Swift Data container with the mock data to display while using it in previews.

```swift
struct MockDataPreviewProvider: PreviewProvider {
    static func makeSharedContext() throws -> ModelContainer {
        let container = try ModelContainer(
            for: Item.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )
     
        populateContainer(container)
        
        return container
    }
    
    static func populateContainer(_ container: ModelContainer) {
        // ...
    }
    
    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}
```

Here we define the *MockDataPreviewProvider* type conforming to the *PreviewProvider* protocol. The *PreviewProvider*s protocol has two requirements the *makeSharedContext* and *body* functions. 

In the *makeSharedContext* you can create and prepare anything you might need in your preview. For example, it might be a mock data-backed instance of the *ModelContainer* or any store type specific to your app logic.

In the *body* function, you have access to the previously created context. The body function is the place where you can apply your context. In our example, we use the *modelContainer* view modifier to set the model container for the view hierarchy. For the instance of a custom store type, you can use the *environment* view modifier to pass it in.

```swift
#Preview(traits: MockDataPreviewProvider()) {
    ItemsView()
}
```

As you can see, we have to pass an instance of the *MockDataPreviewProvider* type to the *Preview* macro’s *trait* parameter to apply it.

The great thing about the *PreviewModifier* is that the Xcode Preview system caches instances returned from the *makeSharedContext* function. Which makes significant performance boost whenever you preview multiple instances with the same trait. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
