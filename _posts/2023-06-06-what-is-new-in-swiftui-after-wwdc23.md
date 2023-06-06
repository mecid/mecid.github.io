---
title: What is new in SwiftUI after WWDC 23.
layout: post
category: Meta
image: /public/wwdc23.jpg
---

WWDC 23 is here, so many things have changed and been added to the SwiftUI framework. In this post, you can find the summary of the most significant SwiftUI features available in the 5th iteration of the framework.

{% include friends.html %}

#### Data flow
Swift 5.9 introduced the macros feature, which became the heart of the SwiftUI data flow. SwiftUI became Combine-free and uses the new Observation framework now. The Observation framework provides us with the *Observable* protocol that we have to use to allow SwiftUI to subscribe to changes and update views.

```swift
@Observable
final class Store {
    var products: [String] = []
    var favorites: [String] = []
    
    func fetch() async {
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        // load products
        products = [
            "Product 1",
            "Product 2"
        ]
    }
}
```

You don't need to conform to the *Observable* protocol in your code. Instead, you mark your type with the **@Observable** macro, which makes that conformance to the *Observable* protocol for you. You also don't need the *@Published* property wrapper now because SwiftUI views automatically track the changes in the available properties of any observable type.

```swift
struct ProductsView: View {
    @State private var store = Store()
    
    var body: some View {
        List(store.products, id: \.self) { product in
            Text(verbatim: product)
        }
        .task {
            if store.products.isEmpty {
                await store.fetch()
            }
        }
    }
}
```

Previously, there were a bunch of property wrappers like *State*, *StateObject*, *ObservedObject*, and *EnvironmentObject*, which you should understand when and why to use. Nowadays, state management has become much more manageable. You will only use the *State* property wrappers both for value types like strings and integers and reference types conforming to the *Observable* protocol.

```swift
struct FavoriteProductsView: View {
    let store: Store
    
    var body: some View {
        List(store.favorites, id: \.self) { product in
            Text(verbatim: product)
        }
    }
}
```

In the example above, we have a view that accepts the *Store* type. In the previous iterations of the SwiftUI framework, we should mark it with the *@ObservedObject* property wrapper to subscribe changes. We don't need it now because SwiftUI views automatically track changes in the types conforming to the *Observable* protocol.

```swift
struct EnvironmentViewExample: View {
    @Environment(Store.self) private var store
    
    var body: some View {
        Button("Fetch") {
            Task {
                await store.fetch()
            }
        }
    }
}

struct ProductsView: View {
    @State private var store = Store()
    
    var body: some View {
        List(store.products, id: \.self) { product in
            Text(verbatim: product)
        }
        .task {
            if store.products.isEmpty {
                await store.fetch()
            }
        }
        .toolbar {
            NavigationLink {
                EnvironmentViewExample()
            } label: {
                Text(verbatim: "Environment")
            }
        }
        .environment(store)
    }
}
```

You can also use the *Environment* property wrapper in pair with the *environment* view modifier to put the observable type into the SwiftUI environment. There is no need to use the *@EnvironmentObject* property wrapper or the *environmentObject* view modifier. The same *Environment* property wrapper works with the observable types now.

#### Animations
Animations always was the most vital part of the SwiftUI framework. It is effortless to animate anything in SwiftUI, but previous framework versions lack some features that we have now.  

```swift
struct AnimationExample: View {
    @State private var value = false
    
    var body: some View {
        Text(verbatim: "Hello")
            .scaleEffect(value ? 2 : 1)
            .onTapGesture {
                withAnimation {
                    value.toggle()
                } completion: {
                    print("Animation have finished")
                }
            }
    }
}
```

As you can see in the example above, we have the new version of the *withAnimation* function allowing us to provide an animation completion handler. It is a great addition, and you can build phased animations now. 

```swift
enum Phase: CaseIterable {
    case start
    case loading
    case finish
}

struct PhasedAnimationExample: View {
    @State private var value = false
    
    var body: some View {
        PhaseAnimator(Phase.allCases, trigger: value) { phase in
            switch phase {
            case .start:
                StartPhaseView()
                    .onTapGesture {
                        value.toggle()
                    }
            case .loading:
                LoadingPhaseView()
            case .finish:
                FinishPhaseView()
            }
        } animation: { phase in
            switch phase {
            case .start: .easeIn(duration: 0.3)
            case .loading: .easeInOut(duration: 0.5)
            case .finish: .easeOut(duration: 0.1)
            }
        }
    }
}
```

