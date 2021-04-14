---
title: Creating DSL in Swift
layout: post
category: Swift Language Features
---

This week we will talk about creating DSL in Swift. Let's start with the understanding of DSL acronym. Domain Specific Language is a language hosted by parent language to solve any specific area. An excellent example of DSL can be HTML which is DSL for creating web page markup.

{% include friends.html %}

There are some requirements for a language in which you want to create DSL. A host language should have first-class functions, trailing closures and operator overloading to make DSL possible. The great news is that Swift has all of these features.

We are going to simplify User Interface development on iOS by creating UIKit specific DSL. We have two ways of building UI in iOS. The first one is by using Interface Builder, and the second one is via code. Both of them have pros and cons. For instance, building UI by Interface Builder is a high-speed and visual process, but you have to deal with a problematic code review process, because of the format of Xibs and Storyboards. In case of building your UI by code, you get the reusability and clean code review process, but you deal with the imperative and error-prone codebase, without a visual understanding of view hierarchy.

Let's set our goals in building UIKit DSL in Swift:
1. Visual view hierarchy
2. Declarative like HTML
3. Type-safe and compile-time guarantee of correctness.

```swift
let rootView = stack {
    $0.spacing = 16
    $0.axis = .vertical
    $0.isLayoutMarginsRelativeArrangement = true

    $0.stack {
        $0.distribution = .fillEqually
        $0.axis = .horizontal

        $0.label {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "Hello"
        }

        $0.label {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "World"
        }

        $0.customLabel {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "!!!"
        }
    }

    let messageButton = $0.button {
        $0.tintColor = .white
        $0.setTitle("Say Hi!", for: .normal)
    }

    $0.view {
        $0.backgroundColor = .clear
    }
}
```

The code above is our target, and this is how we want our DSL to be. Generally, everything in this example is a function with a trailing closure parameter. For more details let's dive into implementation.

#### Root elements
We have to create a function stack which returns *UIStackView* and accepting closure which we apply to this new created *UIStackView*.

```swift
public func stack(apply closure: (UIStackView) -> Void) -> UIStackView {
    let stack = UIStackView()
    closure(stack)
    return stack
}
```

These lines give us an opportunity to use stack function similar to HTML tag.

```swift
stack {
    $0.axis = .vertical
}
```

As the first parameter of the trailing closure, we get created *UIStackView* on which we can call customization functions like changing axis, alignment, etc. Next, we want to call *$0.label* to configure new *UILabel* and add to previous *UIStackView*. Let's create an extension for *UIStackView* which provides us with label function.

```swift
extension UIStackView {
    @discardableResult
    public func label(apply closure: (UILabel) -> Void) -> UILabel {
        let label = UILabel()
        addArrangedSubview(label)
        closure(label)
        return label
    }
}
```

We use *@discardableResult* annotation to disable swift compiler warning on ignoring the result of this function because we already added it to *UIStackView*. Here is the example of label function usage.

```swift
stack {
    $0.axis = .vertical
    $0.label {
        $0.adjustsFontForContentSizeCategory = true
    }
}
```

We have one problem here, and this is the extension on *UIStackView*, only *UIStackView* will have this function. But we need it in any *UIView* subclass, so let's move it to *UIView* extension.

```swift
extension UIView {
    @discardableResult
    public func label(apply closure: (UILabel) -> Void) -> UILabel {
        let label = UILabel()
        if let stack = self as? UIStackView {
            stack.addArrangedSubview(label)
        } else {
            addSubview(label)
        }
        closure(label)
        return label
    }
}
```

We try here to cast self to *UIStackView*, which give us the ability to use *addArrangedSubview* in case of *UIStackView*, if not we add it with the *addSubview* method. Next step is populating our *UIView* extension with functions for all UIKit components to make above usage possible for every UIKit component. I've added DSL support for all UIKit components. You can check it out on [Github](https://github.com/mecid/UIKitSwiftDSL). 

```swift
let rootView = stack {
    $0.spacing = 16
    $0.axis = .vertical
    $0.isLayoutMarginsRelativeArrangement = true

    $0.stack {
        $0.distribution = .fillEqually
        $0.axis = .horizontal

        $0.label {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "Hello"
        }

        $0.label {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "World"
        }

        $0.customLabel {
            $0.textAlignment = .center
            $0.textColor = .white
            $0.text = "!!!"
        }
    }

    let messageButton = $0.button {
        $0.tintColor = .white
        $0.setTitle("Say Hi!", for: .normal)
    }

    $0.view {
        $0.backgroundColor = .clear
    }
}
```

Now we achieve declarative, tree-based and type-safe DSL for building UI for iOS. [It is available via CocoaPods and Carthage](https://github.com/mecid/UIKitSwiftDSL).

#### Conclusion
Today we learned how powerful is Swift, and how easy we can create DSL for any specific domain. I suggest you try to develop your DSL for *DispatchQueue* or any other area.

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!