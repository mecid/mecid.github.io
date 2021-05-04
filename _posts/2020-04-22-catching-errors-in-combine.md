---
title: Catching errors in Combine
layout: post
image: /public/catch.png
category: Combine framework
---

The Combine framework provides a declarative *Swift API* for processing values over time. It is another excellent framework that released side-by-side with SwiftUI. I already covered it multiple times on my blog, but today I want to talk about one of the key aspects of data processing. Today we will learn how to handle errors during data processing using the Combine framework.

{% include friends.html %}

#### The lifecycle of publisher
We use publishers to emit values over time. In the end, the publisher can finish its work by sending the completion event or fail with an error. Neither after completion nor after failure, the publisher can not emit new values. Let's take a look at the typical use-case that we might have in our apps.

```swift
import SwiftUI

struct SearchView: View {
    @ObservedObject var store: SearchStore

    var body: some View {
        NavigationView {
            List {
                TextField("type something...", text: $store.query)
                ForEach(store.repos, id: \.id) { repo in
                    Text(repo.name)
                }
            }.navigationBarTitle("Search")
        }
    }
}
```

In the example above, we have a view that allows users to enter a keyword and the list that renders search results. We use a store object to bind a query and repos array that holds search results. The main goal of the store object is fetching repos matching the query using Github API.

```swift
final class SearchStore: ObservableObject {
    @Published var query: String = ""
    @Published private(set) var repos: [Repo] = []
    private var cancellable: AnyCancellable?

    init(service: GithubService) {
        cancellable = $query
            .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
            .flatMap { service.searchPublisher(matching: $0) }
            .subscribe(on: DispatchQueue.global())
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { completion in
                    switch completion {
                    case .failure(let error): print("Error \(error)")
                    case .finished: print("Publisher is finished")
                    }
            },
                receiveValue: { [weak self] in self?.repos = $0 }
        )
    }
}
```

I don't want to produce too many requests as user types a query. That's why I debounce it for 500ms. It means the publisher will wait at least 500ms whenever the user stops typing and then publish a value. Then I can use a *flatMap* operator to run a search request using a query. In the end, I use the *sink* subscriber to assign search results to a store variable. As soon as published variables change, SwiftUI will update the view to respect new data.

> To learn more about store concept, take a look at my ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

We have one problem here, whenever the publisher fails with an error, nothing will happen in the view. *Sink* subscriber will just print the message in the console.

#### Replace error with the value
Publishers provide us a few ways to handle errors in the chain. Let's start with the easiest one. Every publisher that can fail allows us to replace the error with some default value using *replaceError* operator. Let's take a look at how we can use it.

```swift
init(service: GithubService) {
    $query
        .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
        .flatMap { service.searchPublisher(matching: $0) }
        .replaceError(with: [])
        .subscribe(on: DispatchQueue.global())
        .receive(on: DispatchQueue.main)
        .assign(to: &$repos)
}
```

As you can see, I have inserted *replaceError* operator with an empty array. Publisher will replace any error that might occur with an empty array and then terminate. The downside here is that the publisher completes its work after an error. It means our binding will not process new values. The user will type further queries, but nothing will happen. Let's see how we can fix that.

```swift
init(service: GithubService) {
    $query
        .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
        .flatMap { 
            service
                .searchPublisher(matching: $0)
                .replaceError(with: []) 
        }
        .subscribe(on: DispatchQueue.global())
        .receive(on: DispatchQueue.main)
        .assign(to: &$repos)
}
```

To keep your publisher alive, you have to handle all errors outside of the main chain. We can fix it by moving *replaceError* operator inside the *flatMap*. As you can see, the publisher inside the *flatMap* replaces an error with a value and then terminate its work. The main chain is still alive and emits new values.

#### Retry operator
Another useful operator that can help to solve an issue during value processing is *retry*. *Retry* operator tries to process the value again and again as many times as you specify.

```swift
init(service: GithubService) {
    $query
        .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
        .flatMap {
            service
                .searchPublisher(matching: $0)
                .retry(3)
                .replaceError(with: [])
        }
        .subscribe(on: DispatchQueue.global())
        .receive(on: DispatchQueue.main)
        .assign(to: &$repos)
}
```

As you can see in the example above, I ask the publisher to retry three times before replacing an error with an empty array.

#### Catch operator
Both retry and replace error operators are built on top of the *catch* operator. The *catch* operator allows us to replace a failed publisher with a new one. After that, the chain will use the new publisher to emit values.

```swift
init(service: GithubService) {
    $query
        .debounce(for: .seconds(0.5), scheduler: DispatchQueue.main)
        .flatMap {
            service
                .searchPublisher(matching: $0)
                .catch { error -> AnyPublisher<[Repo], Never> in
                    if error is URLError {
                        return Just([])
                            .eraseToAnyPublisher()
                    } else {
                        return Empty(completeImmediately: true)
                            .eraseToAnyPublisher()
                    }
            }
        }
        .subscribe(on: DispatchQueue.global())
        .receive(on: DispatchQueue.main)
        .assign(to: &$repos)
}
```

Another great thing about the *catch* operator is the opportunity to access the error and return a new publisher after inspecting the error.

#### Conclusion
The Combine is the framework that I use in all of my apps. It has a high performance and friendly *Swift API*. Sometimes it is hard to debug errors in the Combine publisher chains. I hope this post gave you more information on the publisher's lifecycle and explained to you how to catch the errors. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!