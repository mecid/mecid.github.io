---
title: On-demand resources in iOS app 
layout: post
category: Architecture
image: /public/xcode-cloud.png
---

On-Demand Resources allow you to ship a smaller initial app download and fetch additional assets like images, sounds, level data, ML models, and more only when a user requires them. This week, we’ll explore how to utilize on-demand resources to store secrets outside of the app binary. 

{% include friends.html %}

iOS handles downloading, caching, and eviction, providing a seamless streaming experience without the need for your own asset CDN logic. Most apps use on-demand resources for large blobs like level data in games or ML models. But we can also leverage the power of on-demand resources to keep secrets outside of our binary.

For instance, we can fetch API tokens using on-demand resources and save them in the Keychain. This makes reverse engineering our app binary more challenging.

First, we need to enable them in the build settings of our app target. There’s a key called “Enable On Demand Resources” that should be set to YES. Once that’s done, we can start associating app resources with tags in the Resource Tags section of app target settings. This will allow us to fetch a specific collection of resources later on by using those tags.

There are three types of tags: initial install tags, prefetched tags, and download-only tags. Initial install tags are downloaded from the App Store along with the app binary. Prefetched tags are downloaded as soon as the app binary is downloaded. Download-only tags are downloaded only when you request them using an API.

```swift
final class OnDemandResource {
    private let request: NSBundleResourceRequest

    init(tags: Set<String>) {
        request = NSBundleResourceRequest(tags: tags)
    }

    func pin() async throws -> Bundle {
        let isFetched = await request.conditionallyBeginAccessingResources()

        if !isFetched {
            try await request.beginAccessingResources()
        }

        return request.bundle
    }

    func unpin() {
        request.endAccessingResources()
    }
}
```

Let’s create a type that we can use to access our on-demand resources. Here we define the *OnDemandResource* actor with two functions *pin* and *unpin*. The *pin* function initiates a resource request with the provided set of tags and returns a bundle that we can use to access our resources.

We use the *conditionallyBeginAccessingResources* function to check if we can access resources directly. If it returns false, we download them from the App Store using *beginAccessingResources*. If downloaded, it returns true, and we get the bundle to access resources almost immediately. As soon as we finish using resource we should call *unpin* to allow system evict resources.

```swift
let resource = OnDemandResource(tags: ["Config"])
let bundle = try await resource.pin()
defer { resource.unpin() }

if let config = bundle.url(forResource: "Config", withExtension: "json") {
    // decode your config and save to Keychain
}
```

On-Demand Resources are often associated with large assets, but as we’ve seen, they can also be a practical tool for improving the security posture of your iOS app. By moving sensitive data—such as API tokens—out of the main app binary and delivering them only when needed, you reduce the attack surface and make static analysis significantly harder. 

Apple mentioned that on-demand resources is a legacy technology, so migrating to Background Assets is recommended. That's going to be the topic for the next week. I hope you enjoyed this one. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask any questions related to this post. Thanks for reading, and see you next week!
