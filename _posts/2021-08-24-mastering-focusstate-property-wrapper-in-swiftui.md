---
title: Mastering FocusState property wrapper in SwiftUI
layout: post
image: /public/focus-state.png
category: Interactions
---

SwiftUI became very powerful during the last WWDC. We gained many new features, and one of them was a brand new *FocusState* property wrapper. *FocusState* property wrapper allows us to read and write the current focus position in the view hierarchy. This week we will learn how to manage focus in SwiftUI apps using *FocusState* property wrapper and *focused* view modifier.

{% include friends.html %}

SwiftUI provides a new *FocusState* property wrapper that works on all Apple platforms and allows us to focus on a particular view or check if that view is already focused. The usage is effortless. Let's see how we can use it.

```swift
import SwiftUI

struct SignInView: View {
    @FocusState private var isEmailFocused: Bool
    @State private var email = ""

    var body: some View {
        NavigationView {
            Form {
                TextField("email", text: $email, prompt: Text("email"))
                    .focused($isEmailFocused)
            }
            .navigationTitle("Sign in")
            .onChange(of: isEmailFocused) { newValue in
                print(newValue)
            }
        }
    }
}
```

As you can see in the example above, we need to define a boolean variable using the *FocusState* property wrapper. We also have to bind its value to the focus state of a particular view using the *focused* view modifier. SwiftUI sets the boolean value of the view to true as soon as the user focuses on it. It also changes it to false as soon as the view loses focus.

> To learn more about focus management in SwiftUI, take a look at my ["Focus management in SwiftUI"](/2020/12/02/focus-management-in-swiftui/) post.

You can define as many *FocusState* variables as needed to cover your focus management logic. SwiftUI takes care of them by keeping in sync the focused view with its binding.

```swift
import SwiftUI

struct SignInView: View {
    @FocusState private var isEmailFocused: Bool
    @FocusState private var isPasswordFocused: Bool

    @State private var email = ""
    @State private var password = ""

    var body: some View {
        NavigationView {
            Form {
                TextField("email", text: $email, prompt: Text("email"))
                    .focused($isEmailFocused)
                SecureField("password", text: $password, prompt: Text("password"))
                    .focused($isPasswordFocused)
            }
            .navigationTitle("Sign in")
        }
    }
}
```

In the example above, we have two variables bound to email and password text fields. SwiftUI can manage them together and keep them in sync with the user interface. Remember that you can change the value to false programmatically to hide the keyboard or set the value to true to move the focus to a particular view.

```swift
import SwiftUI

struct SignInView: View {
    @FocusState private var isEmailFocused: Bool
    @FocusState private var isPasswordFocused: Bool

    @State private var email = ""
    @State private var password = ""

    var body: some View {
        NavigationView {
            Form {
                TextField("email", text: $email, prompt: Text("email"))
                    .focused($isEmailFocused)
                SecureField("password", text: $password, prompt: Text("password"))
                    .focused($isPasswordFocused)

                Button("login") {
                    if email.isEmpty {
                        isEmailFocused = true
                    } else if password.isEmpty {
                        isPasswordFocused = true
                    } else {
                        isPasswordFocused = false
                        isEmailFocused = false

                        login()
                    }
                }
            }
            .navigationTitle("Sign in")
        }
    }

    private func login() {
        // your logic here
    }
}
```

> To learn about other focus related property wrappers in SwiftUI, take a look at my ["FocusedValue and FocusedBinding property wrappers in SwiftUI"](/2021/03/03/focusedvalue-and-focusedbinding-property-wrappers-in-swiftui/) post.

Defining many *FocusState* properties in complex view hierarchies can become cumbersome. Fortunately, *FocusState* works not only with boolean values but also with any *Hashable* type. It means that we can model the focused state using an enum type conforming to *Hashable* protocol. Let's take a look at the example.

```swift
enum FocusableField: Hashable {
    case email
    case password
}
```

Here we have the *Field* enum that conforms to *Hashable* and defines all the focusable views we manage. Now, we can use this enum to bind the focus state of different views to various enum cases.

```swift
struct ContentView: View {
    @State private var email = ""
    @State private var password = ""
    @FocusState private var focus: FocusableField?

    var body: some View {
        NavigationView {
            Form {
                TextField("email", text: $email, prompt: Text("email"))
                    .focused($focus, equals: .email)
                SecureField("password", text: $password, prompt: Text("password"))
                    .focused($focus, equals: .password)
                Button("login", action: login)
            }
            .toolbar {
                ToolbarItem(placement: .keyboard) {
                    Button("next") {
                        if email.isEmpty {
                            focus = .email
                        } else if password.isEmpty {
                            focus = .password
                        } else {
                            focus = nil
                        }
                    }
                }
            }
            .navigationTitle("Sign in")
            .defaultFocus($focus, .email)
        }
    }

    private func login() {
        // your logic here
    }
}
```

As you can see, we use another version of the *focused* view modifier to bind a view to a concrete case of the *Field* enum. SwiftUI updates the value of the *FocusState* property whenever the user focuses on any of the bound views. We also use, the *defaultFocus* view modifier to programmatically focus on the email field as soon as view appears.

Remember that we should make our *FocusState* property optional to use it with *Hashable* enum because there might be no focused view at the moment.

```swift
struct SearchView: View {
    enum FocusSection: Hashable {
        case results
        case search
    }
    
    @FocusState private var focus: FocusSection?
    
    @ObservedObject var store: SearchStore

    var body: some View {
        SearchResultsView(results: store.results)
            .focused($focus, equals: .results)
            .onAppear { focus = .search }
            .searchable(
                text: store.$query,
                prompt: Text("pasta", bundle: .module)
            )
            .searchFocused($focus, equals: .search)
            .task(id: store.query) {
                await store.search()
            }
    }
}
```

SwiftUI also introduces the *searchFocused* view modifier, allowing us to bind the focus of a search field that we create using the *searchable* view modifier. It works the same way and supports binding both for boolean and *Hashable* types. The only difference is that it only affects the text field in the search bar of the current view hierarchy.

> To learn more about building search user experience in SwiftUI, take a look at my ["Mastering search in SwiftUI"](/2021/06/23/mastering-search-in-swiftui/) post.

Today we learned how to use the *FocusState* property wrapper to manage focus in our views. Remember that *FocusState* allows us both to read and change the focused view programmatically. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

