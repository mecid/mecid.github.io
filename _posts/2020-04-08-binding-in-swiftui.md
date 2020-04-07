---
title: Binding in SwiftUI
layout: post
image: /public/swiftui.png
---

Binding is one of the several property wrappers that *SwiftUI* presents us to control data flow in the app. Binding provides us a reference like access to a value type. This week we will understand how and when to use binding. We will also learn how to avoid common mistakes while using binding in *SwiftUI*.

#### Basics
Binding is a property wrapper type that can read and write a value owned by a source of truth. We have several possible types of sources of truth in *SwiftUI*. It can be *EnvironmentObject*, *ObservedObject*, and *State*. All these property wrappers provide a projection value, which is binding. Let's take a look at the quick example.

```swift
import SwiftUI

struct ExampleView: View {
    @State private var text = "Hello World."

    var body: some View {
        TextField("type something...", text: $text)
    }
}
```

Here we have a state that is a source of truth. We also have a *TextField*, which requires a binding for a text value. We use a dollar sign to access the projected value of the state property wrapper, which is a binding to the value of property wrapper.

> To learn more about property wrappers in SwiftUI, take a look at the dedicated post.

#### Common mistakes
Let's take a look at a more complicated example. We will build an app that shows the list of users and allows us to edit user data.

```swift
import SwiftUI
import Combine

struct Person: Identifiable {
    let id: UUID
    var name: String
    var age: Int
}

final class PersonStore: ObservableObject {
    @Published var persons: [Person] = [
        .init(id: .init(), name: "Majid", age: 27),
        .init(id: .init(), name: "John", age: 31),
        .init(id: .init(), name: "Fred", age: 25)
    ]
}

struct PersonsView : View {
    @ObservedObject var store: PersonStore

    var body: some View {
        NavigationView {
            List {
                ForEach(store.persons.indexed(), id: \.1.id) { index, person in
                    NavigationLink(destination: EditingView(person: self.$store.persons[index])) {
                        VStack(alignment: .leading) {
                            Text(person.name)
                                .font(.headline)
                            Text("Age: \(person.age)")
                                .font(.subheadline)
                                .foregroundColor(.secondary)
                        }
                    }
                }
            }.navigationBarTitle(Text("Persons"))
        }
    }
}
```

Please keep in mind that the value of binding must be a value type. It means it has to be enum or struct. I see that people sometimes use classes to describe a state or entry inside *EnvironmentObject* or *ObservedObject* and notice that binding is not working. Apple's documentation on binding says: if *Value* is not value semantic, the updating behavior for any views that make use of the resulting *Binding* is unspecified.

```swift
struct EditingView: View {
    @Environment(\.presentationMode) var presentation
    @Binding var person: Person

    var body: some View {
        Form {
            Section(header: Text("Personal information")) {
                TextField("type something...", text: $person.name)
                Stepper(value: $person.age) {
                    Text("Age: \(person.age)")
                }
            }

            Section {
                Button("Save") {
                    self.presentation.wrappedValue.dismiss()
                }
            }
        }.navigationBarTitle(Text(person.name))
    }
}
```

As you can see in the example above, we use binding to pass a writable reference to a person struct. As soon as we modify the user instance inside the *EditingView* *SwiftUI* updates *PersonsView* to respect the changes.

```swift
extension Sequence {
    func indexed() -> Array<(offset: Int, element: Element)> {
        return Array(enumerated())
    }
}
```

You can see that I use the indexed function to generate an array of tuples that provides both the element and its index. It allows me to read the item to render it in place and access the binding using the item's index.

#### Computed Binding
Usually, we access binding using a projected value of a source of truth. In this section, we will talk about another way of creating a binding. Binding is a two-way connection between the data and a view that access it. SwiftUI provides a way to construct a binding using getter and setter closures. In this case, we are responsible for calculating all the values inside these closures. It is hard to imagine where we can use it, but it plays very well with Redux-like state containers.

```swift
typealias Reducer<State, Action> = (inout State, Action) -> Void

final class Store<State, Action>: ObservableObject {
    @Published private(set) var state: State
    private let reducer: Reducer<State, Action>

    init(initialState: State, reducer: Reducer<State, Action>) {
        self.state = initialState
        self.reducer = reducer
    }

    func send(_ action: Action) {
        reducer(&state, action)
    }
}
```

Here we have a store concept that holds the entire state of the app. All changes to the state come from the unidirectional flow. Reducer is the single place that can mutate the state of the app. By using computed binding, we can provide read-only access to the state and writing capabilities using a reducer.

```swift
extension Store {
    func binding<Value>(
        for keyPath: KeyPath<State, Value>,
        transform: @escaping (Value) -> Action
    ) -> Binding<Value> {
        Binding<Value>(
            get: { self.state[keyPath: keyPath] },
            set: { self.send(transform($0)) }
        )
    }
}
```

As you can see, we generate a computed binding that reads a part of the state and emits an action through reducer to modify the state when needed.

> To learn more about implementing a Redux-like state container in SwiftUI, take a look at my dedicate series of posts.

#### Constant binding
Another way to create binding is a constant function. This function allows us to create a binding that provides value but ignores any mutations on it. In other words, it generates an immutable binding for provided value.

```swift
import SwiftUI

struct ExampleView: View {
    var body: some View {
        TextField("type something...", text: .constant("Hello!"))
    }
}
```

#### Conclusion
Today we learned another great tool to control data flow in *SwiftUI*. Binding can be a tool which is more complicated than others, but I believe we cover all the needed thing for efficient usage of bindings in *SwiftUI*. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
