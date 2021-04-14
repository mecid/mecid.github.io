---
title: Must-have SwiftUI extensions
layout: post
image: /public/anyview.png
category: View Composition
---

Currently, I have three ongoing SwiftUI projects. During my work on these projects, I find myself in copying some extension files, which are very helpful in any SwiftUI based project.  That's why I decide to share with you that small foundation of useful extensions.

{% include friends.html %}

#### AnyView
*AnyView* is a type-erased view. It allows us to hide the real type of view inside the erased box. Usually, we might use it whenever we want to return different types of views. Let's take a look at a quick example.

```swift
import SwiftUI

struct Item: Identifiable {
    enum ItemType {
        case image
        case text
    }

    let id = UUID()
    let value: String
    let type: ItemType
}

struct ContentView: View {
    let items: [Item] = [
        Item(value: "text", type: .text),
        Item(value: "photo", type: .image)
    ]

    var body: some View {
        List(items) { item -> AnyView in
            if item.type == .image {
                return AnyView(Image(systemName: item.value))
            } else {
                return AnyView(Text(item.value))
            }
        }
    }
}
```

In the example above, we have a list that shows two types of items, and because of *@ViewBuilder*, we can return only the one kind of view. We use *AnyView* to erase our views to the single *AnyView* type. It looks like we are going to use *AnyView* a lot of times across the codebase because many views use *@ViewBuilder* to build its content. Let's introduce the extension on *View* type, which can erase it to *AnyView*.

```swift
extension View {
    func eraseToAnyView() -> AnyView {
        AnyView(self)
    }
}

struct ContentView: View {
    let items: [Item] = [
        Item(value: "text", type: .text),
        Item(value: "photo", type: .image)
    ]

    var body: some View {
        List(items) { item -> AnyView in
            if item.type == .text {
                return Text(item.value)
                    .eraseToAnyView()
            } else {
                return Image(systemName: item.value)
                    .eraseToAnyView()
            }
        }
    }
}
```

Here we use our new extension for erasing views. I think it looks much better, even with this simple example. But remember that *AnyView* has an impact on performance.

>AnyView allows changing the type of view used in a given view hierarchy. Whenever the type of view used with AnyView changes, SwiftUI destroys old hierarchy and creates a new hierarchy the new type.

To learn more about diffing process in SwiftUI take a look at my ["You have to change mindset to use SwiftUI"](/2019/11/19/you-have-to-change-mindset-to-use-swiftui/) post.
That's why it is better to avoid usage of *AnyView* and replace it with *Group* whenever it is possible. Let's see how we can replace *AnyView* with the *Group* component in the previous example.

```swift
struct ContentView: View {
    let items: [Item] = [
        Item(value: "text", type: .text),
        Item(value: "photo", type: .image)
    ]

    var body: some View {
        List(items) { item in
            Group {
                if item.type == .image {
                    Image(systemName: item.value)
                } else {
                    Text(item.value)
                }
            }
        }
    }
}
```

To learn more about hidden benefits of *Group* component take a look at ["View composition in SwiftUI"](/2019/10/30/view-composition-in-swiftui/) post.

#### IndexedCollection
Assume that you are working on a note-taking app, you have a store object where you keep all the notes, you have a notes list screen, and you have a note editing screen. Let's try to implement it.

```swift
import SwiftUI

final class Store: ObservableObject {
    var notes: [String] = ["first", "second"]
}

struct NotesView: View {
    @ObservedObject var store = Store()

    var body: some View {
        NavigationView {
            List(store.notes.indexed(), id: \.self) { note in
                // Compilation error here, EditView needs a binding, but we pass a String
                NavigationLink(destination: EditView(note: note)) {
                    Text(note)
                }
            }.navigationBarTitle("Notes")
        }
    }
}

struct EditView: View {
    @Binding var note: String

    var body: some View {
        TextField("type your note here", text: $note)
    }
}
```

Here is a straightforward implementation of the idea. It looks simple, but it doesn't compile. The problem here is that *EditView* needs a binding to the note to alter it, but instead, we pass a copy of the note. We can't provide both binding and item while using an array. Let's introduce the *IndexedCollection* type.

```swift
extension RandomAccessCollection {
    func indexed() -> Array<(offset: Int, element: Element)> {
        Array(enumerated())
    }
}
```

*IndexedCollection* is a wrapper for any *RandomAccessCollection*, and it provides both index and element via subscript. Now let's take a look at how we can use this collection type.

```swift
struct NotesView: View {
    @ObservedObject var store = Store()

    var body: some View {
        NavigationView {
            List(store.notes.indexed(), id: \.1.self) { index, note in
                NavigationLink(destination: EditView(note: self.$store.notes[index])) {
                    Text(note)
                }
            }.navigationBarTitle("Notes")
        }
    }
}
```

As you can see in the example above, we can get both index and element of the collection. It allows us to provide both Id for the list and binding for the *EditView*.

#### Bonus tip

```swift
extension View {
    func embedInNavigation() -> some View {
        NavigationView { self }
    }
}
```

#### Conclusion
Today we talked about the most useful SwiftUI extensions from my codebase. I hope you will find it valuable. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 