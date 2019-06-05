---
title: Making real-world app with SwiftUI
layout: post
---

That is WWDC week, and everybody so excited about so many new things we have this year like SwiftUI, Dark Mode, Updateable CoreML models, etc. I will try to cover all the new stuff during the upcoming weeks. Let's start with *SwiftUI*. *SwiftUI* is an entirely new approach to building apps for the Apple ecosystem.

*SwiftUI* is a declarative Component-Oriented framework. You have to forget about MVC where you have controllers mediating between view and model. All we have in *SwiftUI* is a state and view derived from the state. As soon as your state changes *SwiftUI* rebuild UI for that state changes. Apple team did an excellent job by providing so beautiful [tutorial page](https://developer.apple.com/tutorials/swiftui/) for *SwiftUI*. It covers a lot of stuff like Layout, Interfacing with UIKit, etc. 

I will try to show you real-world app example written entirely in *SwiftUI*. Let's build an app searching for Github repos. We need a screen with a text field for typing a query and a list of repos which comes from the search query. I assume that you have already read the *SwiftUI* documentation because I'm not going to describe basics, I will try to show something that documentation didn't covers.

Let's start by building the *GithubService* class, which creates a search request and Repo struct, which represents a Github repository.

```swift
struct Repo: Decodable, Identifiable {
    var id: Int
    let name: String
    let description: String
}

struct SearchResponse: Decodable {
    let items: [Repo]
}

class GithubService {
    private let session: URLSession
    private let decoder: JSONDecoder

    init(session: URLSession = .shared, decoder: JSONDecoder = .init()) {
        self.session = session
        self.decoder = decoder
    }

    func search(matching query: String, handler: @escaping (Result<[Repo], Error>) -> Void) {
        guard
            var urlComponents = URLComponents(string: "https://api.github.com/search/repositories")
            else { preconditionFailure("Can't create url components...") }

        urlComponents.queryItems = [
            URLQueryItem(name: "q", value: query)
        ]

        guard
            let url = urlComponents.url
            else { preconditionFailure("Can't create url from url components...") }

        session.dataTask(with: url) { [weak self] data, _, error in
            if let error = error {
                handler(.failure(error))
            } else {
                do {
                    let data = data ?? Data()
                    let response = try self?.decoder.decode(SearchResponse.self, from: data)
                    handler(.success(response?.items ?? []))
                } catch {
                    handler(.failure(error))
                }
            }
        }.resume()
    }
}
```

Our *Repo* struct has only a few fields, but it is enough for our sample. If we want to use our *Repo* struct as the model which should be used by *SwiftUI* to build a View it has to conform to *Identifiable* protocol. The only requirement of *Identifiable* protocol is *id* property, which has to be a *Hashable* value.

Now we can start implementing view which represents *Repo* row in our list of repos. We will use a vertical stack with two text labels.

```swift
struct RepoRow: View {
    let repo: Repo

    var body: some View {
        VStack(alignment: .leading) {
            Text(repo.name)
                .font(.headline)
            Text(repo.description)
                .font(.subheadline)
        }
    }
}
```

Let's move to our SearchView which describes entire the screen.

```swift
struct SearchView : View {
    @State private var query: String = "Swift"
    @EnvironmentObject var repoStore: ReposStore

    var body: some View {
        NavigationView {
            List {
                TextField($query, placeholder: Text("type something..."), onCommit: fetch)
                ForEach(repoStore.repos) { repo in
                    RepoRow(repo: repo)
                }
            }.navigationBarTitle(Text("Search"))
        }.onAppear(perform: fetch)
    }

    private func fetch() {
        repoStore.fetch(matching: query)
    }
}
```

Here we have a query field which is marked as *@State*. It means that this view is derived from this state, and as soon as state changes, *SwiftUI* rebuilds the view. *SwiftUI* stores all the fields marked as *@State* in special separated memory, where only corresponded views can access and update them. *@State* is a new Swift feature called Property Wrapper, more about this feature you can read in [the proposal](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-delegates.md). The exciting aspect is the usage of *$query*, It means to get a reference for property wrapper, not value itself. We use it to connect *TextField* and *query* variable in two way binding.

Another interesting fact here is *@EnvironmentObject*. It is a part of feature called *Environment*. You can populate your *Environment* with all needed service classes and then access them from any view inside that *Environment*. The *Environment* is the right way of Dependency Injection with *SwiftUI*.

```swift
class ReposStore: BindableObject {
    var repos: [Repo] = [] {
        didSet {
            didChange.send(repos)
        }
    }

    var didChange = PassthroughSubject<[Repo], Never>()

    let service: GithubService
    init(service: GithubService) {
        self.service = service
    }

    func fetch(matching query: String) {
        service.search(matching: query) { [weak self] result in
            DispatchQueue.main.async {
                switch result {
                case .success(let repos): self?.repos = repos
                case .failure: self?.repos = []
                }
            }
        }
    }
}
```

*ReposStore* class should conform *BindableObject* protocol, which requires a *didChange* property. It makes possible to use it inside *Environment* and rebuild view as soon as it changes. The *didChange* property should be a *Publisher*, which is a part of a new Apple's Reactive framework called *Combine*. The main goal of *Publisher* is to notify all subscribers when something changes. That's why in *didSet* of our repos array we tell to our subscribers that data changed. As soon as new values appear, *SwiftUI* will rebuild *ReposView*.

The main difference between *@State* and *@EnvironmentObject* is that @State is accessible only to a particular view, in opposite *@EnvironmentObject* available for every view inside the Environment. But both of them used by SwiftUI to track changes and rebuild views as soon as changes appear.

#### Conclusion
This week we talked about an entirely new approach in the iOS development world. I will try to cover more WWDC topics in the upcoming weeks. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!