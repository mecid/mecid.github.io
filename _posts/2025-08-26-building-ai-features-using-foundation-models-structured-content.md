---
title: Building AI features using Foundation Models. Structured Content.
layout: post
category: Foundation Models
image: /public/fm.png
---

Last week, we talked about the basics of Foundation Models, how to generate text content, and how to tune and control the output. This week, we will talk about simple and yet powerful structured content generation.

{% include friends.html %}

Generating text content using Foundation Models is easy-peasy. You can use it for many things like coaching, smart assistants, chats, etc.

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

Structured content generation works pretty much the same way; the only addition is the structure you provide that the Foundation Model should infer and return the result in the very same structure.

```swift
struct Intelligence {
    private var recipeInstructions: String {
    """
    You are a professional nutritionist and chef specializing in healthy meal planning. Generate a recipe using provided ingredients.
    """
    
    func generateRecipe(with ingredients: String) async throws -> Recipe {
        let session = LanguageModelSession(instructions: recipeInstructions)
        let recipe = try await session.respond(to: prompt, generating: Recipe.self)
        return recipe.content
    }
}
```

As you can see in the example above, we use the *respond* function with the *prompt* and an additional parameter called *generating*. The *generating* parameter receives the type that Foundation Model should infer and return the instance. Now, let’s take a look at the *Recipe* type.

```swift
@Generable
struct Recipe {
    @Guide(description: "Shortened title for items, without numbers.")
    let title: String

    @Guide(description: "Total energy in kilocalories.")
    let energy: Double

    @Guide(description: "Total amount of protein in grams.")
    let protein: Double

    @Guide(description: "Total amount of carbohydrates in grams.")
    let carbs: Double

    @Guide(description: "Total amount of fiber in grams.")
    let fiber: Double

    @Guide(description: "Total amount of sugar in grams.")
    let sugar: Double

    @Guide(description: "Total amount of fat in grams.")
    let fatTotal: Double

    @Guide(description: "Total amount of saturated fat in grams.")
    let fatSaturated: Double

    @Guide(description: "Total weight of items in grams.")
    let servingSize: Double

    @Guide(description: "Ingredients, separated by commas.")
    let ingredients: String?

    @Guide(description: "Instructions, separated by newlines.")
    let steps: String?
}
```

In order to receive response in a structured format, you have to conform you response type to the *Generable* protocol. Swift simplifies this task by introducing the *Generable* macro that we use to annotate our type.

As you can see, the Recipe type is a simple Swift struct that uses the *Guide* macro. The *Guide* macro provides additional information to the Foundation Model, so it can understand how to fill that field. The *Guide* macro receives the description in the natural language similar to prompt. You can be very specific while guiding the Foundation Model to make the response nice.

Structured content generation takes the power of Foundation Models one step further by making the results predictable, reusable, and developer-friendly. With just a few annotations and the right guidance, you can transform raw AI output into strongly-typed Swift models that fit seamlessly into your app. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!
