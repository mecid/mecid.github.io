---
title: ViewModifiers in SwiftUI
layout: post
category: View Composition
---

*ViewModifiers* play a significant role in SwiftUI. Most of the functions called on a *View* are *ViewModifiers*. It is the primary way of modifying the view instance in SwiftUI. In this post, we will take a look at some ready to use *ViewModifiers*, and then we will build our own custom *ViewModifier*.

{% include friends.html %}

#### Custom ViewModifier
Assume that we are working on Github repositories search app, and we need some *View* to display a single repository in the search results screen. Here we go.

```swift
struct RepoRow: View {
    let repo: Repo

    var body: some View {
        HStack(alignment: .top) {
            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                Text(repo.description ?? "")
                    .foregroundColor(.secondary)
                    .font(.subheadline)
            }
        }
    }
}
```

In the example above, we use *ViewModifiers* like *foregroundColor* and *font*. I apply these two modifiers together very often to any subheadline text. Let's create custom ViewModifier which combines *foregroundColor* and *font* together.

```swift
import SwiftUI

struct SubheadlineModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .foregroundColor(.secondary)
            .font(.subheadline)
    }
}
```

We can create custom *ViewModifiers* by creating a struct conforming to *ViewModifier* protocol. The single requirement of *ViewModifier* protocol is body function. It looks very similar to View protocol, but instead, it accepts an original view as a function parameter and returns a modified view. Now, let's see how we can use newly created modifier in the code.

```swift
import SwiftUI

struct RepoRow: View {
    let repo: Repo

    var body: some View {
        HStack(alignment: .top) {
            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                ModifiedContent(
                    content: Text(repo.description ?? ""),
                    modifier: SubheadlineModifier()
                )
            }
        }
    }
}
```
As you can see in the example, we use *SubheadlineModifier* by creating *ModifiedContent* struct with original *View* and *SubheadlineModifier* instance as parameters.

#### ViewModifiers can have a @State like Views
Another interesting fact about *ViewModifiers* is that it conforms to View protocol. It means you can use inside *ViewModifiers* property wrappers like *@State, @Binding, @Environment, @ObservableObject and @EnvironmentObject*. To learn more about property wrappers provided by SwiftUI, take a look at ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/).

Swift has a lot of great libraries for loading and caching remote images. Let's integrate one of them with SwiftUI. Most of my projects use *Kingfisher* library for loading and caching remote images, but it doesn't support SwiftUI for now. We will try to integrate it by creating *ViewModifier* which loads remote images with *Kingfisher*.

```swift
import class Kingfisher.KingfisherManager
import SwiftUI

extension Image {
    func fetchingRemoteImage(from url: URL) -> some View {
        ModifiedContent(content: self, modifier: RemoteImageModifier(url: url))
    }
}

struct RemoteImageModifier: ViewModifier {
    let url: URL
    @State private var fetchedImage: UIImage? = nil

    func body(content: Content) -> some View {
        if let image = fetchedImage {
            return Image(uiImage: image)
                .resizable()
                .eraseToAnyView()
        }

        return content
            .onAppear(perform: fetch)
            .eraseToAnyView()
    }

    private func fetch() {
        KingfisherManager.shared.retrieveImage(with: url) { result in
            self.fetchedImage = try? result.get().image
        }
    }
}

extension View {
    func eraseToAnyView() -> AnyView {
        return AnyView(self)
    }
}
```

As you can see in the example above, we use @*State* property wrapper inside *RemoteImageModifier*. It creates an opportunity to reload the *View* as soon as we set something to *fetchedImage* property. We also create here some utility methods to simplify the usage of new *RemoteImageModifier*. Now we can easily use new *ViewModifier* with any *Image* to load remote images.

```swift
import SwiftUI

struct RepoRow: View {
    let repo: Repo

    var body: some View {
        HStack(alignment: .top) {
            Image(systemName: "photo") // placeholder
                .fetchingRemoteImage(from: repo.owner.avatar)
                .frame(width: 44, height: 44)
            VStack(alignment: .leading) {
                Text(repo.name)
                    .font(.headline)
                Text(repo.description ?? "")
                    .font(.subheadline)
            }
        }
    }
}
```

#### Conclusion
Today we talked about another key concept of SwiftUI. *ViewModifiers* allow us to encapsulate and reuse any logic across the *Views*. It is a compelling concept which we can use to build composable pieces of our apps. Try to use this method and share with me your thoughts about it. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 

