---
title: Building AI features using Foundation Models. Multimodal input.
layout: post
---

One of the most eagerly anticipated additions to Foundation Models was the ability to input images. I was almost certain that we would receive this feature during this WWDC, and fortunately, we did. This week, we will learn how to utilize the multimodal input capability in Foundation Models.

I’ve been working on a plate scanner app, a simple app where you capture your plate and it gives you some sort of food analysis, without numbers and calories. This app is a perfect fit for multimodal usage of Foundation Models.

Let’s start with the PlateClassification type, which is the definition of output that we want to get from the Foundation Model. We will use a well-known generable macro.

```swift
import FoundationModels

@Generable struct PlateClassification {
    @Guide(description: "The title of the meal")
    let title: String

    @Guide(description: "Inludes sugar")
    let sugar: Bool

    @Guide(description: "Includes protein")
    let protein: Bool

    @Guide(description: "Includes fiber")
    let fiber: Bool

    @Guide(description: "Includes saturated fat")
    let saturatedFat: Bool
}
```

As you can see, we defined the PlateClassification with a bunch of boolean properties and the title property. This is all we want to get as a result from the model. The next step is to provide the instructions and the image to the Foundation Model.

```swift
func classifyPlate(imageURL: URL) async throws -> PlateClassification {
    let session = LanguageModelSession()
    let result = try await session.respond(generating: PlateClassification.self) {
        "Classify the following meal and provide nutritional information"
        Attachment(imageURL: imageURL)
    }
    return result.content
}
```

Here we use the old LanguageModelSession type to instantiate a language model. Then we use a new overload of the respond function allowing us to build a prompt using the PromptBuilder result builder. It looks similar to the well-known ViewBuilder from SwiftUI.

The PromptBuilder type works with any type conforming to the PromptRepresentable protocol. And the framework provides us with a lot of conformances out of the box, like String, Array, Optionals, our Generable types, and the most important, a new Attachment type.

In our example, we use string instructions in pairs with instances of the Attachment type referring to our image via URL. But you can attach not only images, it can be any other file either.

The Attachment type provides us with a few ways to instantiate it using a URL, a pixel buffer, or in terms of an image, it can be an instance of CGImage or CIImage.

At the end we get an instance of the PlateClassification type as the result from the model and can use it in the app to display nutritional information.

Multimodal input makes Foundation Models much more useful for building real-world features. We are no longer limited to describing the world to the model with text — we can simply give it an image or another file and ask it to reason about its contents.
