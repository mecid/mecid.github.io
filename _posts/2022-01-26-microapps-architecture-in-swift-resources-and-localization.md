---
title: Microapps architecture in Swift. Resources and localization.
layout: post
category: Architecture
image: /public/xcode-spm.png
---

This week we will continue the topic of microapps architecture by touching on another essential edge of this approach. In this post, we will talk about sharing resources between modules and separating the localization of feature modules.

#### Bundle API
Usually, we don't care much about resource bundles in single module apps because the only bundle is the app target bundle. Let's take a quick look at the example using Bundle API.

```swift
Image("image")
Image("image", bundle: nil)
Image("image", bundle: .main)
```

These three lines do the same thing. They look for the image in the bundle of the running target. We can try to access an image from a particular bundle. Here is another example.

```swift
Image("image", bundle: Bundle(identifier: "com.example.myapp"))
```

#### Swift Package Manager and Resources
Swift Package Manager allows us to put code and related resources in Swift modules. In this case, the Swift compiler generates a bundle for every module that contains resources. Imagine our design system module containing an asset catalog with custom icons to reuse in feature modules.

To achieve that, first, we need to create an asset catalog with icons and place it inside the DesignSystem folder. Xcode automatically recognizes the format of the asset catalog and generates a bundle for the module. You can access the bundle of the current module by using the module property on the Bundle type. But remember, Xcode generates it only for modules with resources inside. 

```swift
public struct ImageFactory {
    public enum Icon: String {
        case trash
        case star
        case plus
    }

    public static func get(_ icon: Icon) -> UIImage {
        UIImage(
            named: icon.rawValue,
            in: Bundle.module,
            compatibleWith: .current
        )!
    }
}
```

As you can see in the example above, we create a type-safe image factory that loads icons from the design system bundle. Please look at how we use the module property on the Bundle type. This is how we can load the resources from the bundle of the current module. If we omit the bundle parameter, the app will load the image from the app target module, which doesn't contain these images.

Xcode automatically generates bundles for asset catalogs, Core Data models, Storyboards, NIBs, and localization files. For any other types of resources, you should use Package.swift file to ask the Swift compiler to bundle them.

```swift
// Package.swift
let package = Package(
    ...
    targets: [
        .target(
            name: "DesignSystem",
            dependencies: [],
            resources: [
                .copy("Resources/Fonts"),
                .process("Resources/PNGs")
            ]
        )
    ]
)
```

The design system module contains the Resources folder with fonts and different PNGs that I want to bundle with my module. Unfortunately, Xcode doesn't recognize them automatically, but we can bundle them manually via a few lines of code in the Package.swift file. There are two ways of bundling resources in Swift modules. We can process or copy them directly in the bundle.

When you choose processing resources, Xcode applies platform-specific rules for compressing files, when you select copying, Xcode copies files simply by keeping the structure of the sub-directories.

#### Swift Package Manager and Localization
Swift Packages support localization out of the box. Every feature module should contain its localization file and be ready to build and run as a separate microapp. We can quickly achieve that by enabling localization in the Package.swift file by adding a single line of code.

```swift
// Package.swift
let package = Package(
    name: "MyAppLibrary",
    defaultLocalization: "en",
    platforms: [.iOS(.v15)],
    ...
)
```

We added the defaultLocalization parameter to the package declaration in the example above. This parameter is necessary if you want to support localization. Now you can create en.lproj, es.lproj, or any-locale-identifier.lproj folders in your module to place your Localizable.strings file with particular translations. Remember that you still need to specify the bundle whenever you want to access the localization of the current module.

```swift
// Feature module
Text("helloMessage", bundle: .module)
```

We should learn from SwiftUI views how to leverage the power of modules while modeling the public interfaces of our custom views. For example, the design system module contains a generic view that I use as an empty or error view. I will use it in various feature modules with different localizations coming from different bundles.

```swift
// DesignSystem module
public struct PlaceholderView: View {
    let systemImage: String
    let text: LocalizedStringKey
    let bundle: Bundle?

    public init(
        _ text: LocalizedStringKey,
        bundle: Bundle? = nil,
        systemImage: String
    ) {
        self.systemImage = systemImage
        self.bundle = bundle
        self.text = text
    }

    public var body: some View {
        Label {
            Text(text, bundle: bundle)
                .multilineTextAlignment(.center)
        } icon: {
            Image(systemName: systemImage)
                .imageScale(.large)
                .font(.largeTitle)
        }
        .labelStyle(.vertical)
        .padding(.horizontal)
        .padding(.horizontal)
    }
}
```

As you can see in the example above, we define a public init that accepts the module to retrieve localization. Without this parameter, the view can access only the main or its bundle. But we need to access bundles of various feature modules. That's why we have to introduce an optional bundle parameter.

```swift
// Search feature module
PlaceholderView(
    "emptySearchPlaceholder",
    bundle: .module,
    systemImage: "magnifyingglass"
)
```

#### Conclusion
This week we learned how to use resources within microapps architecture. We also learned how to encapsulate localization inside the feature modules. There are two rules which I use to build well-separated modules.

First, I try to encapsulate the resources inside the module that uses them. Feature modules contain the localization strings they need. Second, create a type-safe public API for retrieving shared resources, as we do in the design system module, to get the icons across different feature modules.

 I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

#### References
1. [Meet the microapps architecture](https://increment.com/mobile/microapps-architecture/)
2. [Introduction to App Modularisation with Swift Package Manager](https://holyswift.app/introduction-to-app-modularisation-with-swift-package-manager-a-tale-to-be-told)
