---
title: Mastering NavigationStack in SwiftUI. NavigationPath.
layout: post
category: Mastering SwiftUI views
image: /public/navigation.png
---

SwiftUI provides us with a brand new data-driven navigation API allowing us to map a value to a destination in the view hierarchy. This week I want to continue the story of the new navigation API in SwiftUI by covering another tool. We will learn how to use the *NavigationPath* type to build a navigation stack with different destinations.

> To learn about the basics of the new data-driven Navigation API in SwiftUI, look at my ["Mastering NavigationStack in SwiftUI. Navigator Pattern."](/2022/06/15/mastering-navigationstack-in-swiftui-navigator-pattern/) post.

As you might know from my previous posts, SwiftUI provides value-based navigation links allowing us to bind value programmatically to any view in the navigation stack. Here is a quick example.

```swift
struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path: [Product] = []

    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                NavigationLink(product.title, value: product)
            }
            .task { await store.fetch() }
            .navigationDestination(for: Product.self) { product in
                ProductView(product: product)
                    .toolbar {
                        Button("Show similar") {
                            path.append(product.similar[0])
                        }
                    }
            }
        }
    }
}
```

In the example above, we define the piece of state that derives our navigation stack. In this case, we can push only values of type *Product*. Usually, we want to push different views into the navigation stack. To achieve that, we can define an enum type covering all the possible cases of navigation destinations we have in the app. But suppose you are a bit lazy to define an enum type. In that case, SwiftUI provides you with a type called *NavigationPath*, allowing us to store any hashable value and map them to the destination in the navigation stack.

```swift
struct ShopContainerView: View {
    @StateObject private var store = Store()
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            List(store.products) { product in
                NavigationLink(product.title, value: product)
            }
            .task { await store.fetch() }
            .navigationDestination(for: String.self) { query in
                SearchView(query: query)
            }
            .navigationDestination(for: Product.self) { product in
                ProductView(product: product)
                    .toolbar {
                        Button("Show similar") {
                            path.append(product.query)
                        }
                    }
            }
        }
    }
}
```

As you can see in the example above, we define a variable of the type *NavigationPath* to store the whole navigable state. *NavigationPath* erases the type of pushed values and allows us to keep values of different types. The only requirement is to push only hashable values.

Another bonus of using *NavigationPath* is the codable representation of pushed values. The *NavigationPath* provides the codable property of type *CodableRepresentation*, allowing us to encode pushed values into the *Data* type and store it somewhere. Make sure that the values you push conform to *Codable*. Otherwise, the codable representation of the *NavigationPath* will be *nil*.

The *NavigationPath* type also has a particular initializer accepting a value of type *CodableRepresentation* to restore the whole navigation stack from the serialized representation.

```swift
protocol UrlHandler {
    func handle(_ url: URL, mutating: inout NavigationPath)
}

protocol ActivityHandler {
    func handle(_ activity: NSUserActivity, mutating: inout NavigationPath)
}

@MainActor final class NavigationStore: ObservableObject {
    @Published var path = NavigationPath()
    
    private let decoder = JSONDecoder()
    private let encoder = JSONEncoder()
    private let urlHandler: UrlHandler
    private let activityHandler: ActivityHandler
    
    init(urlHandler: UrlHandler, activityHandler: ActivityHandler) {
        self.urlHandler = urlHandler
        self.activityHandler = activityHandler
    }
    
    func handle(_ activity: NSUserActivity) {
        activityHandler.handle(activity, mutating: &path)
    }
    
    func handle(_ url: URL) {
        urlHandler.handle(url, mutating: &path)
    }
    
    func encoded() -> Data? {
        try? path.codable.map(encoder.encode)
    }
    
    func restore(from data: Data) {
        do {
            let codable = try decoder.decode(
                NavigationPath.CodableRepresentation.self, from: data
            )
            path = NavigationPath(codable)
        } catch {
            path = NavigationPath()
        }
    }
}
```

Here is the implementation of the *NavigationStore* type that I use in my apps to maintain navigation and deep linking logic. As you can see, I use the *NavigationPath* type to keep the state of a navigation stack. There are also helper functions allowing me to serialize and restore the navigation state quickly.

```swift
struct AppContainerView: View {
    @StateObject private var navigationStore = NavigationStore(
        urlHandler: SomeUrlHandler(),
        activityHandler: SomeActivityHandler()
    )
    
    @SceneStorage("navigation")
    private var navigationData: Data?
    
    var body: some View {
        NavigationStack(path: $navigationStore.path) {
            ContentView()
                .environmentObject(navigationStore)
                .onOpenURL { navigationStore.handle($0) }
                .navigationDestination(for: String.self) { string in
                    Text(string)
                }
                .task {
                    if let navigationData {
                        navigationStore.restore(from: navigationData)
                    }
                    
                    for await _ in navigationStore.$path.values {
                        navigationData = navigationStore.encoded()
                    }
                }
        }
    }
}
```

As you can see in the example above, we use our *NavigationStore* to serialize and store the state of navigation in the scene storage.

> To learn more about state restoration in SwiftUI, look at my ["State restoration in SwiftUI"](/2022/03/10/state-restoration-in-swiftui/) post.

Today we learned how to use the *NavigationPath* type to push different views programmatically without defining additional types. We also learned how to serialize and store the current state of navigation in the scene storage to provide a better user experience. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
