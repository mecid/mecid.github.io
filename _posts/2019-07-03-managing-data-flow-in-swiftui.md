---
title: Managing Data Flow in SwiftUI
layout: post
---

Last week we talked about [Animations and Transitions in SwiftUI](/2019/06/26/animations-in-swiftui/). But it's time to touch the crucial aspect of every app, and it is data. All the apps have data to present or mutate. Data plays a vital role in apps using *SwiftUI*. Every view in *SwiftUI* is just a function of some state, where the state is our data.

#### Fetching data from local/remote storage
Today we will build a small app which uses core *SwiftUI* concepts like Binding and *BindableObject*. Assume that you work on the app, which has two primary responsibilities:

1. Fetch and show the list of employees from local or remote storage
2. Edit personal information about selected employee

Let's start with describing our model layer.

```swift
import SwiftUI
import Combine

struct Person: Identifiable {
    let id: UUID
    var name: String
    var age: Int
}

final class PersonStore: BindableObject {
    let didChange = PassthroughSubject<Void, Never>()

    var persons: [Person] = [] {
        didSet {
            DispatchQueue.main.async {
                self.didChange.send(())
            }
        }
    }

    func fetch() {
        // Fetch your data from real storage here
        persons = [
            .init(id: .init(), name: "Majid", age: 27),
            .init(id: .init(), name: "John", age: 31),
            .init(id: .init(), name: "Fred", age: 25)
        ]
    }
}
```

Here we have simple *Person* struct which conforms *Identifiable* protocol. The single requirement of *Identifiable* is *Hashable* *id* field. We implement it by defining *id* as *UUID*. We also can use *Int* instead of *UUID*.

Next, we can implement *PersonStore* class, which is providing data for our view. *PersonStore* type conforms to *BindableObject* it will allow *SwiftUI* to refresh the view as soon as we notify it by using *didChange* *Subject*. We send a *Void* value to *didChange* *Subject* on every mutation on *persons array*.

Now let's take a look at *PersonListView*.

```swift
struct PersonsView : View {
    @ObjectBinding var store: PersonStore

    var body: some View {
        NavigationView {
            List(store.persons) { person in
                VStack(alignment: .leading) {
                    Text(person.name)
                        .font(.headline)
                    Text("Age: \(person.age)")
                        .font(.subheadline)
                        .color(.secondary)
                }
            }
                .onAppear(perform: { self.store.fetch() })
                .navigationBarTitle(Text("Persons"))
        }
    }
}
```

We use List component to present an array of *Person* structs. Every row in *List* contains *VStack* with two *Text* components representing the name and age of a *Person*. We call *fetch* method on store object as soon as List appears. As you remember, our *PersonStore* object notifies *SwiftUI* about data change by using *Subject*, and *SwiftUI* rebuilds the view to present newly fetched data. 

#### Editing
Next step is creating a new view which allows us to edit personal information of selected *Person*. We will use *Form* component to show nice form for data entry. You can check my [previous post](/2019/06/19/building-forms-with-swiftui/) to learn more about *Form* component and its advantages. Let's dive into code which represents editing view.

```swift
struct EditingView: View {
    @Environment(\.isPresented) var isPresented: Binding<Bool>
    @Binding var person: Person

    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Personal information")) {
                    TextField($person.name)
                    Stepper(value: $person.age) {
                        Text("Age: \(person.age)")
                    }
                }

                Section {
                    Button(action: { self.isPresented?.value.toggle() }) {
                        Text("Save")
                    }
                }
            }.navigationBarTitle(Text(person.name))
        }
    }
}
```

Here we use *Binding* for selected person item. *Binding* property wrapper allows passing a reference to a value type. By using *Binding* property, *EditingView* can read and write to the *Person* struct, but it doesn't store a copy of it. We use this *Binding* to mutate value inside *PersonsStore* and as soon as we do that *SwiftUI* will update the view with the updated list of *Persons*. If you want to learn more about *Property Wrappers* available in *SwiftUI* like @*Binding*, @*Environment*, @*EnvironmentObject*, @*ObjectBinding*, please take a look at the [dedicated post](/2019/06/12/understanding-property-wrappers-in-swiftui/).

Now let's refactor our *PersonsView* to support editing by passing *Binding* to a selected *Person* inside *EditingView*. For that, we will use *PresentationButton* to present new beautiful cart interface in iOS 13.

```swift
struct PersonsView : View {
    @ObjectBinding var store: PersonStore

    var body: some View {
        NavigationView {
            List {
                ForEach(0..<store.persons.count) { index in
                    PresentationButton(destination: EditingView(person: self.$store.persons[index])) {
                        VStack(alignment: .leading) {
                            Text(self.store.persons[index].name)
                                .font(.headline)
                            Text("Age: \(self.store.persons[index].age)")
                                .font(.subheadline)
                                .color(.secondary)
                            }
                    }
                }
            }
                .onAppear(perform: { self.store.fetch() })
                .navigationBarTitle(Text("Persons"))
        }
    }
}
```

#### And here is the screenshot of our app, you can see how it looks.
![managing-data-flow-in-swiftui](/public/managing-data-flow-in-swiftui.png)

#### Conclusion
Today we built simple Master-Detail flow in *SwiftUI*. I've tried to show the power of *Bindings* in *SwiftUI*. You don't need to post notifications or observe key-value to indicate changes in your User Interface, all you need is using correct *Property Wrapper provided by SwiftUI*. Again, if you want to learn when and which one should be used, check out my [post about Property Wrappers in SwiftUI](/2019/06/12/understanding-property-wrappers-in-swiftui/). Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  