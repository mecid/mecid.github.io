---
title: Generating images in Swift using Image Playground
layout: post
category: Foundation Models
image: /public/fm.png
---

I’m continuing to work on AI-generated content in my apps, and this time, we’ll focus on image generation. You might be familiar with the Image Playground app on your Apple devices, which also has a Swift API. This week, we’ll explore how to utilize the Image Playground framework to create image content within our apps.

{% include friends.html %}

Image Playground framework provides us a text-to-image functionality. The core of the framework is the ImageCreator type. Let’s take a look at how we can use it.

```swift
import ImagePlayground

public struct Eye {
    public init() {}
    
    public func visualize(text: String) async throws -> CGImage? {
        let creator = try await ImageCreator()

        let images = creator.images(
            for: [.text(text)],
            style: .sketch,
            limit: 1
        )

        for try await image in images {
            return image.cgImage
        }

        return nil
    }
}
```

As you can see in the example above, we create an instance of the ImageCreator type. The initialize may throw an error if the running device doesn’t support image generation.

Next, we create the concepts describing the image we want to generate. Here you you can be creative and use a combination of text and source image if needed.

The final step is the call to the images function on instance of the ImageCreator type. It requires a few parameters: concepts, style and limit.

The ImageCreator type supports a set of styles: animation, illustration and sketch. You can choose the one you need. The limit parameter allows you to limit the number of results, in our example I ask only for a single image. Keep, in mind that system allows no more than 4 images.

The images function returns an instance of AsyncSequence which emits the result images one by one as soon as they become ready. That’s why we use here for loop to receive the images.

As I said before, we can mix and match multiple concepts to generate a single image. For example, you can provide a source image and some text.

```swift
public struct Eye {
    public init() {}
    
    public func visualize(text: String, image: CGImage) async throws -> CGImage? {
        let creator = try await ImageCreator()

        let images = creator.images(
            for: [.text(text), .image(image)],
            style: .sketch,
            limit: 1
        )

        for try await image in images {
            return image.cgImage
        }

        return nil
    }
}
```

The ImagePlaygroundConcept type provides us a few static functions allowing us to create a concept. We already use the text and image. 

There are also drawing and extracted functions. The drawing function allows us to provide an instance of the PKDrawing from the PencilKit framework.

The extracted function become useful when you have a huge text like article and you can use it to generate the image for an article.

Not all the styles might be available on your device. That’s why the ImageCreator type provides the static property called availableStyles. It is an array of the supported styles. You should always check if the selected style is available and use only available one.

```swift
public struct Eye {
    public init() {}
    
    public func visualize(text: String) async throws -> CGImage? {
        let creator = try await ImageCreator()

        guard let availableStyle = creator.availableStyles.first else {
            return nil
        }

        let style = !creator.availableStyles.contains(.animation) ? availableStyle : .animation

        let images = creator.images(
            for: [.text(text)],
            style: style,
            limit: 1
        )

        for try await image in images {
            return image.cgImage
        }

        return nil
    }
}
```

The Image Playground framework brings Apple’s generative image capabilities right into Swift, making it surprisingly simple to create visuals from text, drawings, or even existing photos. With just a few lines of code, you can generate styled images and integrate them directly into your app’s experience.
