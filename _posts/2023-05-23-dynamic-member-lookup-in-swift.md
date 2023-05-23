---
title: Dynamic member lookup in Swift
layout: post
---

One of my favorite features of the Swift Language is the dynamic member lookup. We don't use it very often, but it improves the API of the provided type significantly by improving the way we access the data of the particular type.

#### Basics
Assume that we are working on a type providing caching functionality, and we model it as the struct called Cache.

=====================================================

To access cached data, we call the subscript of the storage property that the Dictionary type provides us.

=====================================================

Nothing is exceptional here. We access the dictionary as we used by using the subscript of the Dictionary type. Let's look at how we can improve the API of the Cache type using the @dynamicMemberLookup attribute.

=====================================================

As you can see in the example above, we mark our Cache type with the @dynamicMemberLookup attribute. We must implement the subscript with the dynamicMember parameter returning anything we need.

=====================================================

Now, we can access the profile data of our Cache type more nicely. The user of our API may assume that the profile is the property of the Cache type. But it is not.

This feature works completely in runtime and leverages the name of any property we type after the dot symbol to the subscript of the Cache type with the dynamicMember parameter. The whole logic runs in runtime, and the result is undefined during compilation. It is entirely up to you to decide which data you should return from the subscript during runtime.

#### Compile-time safety with KeyPath
The only downside we can find is the absence of compile-time safety. We can treat the Cache type as if it has any property name we type in the code. Fortunately, the parameter of the @dynamicMemberLookup subscript may be not only string but also KeyPath.

=====================================================

As you can see in the example above, we define the subscript with the dynamicMember parameter accepting an instance of the strong-typed KeyPath. In this case, we allow KeyPath of the State type, which helps us to have compile-time safety. Because the compiler will show an error anytime we pass a wrong KeyPath, which is not connected to the State type.

=====================================================

In the example above, we access the private state property of the Store using the subscript accepting the KeyPath. It looks similar to the previous example, but in this case, the compiler shows an error whenever you try to access an unavailable property of the State type.

Today we learned how to improve the API of a particular type by using the @dynamicMemberLookup attribute. You don't need it in every type, but you can use it carefully to improve the API.
