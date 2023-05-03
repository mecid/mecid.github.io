---
title: The power of overlays in SwiftUI
layout: post
image: /public/layout.png
category: View Composition
---

An overlay is a view drawing on top of another view. And today, we will talk about two interesting use cases of using overlays in SwiftUI. One of them allows us to keep the structural identity of the view, and another one becomes very handy whenever you build custom navigation transitions.

#### Keeping structural identity with overlays
Structural identity is the type of identity that SwiftUI uses to understand your views without an explicit identifier by using your layout description. It is essential to keep your view hierarchy without unnecessary branches that you may create using **if** statements in the body of a *ViewBuilder* closure because it may hurt the performance of your views and produce state losses.

```swift
struct ContentView: View {
    @State private var isDownloading = false
    
    var body: some View {
        if isDownloading {
            ProgressView()
        } else {
            DownloadButton("Download") {
                isDownloading = true
                // start download here
                isDownloading = false
            }
        }
    }
}
```

In the example above, whenever the *isDownloading* property changes, the framework creates a new button or new progress view. In the case of our custom button, it completely loses its state because SwiftUI makes a new one. This behavior can be unexpected in different scenarios, so avoid branching using **if** statements in *ViewBuilder* closures as much as possible.

> Structural identity link

Instead of branching via **if** statements, we can use overlays to keep the structural identity of the view.

```swift
struct ContentView: View {
    @State private var isDownloading = false
    
    var body: some View {
        DownloadButton() {
            isDownloading = true
            // start download here
            isDownloading = false
        }
        .disabled(isDownloading)
        .overlay {
            if isDownloading {
                ProgressView()
            }
        }
    }
}
```

As you can see in the example above, we use the *overlay* view modifier to display the progress view on the top button and disable it by using one of the inert view modifiers.

#### Custom transitions with overlays
From the very first day, the SwiftUI framework shows us how easily we can animate changes in the view hierarchy. Working with SwiftUI to build fluid animations is super easy. The only downside I can find is the custom navigation transitions. Unfortunately, there is no way to customize navigation transitions inside the *NavigationView* or *NavigationStack*.

One of the powerful animation tools of the SwiftUI framework, the *matchedGeomerty* view modifier, doesn't support *NavigationView* and *NavigationStack* at the very moment. Fortunately, we can use overlay view modifiers to build custom navigation transitions without using *NavigationView* or *NavigationStack*.

```swift
struct ContentView: View {
    @State private var selectedImage: String?
    @Namespace private var hero

    let images: [String] = [
        "pencil",
        "trash",
        "lock.doc",
        "person",
        "figure.run"
    ]

    var body: some View {
        NavigationStack {
            LazyVGrid(columns: Array(repeating: .init(.flexible()), count: 3)) {
                ForEach(images, id: \.self) { image in
                    Image(systemName: selectedImage == image ? "" : image)
                        .resizable()
                        .scaledToFit()
                        .background(Material.regular)
                        .matchedGeometryEffect(id: image, in: hero)
                        .onTapGesture {
                            selectedImage = image
                        }
                }
            }
            .overlay {
                if let image = selectedImage {
                    Image(systemName: image)
                        .resizable()
                        .scaledToFill()
                        .background(Material.thin)
                        .matchedGeometryEffect(id: image, in: hero)
                        .animation(.easeInOut, value: selectedImage)
                        .onTapGesture {
                            selectedImage = nil
                        }
                }
            }
        }
        .animation(.default, value: selectedImage)
    }
}
```

Here is another example where the overlay trick shines. Instead of *NavigationLink*, we pair the overlay view modifier with the *matchedGeomerty* view modifier. This pair allows us to build super custom navigation transitions like hero animation. Yes, it adds some work to maintain the navigation state, but it will enable us to provide an excellent user experience in our apps.

> Matched geometry link

Today we learned how valuable is the *overlay* view modifier in SwiftUI, and with the latest addition allowing us to build overlays by using the *ViewBuilder* closure, it became effortless.

