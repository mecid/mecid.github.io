---
title: On-demand resources in iOS app 
layout: post
---

On-Demand Resources allow you to ship a smaller initial app download and fetch additional assets like images, sounds, level data, ML models, and more only when a user requires them. This week, we’ll explore how to utilize on-demand resources to store secrets outside of the app binary. 

iOS handles downloading, caching, and eviction, providing a seamless streaming experience without the need for your own asset CDN logic. Most of the apps uses on-demand resources for large blobs like level data in games or ML models. But we can also leverage the power of on-demand resources to keep secrets outside of our binary.

For instance, we can fetch API tokens using on-demand resources and save them in the keychain. This makes reverse engineering our app binary more challenging.

First, we need to enable them in the build settings of our app target. There’s a key called “Enable On Demand Resources” that should be set to YES. Once that’s done, we can start associating app resources with tags. This will allow us to fetch a specific collection of resources later on by using those tags.

There are three types of tags: initial install tags, prefetched tags, and download-only tags. Initial install tags are downloaded from the App Store along with the app binary. Prefetched tags are downloaded as soon as the app binary is downloaded. Download-only tags are downloaded only when you request them using an API.

======================================================

Let’s create a type that we can use to access our on-demand resources. Here we define the OnDemandResourcesService struct with the single access function. The access function initiate a resource request with the provided set of tags and returns a bundle that we can use to access our resources.

We use the conditionallyBeginAccessingResources function to check if we can access resources directly. If it returns false, we download them from the App Store using beginAccessingResources. If downloaded, it returns true, and we get the bundle to access resources almost immediately.

======================================================

On-Demand Resources are often associated with large assets, but as we’ve seen, they can also be a practical tool for improving the security posture of your iOS app. By moving sensitive data—such as API tokens—out of the main app binary and delivering them only when needed, you reduce the attack surface and make static analysis significantly harder. I hope you enjoyed this one. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask any questions related to this post. Thanks for reading, and see you next week!
