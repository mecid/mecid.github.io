---
title: PhotoPicker in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/photokit.jpeg
---

Nowadays, many frameworks Xcode provides us contain SwiftUI views, including the PhotosUI framework. The PhotosUI framework provides the *PhotoPicker* button, allowing us to offer photo-picking functionality in our apps quickly. This week we will learn how use the *PhotoPicker* view in SwiftUI.

{% include friends.html %}

The *PhotoPicker* view is a simple SwiftUI button handling the photo-picking experience for us for free. It is straightforward to use and provides a nice API. Let's take a look at a simple example.

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(selection: $selectedPhoto) {
                    Image(systemName: "pencil")
                }
            }
        }
    }
}
```

First, we must import the PhotosUI framework to use the *PhotoPicker* view. As you can see in the example above, we use an instance of the *PhotoPicker* view in the toolbar. The *PhotoPicker* button has only one required parameter: the selection binding. It should be binding to the *PhotosPickerItem* type. 

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(
                    selection: $selectedPhoto,
                    matching: .images
                ) {
                    Image(systemName: "pencil")
                }
            }
        }
    }
}
```

We can also limit the type of the photos user allowed to pick by using the *matching* parameter, an instance of the *PHPickerFilter* type. The *PHPickerFilter* provides predefined filters like images, screenshots, live photos, etc. You can also combine filters by using *any*, *all*, and *not* functions.

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(
                    selection: $selectedPhoto,
                    matching: .all(of: [.screenshots, .panoramas])
                ) {
                    Image(systemName: "pencil")
                }
            }
        }
    }
}
```

Remember, you can pick not only images but also video content by using the *PhotoPicker* view.

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(
                    selection: $selectedPhoto,
                    matching: .videos
                ) {
                    Image(systemName: "pencil")
                }
            }
        }
    }
}
```

You can also provide a binding to an array of the *PhotoPickerItem* type to enable multiple selections. In this case, you can use the *maxSelectionCount* parameter to control the maximal number of allowed items.

```swift
struct ContentView: View {
    @State private var selectedPhotos: [PhotosPickerItem] = []
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(selection: $selectedPhoto, maxSelectionCount: 3) {
                    Image(systemName: "pencil")
                }
            }
        }
    }
}
```

OK, we know how to provide a photo-picking experience, but what next? How can we load the selected images? The *PhotosPickerItem* type contains the whole image and video loading logic. It provides the *loadTransferable* function allowing us to load any type conforming to the *Transferable* protocol.

> To learn more about the basics of the *Transferable* protocol, take a look at my ["Sharing content in SwiftUI"](/2023/03/28/sharing-content-in-swiftui/) post.

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(
                    selection: $selectedPhoto,
                    matching: .all(of: [.screenshots, .panoramas])
                ) {
                    Image(systemName: "pencil")
                }
            }
            .task(id: selectedPhoto) {
                image = try? await selectedPhoto?.loadTransferable(type: Image.self)
            }
        }
    }
}
```

As you can see in the example above, we use the *loadTransferable* function to decode the image by using the new Swift Concurrency feature. The *PhotoPickerItem* type also provides a callback-based alternative if you don't use Swift Concurrency.

```swift
struct ContentView: View {
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var image: Image?
    
    var body: some View {
        NavigationStack {
            ZStack {
                image?
                    .resizable()
                    .scaledToFit()
            }
            .toolbar {
                PhotosPicker(
                    selection: $selectedPhoto,
                    matching: .all(of: [.screenshots, .panoramas])
                ) {
                    Image(systemName: "pencil")
                }
            }
            .task(id: selectedPhoto) {
                if selectedPhoto?.supportedContentTypes.first == .image {
                    image = try? await selectedPhoto?.loadTransferable(type: Image.self)
                }
            }
        }
    }
}
```

The *PhotoPickerItem* type also has the *supportedContentTypes* property defining the supported content type of the selected image or video item, allowing us to understand how to decode the item.

Today we learned how to use the *PhotoPicker* view in SwiftUI to implement the image-picking functionality in our app. I enjoy the API it provides and believe it is an excellent example of SwiftUI and Swift Concurrency feature usage. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
