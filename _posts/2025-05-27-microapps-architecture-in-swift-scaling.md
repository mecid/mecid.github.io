---
title: Microapps architecture in Swift. Scaling.
layout: post
category: Architecture
image: /public/spm.jpg
---

The primary goals of the microapps architecture are to maintain separation of concerns to enhance compile time, adhere to the single responsibility principle, and facilitate continuous delivery, allowing for the deployment of a feature without the need for the completion of other features.

{% include friends.html %}

Swift Package Manager became the heart of this approach because it allows us to easily create Swift packages in Xcode and maintain them. As we discussed in the first post of the series, as soon as I create a new project in Xcode, I also create a Swift Package inside the project where I will place all the feature code and keep my app target as tiny as possible.

![microapps-package](/public/scale.png)

This approach worked great and served my indie apps for a long time. As always, it has both pros and cons. It is really simple to maintain a single package with a bunch of separate modules where you can clearly identify the dependency tree of the modules. Everything works fast and reliably as long as you have up to 20 packages.

> To learn more about the basics of the microapps architecture, take a look at my ["Microapps architecture in Swift. SPM basics."](/2022/01/12/microapps-architecture-in-swift-spm-basics/) post.

Let’s talk about large and extra-large apps where you can have more than 100 modules. In this case, a single package containing all the codebase is not a good solution because your developer experience might go down very fast.

Editing a single Package.swift file with hundreds of modules becomes a problem, and there are a bunch of reasons for that. First of all, Xcode can’t handle the dependency graph of such a huge package efficiently. Second, the Package.swift file becomes a real mess with thousands of lines of code that are really hard to navigate.

Another approach I can suggest for large and extra-large apps is to use a package-per-feature, where you still use different modules inside a feature package to divide it into layers like domain, UI, etc. In this case, you will have a collection of packages for the app, where every feature lives in a single package.

Whenever a feature A depends on a particular module of feature B, you can only connect that particular module. For example, you might have a Health package with a few modules for models, services, UI, etc. When you work on onboarding features, you might need a health authorization service, and this approach allows you to create a module in the Onboarding package that only depends on the service module of the Health package.

```swift
import PackageDescription

let package = Package(
    name: "FeatureA",
    products: [
        .library(name: "FeatureAModels", targets: ["Models"]),
    ],
    dependencies: [
        .package(path: "../FeatureB")
    ],
    targets: [
        .target(
            name: "Models",
            dependencies: [
                .product(name: "FeatureBModels", package: "FeatureB")
            ]
        ),

    ]
)
```

While a single-package approach works well for small to mid-sized projects, it can become a bottleneck as the number of modules grows. For large and extra-large apps, transitioning to a feature-per-package strategy provides better separation, improved dependency control, and a smoother developer experience.

As with any architecture, it’s important to balance simplicity and scalability, adapting your approach as the app evolves. Ultimately, the goal is to maintain a modular, testable, and continuously deliverable codebase — and this approach makes that possible, even at scale. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