The SwiftUI framework introduces the new *PhaseAnimator* view that iterates over the sequence of phases, allows you to provide different animations for every phase, and updates the content whenever phase changes. There is also the *KeyframeAnimator* view allowing us to animate changes with keyframes.

#### ScrollView
ScrollView has excellent additions this year. First, we can observe content offset using the *scrollPosition* view modifier.

```swift
struct ContentView: View {
    @State private var scrollPosition: Int? = 0
    
    var body: some View {
        ScrollView {
            Button("Scroll") {
                scrollPosition = 80
            }
            
            ForEach(1..<100, id: \.self) { number in
                Text(verbatim: number.formatted())
            }
            .scrollTargetLayout()
        }
        .scrollPosition(id: $scrollPosition)
    }
}
```

As you can see, we use the *scrollPosition* view modifier to bind the content offset to a state property. Whenever the user scrolls the view, it updates the binding by setting the identity of the first visible view. You can also scroll to any view by updating the binding programmatically. But remember that you should use the *scrollTargetLayout* view modifier to tell the SwiftUI framework where to find identities to update the binding.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            ForEach(1..<100, id: \.self) { number in
                Text(verbatim: number.formatted())
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.paging)
    }
}
```

You can now change the scroll behavior by using the *scrollTargetBehavior* view modifier. It allows you to enable paging in a scroll view.

#### Search
Search-related view modifiers have a few great additions too. For example, you can programmatically focus on a search field.

```swift
struct ProductsView: View {
    @State private var store = Store()
    @State private var query = ""
    @State private var scope: Scope = .default
    
    var body: some View {
        List(store.products, id: \.self) { product in
            Text(verbatim: product)
        }
        .task {
            if store.products.isEmpty {
                await store.fetch()
            }
        }
        .searchable(text: $query, isPresented: .constant(true), prompt: "Query")
        .searchScopes($scope, activation: .onTextEntry) {
            Text(verbatim: scope.rawValue)
        }
    }
}
```

As you can see in the example above, you can use the *isPresented* parameter of the *searchable* view modifier to show/hide the search field. You can also use the *activation* parameter of the *searchScopes* view modifier to define scopes visibility logic.

#### New gestures
New *RotateGesture* and *MagnifyGesture* allow us to track the view's rotation and magnification.

```swift
struct RotateGestureView: View {
    @State private var angle = Angle(degrees: 0.0)

    var rotation: some Gesture {
        RotateGesture()
            .onChanged { value in
                angle = value.rotation
            }
    }

    var body: some View {
        Rectangle()
            .frame(width: 200, height: 200, alignment: .center)
            .rotationEffect(angle)
            .gesture(rotation)
    }
}
```

#### Small additions
We have the brand new *ContentUnavailableView* type that we can use whenever we should display an empty view. It is a small but delightful addition.

```swift
struct ProductsView: View {
    @State private var store = Store()
    
    var body: some View {
        List(store.products, id: \.self) { product in
            Text(verbatim: product)
        }
        .background {
            if store.products.isEmpty {
                ContentUnavailableView("Products list is empty", systemImage: "list.dash")
            }
        }
        .task {
            if store.products.isEmpty {
                await store.fetch()
            }
        }
    }
}
```

There are also a pair of new view modifiers allowing us to tune the spacing in the list. You can use the *listRowSpacing* and *listSectionSpacing* view modifier to set the spacing you need in the list. The *EnvironmentValues* struct includes a bunch of new properties related to the latest platform updates like *isActivityFullscreen* and *showsWidgetContainerBackground*.

```swift
#Preview {
    ContentView()
}
```

There is a new *Preview* macro allowing us to build previews easily both for UIKit and SwiftUI in a few lines of code.

#### Conclusion
There are tons of small additions all over the SwiftUI framework that we will cover during the upcoming months. So stay tuned to my blog, and don't miss anything. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
