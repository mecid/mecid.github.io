---
title: Mastering NavigationStack in SwiftUI. Deep Linking.
layout: post
category: Mastering SwiftUI views
image: /public/navigation.png
---

This week we will continue exploring the new Navigation API in SwiftUI. One of the benefits of the new data-driven Navigation API is the programmatic navigation with deep-linking possibilities. Let's dive into the new API by learning how to build programmatic deep navigation flows.

> To learn about the basics of the new data-driven Navigation API in SwiftUI, look at my "Mastering NavigationStack in SwiftUI. Navigator Pattern." post.

#### Programmatic navigation
There is a special NavigationStack initializer accepting a binding to a mutable collection. SwiftUI maps values of the mutable collection into a view hierarchy and allows us to push and pop views into the NavigationStack programmatically. Let's take a look at the example.

```swift
struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path: [Product] = []

    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                Text(product.title)
                    .toolbar {
                        Button("Show similar") {
                            path.append(product.similar[0])
                        }
                    }
            }
            .task { await store.fetch() }
            .navigationDestination(for: Product.self) { product in
                ProductView(product: product)
            }
        }
    }
}
```

As you can see in the example above, we define a piece of state that drives our navigation via binding. We also display a button that adds another product to the path. SwiftUI automatically maps the contents of the path binding to the view hierarchy in the navigation stack and automatically removes the product from the path whenever the user presses the back button.

We also can quickly implement pop to the root view by removing all the items from the path. With this new data-driven approach, SwiftUI is responsible for keeping your navigation stack in sync with the path binding.

```swift
struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path: [Product] = []

    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                Text(product.title)
                    .toolbar {
                        Button("Show similar") {
                            path.append(product.similar[0])
                        }
                    }
            }
            .task { await store.fetch() }
            .navigationDestination(for: Product.self) { product in
                ProductView(product: product)
            }
            .toolbar {
                Button("Back to the list") {
                    path.removeAll()
                }
            }
        }
    }
}
```

Usually, our navigation stack contains different screens representing different values. In this case, we can define the path as an array of enum where every case specifies a particular destination in our app. 

```swift
enum Route: Hashable {
    case search(String)
    case product(Product)
    case related(Product)
}

struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path: [Route] = []

    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                Text(product.title)
                    .toolbar {
                        Button("Show similar") {
                            path.append(.related(product))
                        }
                    }
            }
            .task { await store.fetch() }
            .navigationDestination(for: Route.self) { route in
                switch route {
                case let .related(product):
                    ProductView(product: product.similar[0])
                case let .product(product):
                    ProductView(product: product)
                case let .search(query):
                    SearchView(query: query)
                }
            }
        }
    }
}
```

#### Programmatic navigation with multiple scenes
One thing I have to mention is that you never should define the path in the App protocol. In this case, you will have a synchronized navigation stack across all of the scenes of your app. Usually, users create multiple scenes of our apps to use different parts of our apps simultaneously.

```swift
@main
// Never do this!
struct NavigationTestApp: App {
    @State private var path: [Route] = []
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $path) {
                RootView()
            }
        }
    }
}
```

Instead, we should hold our state defining the path in the root view of our app or inside a container view representing a particular user flow.

#### Deep linking and handoff
Let's move to the next thing we might need in our app: deep linking and handoff. With the new data-driven Navigation API, we can quickly implement deep linking and handoff support. All we need to do is handle the URL and add the corresponding value to the array specifying the path of the navigation stack.

```swift
struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path: [Route] = []

    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                Text(product.title)
            }
            .task { await store.fetch() }
            .navigationDestination(for: Route.self) { route in
                // ...
            }
            .onOpenURL { url in
                let components = URLComponents(string: url.absoluteString)
                let searchQuery = components?.queryItems?.first { $0.name == "query" }?.value

                guard let query = searchQuery else {
                    return
                }
                path.append(.search(query))
            }
            .onContinueUserActivity("com.app.search") { activity in
                guard let query = activity.userInfo?["query"] as? String else {
                    return
                }
                path.append(.search(query))
            }
        }
    }
}
```

#### State restoration
State restoration is one of the essential features that you should implement to provide a pleasant user experience. SwiftUI provides the SceneStorage property wrapper, allowing us to keep data in the specific storage bound to the scene and survive when the system shuts down the app.

> To learn more about state restoration in SwiftUI, look at my "State restoration in SwiftUI" post.

We can use the SceneStorage property wrapper to encode our navigation path and store it in the scene memory. Whenever the system kills the app, we can restore the path from the scene storage and programmatically navigate to the last entry.

```swift
@MainActor final class NavigationStore<Route: Hashable>: ObservableObject {
    @Published var path: [Route] = []

    private let decoder = JSONDecoder()
    private let encoder = JSONEncoder()
    private let urlHandler: any UrlHandler<Route>
    private let activityHandler: any ActivityHandler<Route>

    init(
        urlHandler: some UrlHandler<Route>,
        activityHandler: some ActivityHandler<Route>
    ) {
        self.urlHandler = urlHandler
        self.activityHandler = activityHandler
    }

    func handle(_ activity: NSUserActivity) {
        activityHandler.handle(activity, mutating: &path)
    }

    func handle(_ url: URL) {
        urlHandler.handle(url, mutating: &path)
    }
}

extension NavigationStore where Route: Codable {
    func encoded() -> Data? {
        try? encoder.encode(path)
    }
    
    func restore(from data: Data) {
        do {
            path = try decoder.decode([Route].self, from: data)
        } catch {
            path = []
        }
    }
}
```

Here we have the NavigationStore class providing the common functionality for deep linking and handoff support. It also allows us to encode our path and decode it from the serialized representation. Now we can use it in our root view for state restoration whenever needed.

```swift
struct RootView: View {
    @SceneStorage("navigation") private var path: Data?
    @StateObject private var store = Store()
    @StateObject private var navigation = NavigationStore<Route>(
        urlHandler: SomeUrlHandler(),
        activityHandler: SomeActivityHandler()
    )
    
    var body: some View {
        NavigationStack(path: $navigation.path) {
            CategoriesView(categories: store.categories)
                .task { await store.fetch() }
                .navigationDestination(for: Route.self) { route in
                    // ...
                }
                .onOpenURL { navigation.handle($0) }
        }
        .task {
            if let path {
                navigation.restore(from: path)
            }
            
            for await _ in navigation.$path.values {
                path = navigation.encoded()
            }
        }
    }
}
```

As you can see in the example above, we use the task view modifier to restore the navigation from the scene storage and observe the navigation path to save it as soon as any changes appear.

#### Conclusion
Today we learned how to use the new data-driven Navigation API to control our navigation stack programmatically and implement the deep linking feature. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
