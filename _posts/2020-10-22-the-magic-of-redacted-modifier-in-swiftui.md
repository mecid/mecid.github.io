---
title: The magic of redacted modifier in SwiftUI
layout: post
category: Interactions
image: /public/redacted.jpeg
---

Redacted modifier is the thing that will have a great impact on how iOS apps handle loading states. During WWDC20, Apple showed us the easy way of hiding the data from home-screen widgets using the redacted modifier. Today we will talk about using the redacted modifier to hide sensitive data and handle loading states.

{% include friends.html %}

#### Redacted modifier
The redacted modifier transforms the view hierarchy into a skeleton view when added. Don't worry if you are not familiar with the skeleton view pattern. You will see how it works very soon. Assume that you are working on the Github app. You have a view that represents a repo on the list. 

```swift
struct Repo: Hashable, Decodable {
    let name: String
    let description: String
    let stars: Int
}

struct RepoView: View {
    let repo: Repo

    var body: some View {
        HStack {
            VStack(alignment: .center) {
                Image(systemName: "star.fill")
                    .resizable()
                    .frame(width: 44, height: 44)
                Text(String(repo.stars))
                    .font(.title)
            }.foregroundColor(.red)

            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                Text(repo.description)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

Let's create a sample data that we can use to preview our *RepoView*.

```swift
extension Repo {
    static let mock = Repo(
        name: "SwiftUICharts",
        description: "A simple line and bar charting library that support accessibility written using SwiftUI. ",
        stars: 579
    )
}
```

Now we can use our *RepoView* in preview to see how it looks with or without a redacted modifier.

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            RepoView(repo: .mock)
            Divider()
            RepoView1(repo: .mock)
                .redacted(reason: .placeholder)
        }
    }
}
```

![redacted](/public/redacted1.png)

As you can see in the example above, we have a plain *RepoView* on the left and a redacted version on the right. The redacted modifier transforms images and text views in the view hierarchy to hide its content using overlays. Let's take a look at a more advanced example.

```swift
final class Store: ObservableObject {
    @Published private(set) var repos: [Repo]
    @Published private(set) var isLoading = false

    private let service: GithubService
    init(
        service: GithubService,
        initialState: [Repo] = Array(repeating: .mock, count: 5)
    ) {
        self.repos = initialState
        self.service = service
    }

    func fetch() {
        isLoading = true
        service
            .fetchRepos(matching: "SwiftUI")
            .replaceError(with: [])
            .receive(on: DispatchQueue.main)
            .handleEvents(receiveCompletion: { [weak self] _ in self?.isLoading = false})
            .assign(to: &$repos)
    }
}
```

Here we have a store object that handles the data loading. We will use the redacted modifier to hide the mock data that we have as our store object's initial state.

> To learn more about store objects, take a look at my ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

```swift
struct ContentView: View {
    @StateObject var store = Store(service: .init())

    var body: some View {
        List(store.repos, id: \.self) { repo in
            RepoView(repo: repo)
        }
        .onAppear(perform: store.fetch)
        .redacted(reason: store.isLoading ? .placeholder : [])
    }
}
```

While attaching the redacted modifier, we have to provide an instance of *RedactionReasons* struct using the reason parameter. *RedactionReasons* is an option set that we can extend with as many reasons as we need. *RedactionReasons* struct provides us a ready to use placeholder instance that we use in the example above.

Remember that the redacted modifier hides the data only visually. It is still clickable in case of buttons. It is your responsibility to disable buttons while using the redacted modifier.

#### Unredacted modifier
As we already know, the redacted modifier traverses the view hierarchy and applies its effect to hide the actual data, but what if we want to keep a certain part of the view visible? SwiftUI provides us another modifier called unredacted. Unredacted modifier allows us to keep the view unredacted while applying the redacted modifier.

```swift
struct RepoView: View {
    let repo: Repo

    var body: some View {
        HStack {
            VStack(alignment: .center) {
                Image(systemName: "star.fill")
                    .resizable()
                    .frame(width: 44, height: 44)
                    .unredacted()
                Text(String(repo.stars))
                    .font(.title)
            }.foregroundColor(.red)

            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                Text(repo.description)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

![unredacted](/public/unredacted.png)

#### Reasons
As we learned, the redacted modifier accepts a reason parameter. It's great that we can create as many different reasons and hide only the part we need. SwiftUI provides a special environment value called *redactionReasons* to get the redaction reason applied to the current view hierarchy. Let's start first with the extending *RedactionReasons* struct with more options.

```swift
extension RedactionReasons {
    static let text = RedactionReasons(rawValue: 1 << 2)
    static let images = RedactionReasons(rawValue: 1 << 4)
}
```

> To learn more about *OptionSet* protocol in Swift, take a look at my ["Inclusive enums with OptionSet"](/2019/04/10/inclusive-enums-with-optionset/) post.

Now we can tune our *RepoView* to redact the only needed parts of the view.

```swift
struct RepoView1: View {
    @Environment(\.redactionReasons) var reasons
    let repo: Repo

    var body: some View {
        HStack {
            VStack(alignment: .center) {
                Image(systemName: "star.fill")
                    .resizable()
                    .frame(width: 44, height: 44)
                    .unredacted(when: !reasons.contains(.images))
                Text(String(repo.stars))
                    .font(.title)
                    .unredacted(when: !reasons.contains(.text))
            }.foregroundColor(.red)

            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                Text(repo.description)
                    .foregroundColor(.secondary)
            }.unredacted(when: !reasons.contains(.text))
        }
    }
}
```

Remember that SwiftUI applies skeleton view effect only when we use placeholder redaction reason. Any other reasons should be handled manually.

```swift
extension View {
    @ViewBuilder func unredacted(when condition: Bool) -> some View {
        if condition {
            unredacted()
        } else {
            // Use default .placeholder or implement your custom effect
            redacted(reason: .placeholder)
        }
    }
}
```

#### Conclusion
Today we learned another great feature that SwiftUI provides us for free out of the box. I really love the skeleton view pattern, and with SwiftUI, I started using it on every screen that loads some data. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!