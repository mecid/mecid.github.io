---
title: Building AI features using Foundation Models. Structured Content.
layout: post
---

Last week, we talked about the basics of Foundation Models, how to generate text content, and how to tune and control the output. This week, we will talk about simple and yet powerful structured content generation.

Generating text content using Foundation Models is easy-peasy. You can use it for many things like coaching, smart assistants, chats, etc.

======================================================

Structured content generation works pretty much the same way; the only addition is the structure you provide that the Foundation Model should infer and return the result in the very same structure.

======================================================

As you can see in the example above, we use the respond function with the prompt and an additional parameter called generating. The generating parameter receives the type that Foundation Model should infer and return the instance. Now, let’s take a look at the Recipe type.

======================================================

In order to receive response in a structured format, you have to conform you response type to the Generable protocol. Swift simplifies this task by introducing the Generable macro that we use to annotate our type.

As you can see, the Recipe type is a simple Swift struct that uses the Guide macro. The Guide macro provides additional information to the Foundation Model, so it can understand how to fill that field. The Guide macro receives the description in the natural language similar to prompt. You can be very specific while guiding the Foundation Model to make the response nice.

Structured content generation takes the power of Foundation Models one step further by making the results predictable, reusable, and developer-friendly. With just a few annotations and the right guidance, you can transform raw AI output into strongly-typed Swift models that fit seamlessly into your app.
