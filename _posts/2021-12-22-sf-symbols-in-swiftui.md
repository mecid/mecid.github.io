---
title: SF Symbols in SwiftUI
layout: post
---

Apple provides us with a set of different icons available across all the platforms called SF Symbols. SF Symbols package contains more than 3200 icons that we can use to visualize different states and actions in our apps. This week we will learn how to utilize the power of SF Symbols in SwiftUI views.

#### Basics
You can easily use SF Symbols in SwiftUI by using a particular initializer of the Image struct. Let's take a look at a few quick examples.

=====================================================

As you can see in the example above, we define an image with a system name. SwiftUI uses the systemName parameter for SF Symbol lookup. Keep in mind that you can use string interpolation to show an SF Symbol as the part of any text.

=====================================================

Another SwiftUI view that plays well with SF Symbols is Label. The Label view contains both text and an image and shows them according to the current context. For example, it hides the text and shows only the image in a toolbar.

=====================================================

#### Styling options
You can easily change the color of an SF Symbol using the foregroundStyle or foregroundColor view modifiers.

=====================================================

There are two ways of changing the size of an SF Symbol in SwiftUI view. First, you can apply the font view modifier to keep the text and the image size in sync with the selected Dynamic Type category.

=====================================================

Another option is the imageScale view modifier that doesn't affect the size of the text and only changes the size of the SF Symbol relatively.

=====================================================

#### Variants
SF Symbols package provides us with icons in many different variants. For example, the trash icon comes with filled, circled, squared, and slashed versions. You can define the variant you need by providing the dedicated symbol name.

=====================================================

SwiftUI views are smart enough to display the variant they need without the concrete name. For example, tab bars use the filled version of SF Symbols, and you don't need to specify it manually. SwiftUI picks up the correct version of the SF Symbol automatically.

=====================================================

SwiftUI also provides the symbolVariant view modifier that sets the particular variant in the environment and forces it to use in the view hierarchy.

=====================================================

#### Rendering mode
SF Symbols support four rendering modes that allow you to customize the way SwiftUI colors them. Let's take a look at them.

Monochrome: A mode that renders symbols as a single layer filled with color.
Multicolor: This method generates symbols as multiple layers with their inherited styles.
Hierarchical: A mode renders symbols as various layers, with different opacities applied to the foreground style.
Palette: A way that renders symbols as numerous layers, with different styles used as the layers.

You can use the symbolRenderingMode view modifier to set a particular rendering mode in the environment and apply it across the view hierarchy.

=====================================================


#### Conclusion
It became straightforward to build friendly apps with a well-crafted set of icons using SF Symbols. Moreover, SwiftUI provides a few ways of customizing these icons by using dedicated view modifiers that we have learned today.
