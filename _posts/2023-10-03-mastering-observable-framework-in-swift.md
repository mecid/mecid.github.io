---
title: Mastering Observation framework in Swift
layout: post
---

Apple introduced the new Observation framework powered by the macro feature of the Swift language. The new Observation framework, in combination with the Swift Concurrency features, allows us to replace the Combine framework that looks deprecated by Apple. This week, we will learn how to use the Observation framework to handle data flow in our apps.

{% include friends.html %}

Using the new Observation Framework is super easy. All you need to do is to mark your class with the **@Observable** macro.

```swift
@Observable final class Store<State, Action> {
    typealias Reduce = (State, Action) -> State
    
    private(set) var state: State
    private let reduce: Reduce
    
    init(initialState state: State, reduce: @escaping Reduce) {
        self.state = state
        self.reduce = reduce
    }
    
    func send(_ action: Action) {
        state = reduce(state, action)
    }
}
```

As you can see in the example above, we use the *@Observable* macro to annotate our *Store* type. After that, we can observe any variable in the *Store* type. We have only one variable in the *Store* type that defines the store's state. Another field is a *let* constant that never changes.

```swift
withObservationTracking {
    render(store.state)
} onChange: {
    print("State changed")
}
```

To observe an instance of the *Store* type, we need to call the *withObservationTracking* function with two closures. In the first closure, we can touch all the needed properties of the observable type. The Observation framework calls the second closure only once as soon as any touched property of the observed type changes.

```swift
func startObservation() {
    withObservationTracking {
        render(store.state)
    } onChange: {
        Task { startObservation() }
    }
}
```

The Observation framework runs the *onChange* only ones, which means you should call it recursively to observe changes constantly. Another thing you should be aware of is that *onChange* closure runs before the actual change applies. That's why we postpone the *onChange* action by starting a new task.

In SwiftUI, you don't need to use the *withObservationTracking* function to observe changes. SwiftUI automatically tracks changes of any observable type inside the SwiftUI view.

```swift
struct ProductsView: View {
    let store: Store<AppState, AppAction>
    
    var body: some View {
        List(store.state.products, id: \.self) { product in
            Text(product)
        }
        .onAppear {
            store.send(.fetch)
        }
    }
}
```

As you can see in the example above, we don't use any property wrappers to observe the store. SwiftUI does it automatically. As soon as the state property of the store changes, SwiftUI updates the view. We don't need the *@ObservedObject* property wrapper to track changes in observable types, but we still need the *@StateObject* alternative to survive through the SwiftUI lifecycle.

Apple simplifies the set of property wrappers we should use with the new Observation framework. Instead of the *@StateObject* property wrapper, we can use *@State* now. *@State* property wrapper works for simple value types and any observable types now.

```swift
struct ContentView: View {
    @State private var store = Store<AppState, AppAction>(
        initialState: .init(),
        reduce: reduce
    )
    
    var body: some View {
        ProductsView(store: store)
    }
}
```

The same approach goes to the environment feature of the SwiftUI framework. There is no need for the *@EnvironmentObject* property wrapper now. You can now use the *@Environment* property wrapper and the *environment* view modifier with observable types.

```swift
struct ContentView: View {
    @State private var store = Store<AppState, AppAction>(
        initialState: .init(),
        reduce: reduce
    )
    
    var body: some View {
        ProductsView()
            .environment(store)
    }
}

struct ProductsView: View {
    @Environment(Store<AppState, AppAction>.self) var store
    
    var body: some View {
        List(store.state.products, id: \.self) { product in
            Text(product)
        }
        .onAppear {
            store.send(.fetch)
        }
    }
}
```

The last thing you may wonder is how to derive a binding from an observable type. SwiftUI introduces a *@Bindable* property wrapper for this case that works only with observable types.

```swift
@Observable final class AuthViewModel {
    var username = ""
    var password = ""
    
    var isAuthorized = false
    
    func authorize() {
        isAuthorized.toggle()
    }
}

struct AuthView: View {
    @Bindable var viewModel: AuthViewModel
    
    var body: some View {
        VStack {
            if !viewModel.isAuthorized {
                TextField("username", text: $viewModel.username)
                SecureField("password", text: $viewModel.password)
                
                Button("authorize") {
                    viewModel.authorize()
                }
            } else {
                Text("Hello, \(viewModel.username)")
            }
        }
    }
}
```

You can use the *@Bindanble* property wrapper to create bindings from the properties of any observable type easily. Sometimes, you may need to inline *@Bindable* inside the view body to create bindings.

```swift
struct InlineAuthView: View {
    @State var viewModel = AuthViewModel()
    
    var body: some View {
        @Bindable var viewModel = viewModel
        
        VStack {
            if !viewModel.isAuthorized {
                TextField("username", text: $viewModel.username)
                SecureField("password", text: $viewModel.password)
                
                Button("authorize") {
                    viewModel.authorize()
                }
            } else {
                Text("Hello, \(viewModel.username)")
            }
        }
    }
}
```

I love how the new Observation framework simplifies the data flow in SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
