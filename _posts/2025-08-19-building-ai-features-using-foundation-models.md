---
title: Building AI features using Foundation Models
layout: post
category: Foundation Models
image: /public/fm.png
---

Apple introduced the brand-new Foundational Models framework, providing type-safe APIs for using Apple Intelligence models in your apps. This week, we will learn how to use this new framework while building AI features in your apps.

{% include friends.html %}

First of all, we should import the Foundation Models framework and check the availability on the device. Not every device supports Apple Intelligence features. So we should keep it in mind while building AI features.

```swift
import FoundationModels

struct Intelligence {
    public func generate(_ input: String) async -> String {
        guard SystemLanguageModel.default.isAvailable else {
            return input
        }
        
        // ...
    }
}
```

Here we use the *SystemLanguageModel* type representing a text-only language model available on Apple platforms. We access the default instance of the model and use the *isAvailable* property.

Sometimes, it is not enough to check the *isAvailable* property and you want to know exact the reason. For example, user can keep Apple Intelligence disabled and that’s why the *SystemLanguageModel* is not available. You can display an alert asking the user to turn on Apple Intelligence to access particular features.

For this case, the *SystemLanguageModel* type provides us the *availability* property which is the instance of the *SystemLanguageModel.Availability* type. You can use this property for more granular experience in your app. Now we can initiate language model session and chat with it.

```swift
import FoundationModels

struct Intelligence {
    public func generate(_ input: String) async throws -> String {
        guard SystemLanguageModel.default.isAvailable else {
            return input
        }
        
        let session = LanguageModelSession()
        
        let response = try await session.respond(to: input)
        return response.content
    }
}
```

As you can see in the example above, we initiate a session using the *LanguageModelSession* type and call the *respond* function to get the text content returned by the model. That’s so easy!

We are not going to build a chat, instead we will try to generate some recommendations using AI. In this case, we should provide some instructions to the model describing what we need from the model in the natural language. The prompt in this case, is a secondary input. For example, in my app I analyze food nutrients and try to give some advices.

```swift
import FoundationModels

struct Intelligence {
    private let instructions = """
        You are a healthy lifestyle coach. Write a recommendation. Keep it short, positive and inspiring. Respond only with the final recommendation, no explanations.
    """
    
    public func generate(_ input: String) async throws -> String {
        guard SystemLanguageModel.default.isAvailable else {
            return input
        }
        
        let session = LanguageModelSession(instructions: instructions)
        
        let response = try await session.respond(to: input)
        return response.content
    }
}
```

In the example above, I define the *Intelligence* struct holding the *generate* function. This function takes the input and apply the instructions we passed to the model. For example, you can pass the “Reduce carbs” string to the generate function and it will write a nice recommendation for you.

Now let’s talk about tuning the response of the model. The *respond* function allows us to pass an instance of the *GenerationOptions* type using the *options* parameter. The *GenerationOptions* type defines a few interesting properties.

```swift
import FoundationModels

struct Intelligence {
    private let instructions = """
        You are a healthy lifestyle coach. Write a recommendation. Keep it short, positive and inspiring. Respond only with the final recommendation, no explanations.
    """
    
    public func generate(_ input: String) async throws -> String {
        guard SystemLanguageModel.default.isAvailable else {
            return input
        }
        
        let session = LanguageModelSession(instructions: instructions)
        
        let seed = UInt64(Calendar.current.component(.dayOfYear, from: .now))
        let sampling = GenerationOptions.SamplingMode.random(top: 10, seed: seed)
        let options = GenerationOptions(sampling: sampling, temperature: 0.7)
        
        let response = try await session.respond(to: input, options: options)
        return response.content
    }
}
```

The *temperature* property allows us to control the creativity of the model. It is a numeric value from 0 to 1. There is no best value for this parameter and you should experiment to find which value fits your case.

The *sampling* parameter allows us to control how the model chooses the final answer from multiple possible answers. For example, whenever you set it to *greedy* it always return the same answer for the same prompt. It might be useful for debugging purposes.

In my app, I don’t want to display different suggestions to the user on every launch of the app. Instead, I would like to display a unique recommendation on daily basis. So, whenever day changes it might use another recommendation for the same case. 

This is how the random sampling works. You fix the number of available responses and provide a stable seed. As soon as, seed changes the response changes, but it returns the same response for the same prompt while the seed is the same.

Building AI features with Apple’s new Foundation Models framework feels natural and developer-friendly. Whether you’re generating personalized recommendations, enabling intelligent workflows, or building entirely new features, this framework provides the foundation for integrating Apple Intelligence into your apps in a reliable way. The key is to experiment, tune the options for your use case. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!
