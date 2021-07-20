---
title: Submitting values to SwiftUI view
layout: post
image: /public/todo.jpeg
category: Data Flow
---

SwiftUI Release 3 brought us a new declarative approach for handling submitted values. Text fields, forms, search bars allow users to submit values that we can take and react to them using the new *onSubmit* view modifier. This week we will learn how to use the *onSubmit* view modifier and what benefits it provides us.

{% include friends.html %}

#### Basics
Let's build a view that allows us to search for messages using the *searchable* view modifier.

```swift
struct SearchView: View {
    @ObservedObject var viewModel: ViewModel
    @Binding var query: String

    var body: some View {
        List(viewModel.messages, id: \.self) { message in
            Text(message)
        }
        .searchable(text: $query)
        .onSubmit(of: .search) {
            viewModel.search(matching: query)
        }
        .navigationTitle("Search")
    }
}
```

As you can see in the example above, we display the list of messages from *ViewModel* and provide the search functionality. We also use the *onSubmit* view modifier to provide a closure that SwiftUI runs whenever the user submits the value. As soon as the user hits the return key on the software or hardware keyboard, SwiftUI calls the provided closure.

> To learn more about using the search view modifier, take a look at my ["Mastering search in SwiftUI"](/2021/06/23/mastering-search-in-swiftui/) post.

We use the *onSubmit* view modifier with a search submit trigger. It means SwiftUI runs the given closure only as a result of search action. SwiftUI provides us a set of different submit triggers like *search, text, form,* and its count can increase in the future releases of SwiftUI.

Other views which we can use in conjunction with the *onSubmit* view modifier are *TextField* and *SecureField*. We can attach the *onSubmit* view modifier directly to the text field. In this case, we have to use the text submit trigger.

```swift
struct ContentView: View {
    @State private var query = ""

    var body: some View {
        NavigationView {
            TextField("query", text: $query)
                .onSubmit(of: .text) {
                    print(query)
                }
        }
    }
}
```

Keep in mind that we can attach multiple *onSubmit* view modifiers with various submit triggers to the view hierarchy and provide different closures for separate triggers.

We can also change the label of the return key on the software keyboard using the *submitLabel* view modifier. *submitLabel* view modifier requires the instance of *SubmitLabel* struct as the parameter which defined the return key label. It has many predefined values like *done, go, send, join, route, search, return, next and continue.*

```swift
struct ContentView: View {
    @State private var query = ""

    var body: some View {
        NavigationView {
            TextField("query", text: $query)
                .submitLabel(.send)
                .onSubmit(of: .text) {
                    print(query)
                }
        }
    }
}
```

#### Scopes
I should mention that you can place the *onSubmit* view modifier not only under text fields, but it can be anywhere in the view hierarchy. That's why SwiftUI provides us an opportunity to control submit scopes. For example, you can disable a part of the view hierarchy to react on submitting values.

```swift
struct ContentView: View {
    @StateObject private var viewModel = ViewModel()

    var body: some View {
        NavigationView {
            VStack {
                TextField("phone", text: $viewModel.phone)
                    .submitScope(viewModel.phone.count > 11)

                VStack {
                    TextField("email", text: $viewModel.email)
                    TextField("password", text: $viewModel.password)
                }
            }
            .onSubmit(of: .text) {
                viewModel.signUp()
            }
        }
    }
}
```

The *submitScope* view modifier allows you to avoid specific views from invoking a submission action. In our example, the phone text field will not initiate a submission while the provided condition is false.

#### Conclusion
This week, we learned about the new *onSubmit* view modifier that provides us with a generic way to handle the value submission for different views. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

