---
title: SwiftUI Performance - How to use UIKit
layout: post
category: View Composition
---

Nowadays, Apple platform development has undergone significant changes. Previously, we believed that building the core of an app around UIKit and using SwiftUI for certain screens was a good idea. This week, we’ll delve into the foundation of app development using SwiftUI, while also exploring UIKit for scenarios where performance truly matters.

{% include friends.html %}

Whenever I say core part of the app, I mean app lifecycle and navigation. Using UIKit, we can build the navigation using the pretty popular coordinator pattern and push SwiftUI views by wrapping them with UIHostingController. We can still use the app delegate for managing the app lifecycle. That works great in terms of performance and legacy codebase. But, unfortunately, this approach has its own downsides.

```swift
struct Coordinator {
    let navigation: UINavigationController
    
    init(navigation: UINavigationController) {
        self.navigation = navigation
    }
    
    func start() {
        let auth = UIHostingController(rootView: AuthView())
        navigation.viewControllers = [auth]
    }
    
    func startMainFlow() {
        let main = UIHostingController(rootView: MainView())
        navigation.viewControllers = [main]
    }
}


class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    private lazy var navigation = UINavigationController()
    private lazy var coordinator = Coordinator(navigation: navigation)
    
    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = navigation
        window?.makeKeyAndVisible()
        coordinator.start()
    }
}
```

Every UIHostingController creates its own SwiftUI environment, which means you can’t share data via environment. Environment in SwiftUI plays a huge role in anything related to data propagation through the view hierarchy. SwiftUI uses environment for many things, starting from view styling and sharing data. That’s why I never liked this approach; it doesn’t allow us to fully engage with SwiftUI features.

Navigation in SwiftUI was really weak, very hard to build deep linking and a modular approach. But with the NavigationStack released on iOS 16, we gain a pretty new data-driven API allowing us to build deep-linking functionality in an easy way.

First of all, I should mention that building the app around UIKit is still a good idea if you are going to support platform versions prior to iOS 16. For modern apps targeting iOS 16 and above, I recommend building the core of the app using SwiftUI and incorporating UIKit in certain parts where SwiftUI’s performance may not meet your expectations.

```swift
@main
struct ExamplesApp: App {
    @State private var store = Store()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                HugeListView(items: store.items)
            }
        }
    }
}
```

We can expect that SwiftUI App Lifecycle and the recent navigation API allow us to handle almost every feature we need: deep-linking, state restoration, etc. Unfortunately, there are some cases where the performance of SwiftUI may cause some glitches. Especially when you have infinite collections of data like social timelines or calendar layouts.

Don’t worry because we already have a solution for this case called UIHostingConfiguration. It is a new type of UITableViewCell or UICollectionViewCell configuration allowing us to embed SwiftUI views into a cell, but still use the performant dequeuing capabilities of the UICollectionView. This approach works great where performance really matters.

```swift
struct HugeListView: UIViewRepresentable {
    let items: [Item]
    
    func makeUIView(context: Context) -> UITableView {
        let tableVIew = UITableView()
        tableVIew.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableVIew.dataSource = context.coordinator
        return tableVIew
    }
    
    func updateUIView(_ uiView: UITableView, context: Context) {
        uiView.reloadData()
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(items: items)
    }
    
    final class Coordinator: NSObject, UITableViewDataSource {
        let items: [Item]
        
        init(items: [Item]) {
            self.items = items
        }
        
        func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return items.count
        }
        
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let item = items[indexPath.row]
            let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
            cell.contentConfiguration = UIHostingConfiguration {
                ItemView(item: item)
            }
            
            return cell
        }
    }
}
```

Keep in mind that UIHostingConfiguration is created only for embedding SwiftUI views in the UICollectionView or UITableView. Don’t use NavigationLinks inside the UIHostingConfiguration. Instead, leverage the Coordinator to handle selection and push your values to the instance of  the NavigationStack.

SwiftUI has evolved significantly, making it a viable choice for building the core of modern iOS apps, especially with the improved navigation system in iOS 16. However, UIKit remains essential for handling performance-sensitive scenarios, particularly when working with large datasets or complex UI structures.
