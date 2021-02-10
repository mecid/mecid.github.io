---
title: Building type-safe networking in Swift
layout: post
image: /public/networking.jpg
---

More than half of the apps I built during my career have networking code. Usually, we build apps for web services. I build CardioBot and NapBot apps that do all the logic on the user's device and don't need an internet connection. But this is just a small part of the real world that needs a networking code. Today we will talk about building the type-safe networking layer in Swift for iOS apps.

#### Basics
Let's first take a look at the typical networking code and recognize the issues that we want to avoid in our solution.

=====================================================

In the example above, we have a standard networking code that creates the request and decodes the response. There are a few things that I want to avoid in the future.
We create a GET request, but there is a possibility to set an HTTP body for a GET request, which looks meaningless.
We try to decode the response by providing the type of resulting data. Usually, every request has one and only one type that we can obtain. In this case, it is better to have the response type encoded into the request.

Let's start building our type-safe networking by introducing the Request type. The Request type should contain the URL we need to access, headers, and HTTP method.

=====================================================

The HTTP method is an exclusive thing. You can't use both GET and POST, and you can choose only one HTTP method. It looks like a perfect use-case for an enum type.

=====================================================

As you can see in the example above, we define the HTTPMethod enum that describes various HTTP methods. We use cases with associated types to hold correlated with the HTTP method. For example, the GET method contains URL query items, POST and PUT methods have the data we use as the HTTP body.

Now we need to make somehow URLSession working with our Request type. The easiest way is defining a calculated property on the Request type that converts it to the URLRequest.

=====================================================

Finally, we can create an extension on URLSession to make requests with our new type.

=====================================================

#### Phantom response type
One thing we forgot to do is encoding of result type into the request. We can make it really easy by introducing phantom type. Phantom type is a generic constraint defined in any type but is not used inside. Let's take a look at the example.

=====================================================

As you can see, we define Response type, but we didn't use it anywhere in the Request type. That's why it is called phantom type. Defining phantom types allows us to store information about the response in our request type. For example, it might be a type conforming or Decodable or even an instance of the Data type. Let's update our extension on URLSession to support response decoding.

=====================================================

In the example above, we also introduced proper error management to handle errors more gracefully. Let's define a Github repo search request using the new API.

=====================================================

#### Conclusion
Today we built a type-safe networking layer using Swift features like enums, phantom types, and extensions. This toolbox allows you to transform any old API into a safe and modern API. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!