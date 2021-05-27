---
title: SwiftUI wishlist for WWDC21
layout: post
category: Meta
image: /public/wwdc21.jpg
---
WWDC21 is coming pretty soon, and it is a great chance to think about the new features that I want to see in SwiftUI. This wishlist contains not only the list of the features I want to use but also possible APIs. Remember that this post is the result of my imagination, and most of the code examples don't exist at the moment. 

{% include friends.html %}

#### List and ScrollView
SwiftUI provides you both *List* and *ScrollView*, but under the hood, these views still use the UIKit implementation of *UITableView* and *UIScrollView*. I love how *UITableView* works and the API it provides us. But SwiftUI's *List* and *ScrollView* don't expose all the powerful features of *UITableView* and *UIScrollView*.

Almost all the screens in my apps use the List view in different styles, and I hope to see more APIs from *UITableView*, which allows styling separators, cell backgrounds using the *ListStyle* protocol.

```swift
struct ContentView: View {
    let messages: [String]

    var body: some View {
        List {
            ForEach(messages, id: \.self) { message in
                Text(message)
            }
        }.listStyle(DefaultListStyle(separator: .none, selection: .single))
    }
}
```

*ScrollView* is another crucial component for many screens. I usually build my apps with accessibility in mind and support Dynamic Type out of the box. *ScrollView* is must-have root view for every screen where you want Dynamic Type. *ScrollView* in SwiftUI is still missing paging and content offset features that we used to see in *UIScrollView*.

```swift
struct RootView: View {
    @State private var offset: CGPoint = .zero

    var body: some View {
        ScrollView(.vertical, showsIndicators: false, offset: $offset) {
            Text("Very long text")
            Button("Jump to top") {
                self.offset = .zero
            }
        }
    }
}
```

> We can read the content offset of ScrollView using preferences API in SwiftUI. To learn more, take a look at my dedicated ["Mastering ScrollView in SwiftUI"](/2020/09/24/mastering-scrollview-in-swiftui/) post.

#### CompositionalLayout
During the last year, Apple gave us *LazyHGrid* and *LazyVGrid* views, which we can use to build views like calendars or photo grids. Grids work great, and I love them, but we want to use the power of *CompositionalLayout* that we have in UIKit. I don't think that Apple should get rid of *LazyHGrid* and *LazyVGrid* views, but they can introduce a new *CompositionaView* that supports all the features of *CompositionalLayout*.

```swift
struct AppStoreView: View {
    let featured: [App]
    let appsWeLove: [App]

    var body: some View {
        CompositionaView {
            Section(.groupPagingCentered) {
                Group(.horizontal, width: .fractionalWidth(0.5), height: .fractionalHeight(0.5)) {
                    ForEach(featured) { app in
                        FeatureAppView(app: app)
                    }
                }
            }

            Section {
                Group(.vertical, width: .fractionalWidth(0.9), height: .estimated(200)) {
                    ForEach(appsWeLove) { app in
                        SmallAppView(app: app)
                    }
                }
            }
        }
    }
}
```

#### Navigation
Navigation is another point of pain in SwiftUI. It works great for simple use cases but doesn't provide enough flexibility for complex solutions. What I want to use is some sort of *RouterView* that maps destinations to concrete views.

```swift
enum Destination {
    case home
    case login
    case profile(UUID)
    case settings
}

struct RootView: View {
    var body: some View {
        RouterView(initial: Destination.home) { destination in
            switch destination {
            case .home: HomeView()
            case .login: LoginView()
            case let .profile(user): ProfileView(user: user)
            case .settings: SettingsView()
            }
        }
    }
}
```

We also can have a router in the environment and mutate the state of navigation using the environment value.

```swift
struct ProfileView: View {
    let user: UUID
    @Environment(\.router) var router

    var body: some View {
        VStack {
            Button("push settings screen") {
                router.push(.settings)
            }

            Button("go home") {
                router.popToRoot()
            }
        }
    }
}
```

> To learn more about advanced techniques while building navigation in SwiftUI, take a look at my ["Lazy navigation in SwiftUI"](/2021/01/27/lazy-navigation-in-swiftui/) post.

#### Focus management and text fields
Sign-up form with multiple text fields is what I usually implement using UIKit and then wrap with *UIViewControllerRepresentable*. It is literally impossible to handle the first responder in SwiftUI and move the focus from one view to another.

```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""

    @State private var hasFilledCredentials = false
    @Namespace private var namespace

    @Environment(\.resetFocus) var resetFocus

    var body: some View {
        VStack {
            TextField("email", text: $email)
                .prefersDefaultFocus(!hasFilledCredentials, in: namespace)

            SecureField("password", text: $password)

            Button("login") {}
                .prefersDefaultFocus(hasFilledCredentials, in: namespace)

            Button("reset credentials") {
                hasFilledCredentials = false
                resetFocus(in: namespace)
            }
        }.focusScope(namespace)
    }
}
```

You might wonder, but this is the real API that we have in SwiftUI at the moment. Unfortunately, it is available only for tvOS and watchOS, but I wish to see it for iOS and macOS.

> To learn more about focus management in SwiftUI, take a look at my ["Focus management in SwiftUI"](/2020/12/02/focus-management-in-swiftui/) post.

#### Conclusion
I didn't mention small things like *Pull-to-Refresh, SearchBar, TextView*, but I still expect them to appear. SwiftUI has its own set of pros and cons. But for me, it is the way to go with my projects. I'm so excited about the upcoming WWDC and hope to see at least a part of the features I mentioned in this post. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!