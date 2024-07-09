---
title: Introducing Entry macro in SwiftUI
layout: post
---

The Swift macros feature became very popular last year in the community and inside Apple. As a result, the SwiftUI framework introduced a set of macro types that helped us reduce boilerplates in our codebases. This week, we will talk about the Entry macro type.

The SwiftUI framework introduces the environment feature, implicitly allowing us to share data in the view hierarchy. This feature becomes very useful when you need to share settings or the user state across many views of your app. Let's look at the example of defining a custom environment key holding the user subscription state.

=====================================================

As you can see in the example above, we define the UserStateEnvironmentKey type conforming to the EnvironmentKey protocol. The only requirement of the EnvironmentKey protocol is the defaultValue property. Next, we have to add an extension for the EnvironmentValues type where we provide access to the instance of the UserStateEnvironmentKey type.

Imagine you have a set of custom properties you must share via the environment. In this case, you must create many types conforming to the EnvironmentKey protocol and repeat the code for every property. Fortunately, the new Entry macro saves us from making a boilerplate by simplifying the code we need to write to conform to the custom environment key.

=====================================================

As you can see in the example above, all you need to do is create an extension for the EnvironmentValues type and define a variable with the Entry macro. It's that simple. The Entry macro will take care of the rest, relieving you from the need to specify a type conforming to the EnvironmentKey protocol. The Entry macro will do it implicitly for you, making your coding experience more straightforward and less complex.

=====================================================

Now, you can easily use the environment view modifier to set the particular environment value using the keypath that the Entry macro creates for you. The Entry macro works for the environment, transactions, container, and focused values.

=====================================================

Here is another example demonstrating the use of focused values. As you can see, we define the FocusedNoteValue type conforming to the FocusedValueKey protocol. It looks very similar to the custom environment key and requires the same amount of boilerplate per custom value.

=====================================================

As I said before, the Entry macro also works excellently with focused values. The only difference is the property that you define in the extension of the FocusedValues type must be optional because it is only available when the view is focused.

Today, we learned how to use the Entry macro to simplify our code and reduce the boilerplate. The Entry macro is backward compatible, and you can use it even with previous versions of Apple platforms.

