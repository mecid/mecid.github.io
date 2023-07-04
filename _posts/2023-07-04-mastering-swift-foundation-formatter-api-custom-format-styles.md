---
title: Mastering Swift Foundation Formatter API. Custom Format Styles
layout: post
image: /public/swift.png
category: Swift Language Features
---

Swift Foundation Formatter API is one of my favorite recent additions to the Swift ecosystem. I use it in every project and build my custom-type formatting logic using the same approach. This week we will learn how to introduce custom formatters and use them with our own types.

{% include friends.html %}

As I said in the previous post, many types from the Swift Foundation provides *formatted* function allowing us to format dates, numbers, etc., into the user-presentable style. The whole formatting logic lives outside of the concrete type. The Swift Foundation defines many different types conforming to the *FormatStyle* protocol, allowing encapsulation of the formatting logic.

```swift
protocol FormatStyle {
    associatedtype Input
    associatedtype Output

    func format(_ value: Input) -> Output
}
```

The *FormatStyle* protocol is simple. It has the only required function called *format*. It takes two generic parameters: one for input and another for output. Using generic parameters allows us to format any type into any presentation you need. But let's start with a simple example of a custom format style.

```swift
struct ShortNumberFormat: FormatStyle {
    let length: Int
    func format(_ value: Double) -> String {
        value.formatted(.number.precision(.fractionLength(0...length)))
    }
}

extension FormatStyle where Self == ShortNumberFormat {
    static func short(length: Int = 1) -> ShortNumberFormat {
        .init(length: length)
    }
}
```

As you can see in the example above, we create the *ShortNumberFormat* type conforming to the *FormatStyle* protocol. We define generic parameters of the *format* function as *Double* and *String*. It means it should convert any instance of the *Double* type to a *String*.

```swift
let number = 100.001
Text(verbatim: number.formatted(.short(length: 2)))
```

Now we can easily reuse our short number format style across the codebase and cover it with unit tests to verify the formatting logic. As I said before, the *FormatStyle* protocol allows us to convert any type into anything we might need. In the following example, we will create a format style converting a double to an attributed string.

```swift
struct BoldNumberFormat: FormatStyle {
    func format(_ value: Double) -> AttributedString {
        var string = AttributedString(value.formatted(.number))
        
        if let range = string.range(of: String(Int(value))) {
            string[range].inlinePresentationIntent = .stronglyEmphasized
        }
        
        return string
    }
}

extension FormatStyle where Self == BoldNumberFormat {
    static var bold: BoldNumberFormat { .init() }
}
```

As you can see in the example above, we define the *BoldNumberFormat* type that conforms to the *FormatStyle* and generates an attributed string from a double. We use the bold style for the part of the number before the decimal separator.

```swift
let a = 10.001
Text(a.formatted(.bold))
```

We can use the *BoldNumberFormat* type with *Text* in SwiftUI or *UILabel* in UIKit to display doubles using attributed strings.

The *formatted* function is part of the many Swift Foundation types, but what about our own types? We can easily define the same formatted function on our types and reuse the *FormatStyle* protocol.

```swift
struct Product {
    let title: String
    let price: Decimal
    let oldPrice: Decimal?
}

extension Product {
    func formatted<Style: FormatStyle>(
        _ style: Style
    ) -> Style.FormatOutput where Style.FormatInput == Self {
        style.format(self)
    }
}

struct ProductPriceFormat: FormatStyle {
    func format(_ value: Product) -> AttributedString {
        guard let oldPrice = value.oldPrice else {
            return value.price.formatted(.number.attributed)
        }

        let priceFormatted = value.price.formatted()
        let oldPriceFormatted = oldPrice.formatted()

        var string = AttributedString("\(priceFormatted) \(oldPriceFormatted)")

        if let range = string.range(of: oldPriceFormatted) {
            string[range].inlinePresentationIntent = .strikethrough
        }

        return string
    }
}

extension FormatStyle where Self == ProductPriceFormat {
    static var price: ProductPriceFormat { .init() }
}
```

As you can see in the example above, we define the formatted function on our *Product* type. We also create the *ProductPriceFormat* type that uses current and old prices to generate an attributed string with a price where the old price is crossed out.

```swift
let product = Product(title: "App", price: 0.99, oldPrice: 1.99)
Text(product.formatted(.price))
```

I use this approach to encapsulate and reuse my formatting logic for my own model types. But usually, instead of creating a bunch of types conforming to the *FormatStyle* protocol, I define a nested *Formatter* struct inside a model type and create a bunch of extensions to cover every single use-case I need.

```swift
extension Product {
    struct Formatter<Output> {
        let format: (Product) -> Output
    }
    
    func formatted<Output>(_ formatter: Formatter<Output>) -> Output {
        formatter.format(self)
    }
}

extension Product.Formatter where Output == AttributedString {
    static var price: Self {
        .init { value in
            guard let oldPrice = value.oldPrice else {
                return value.price.formatted(.number.attributed)
            }
            
            let priceFormatted = value.price.formatted()
            let oldPriceFormatted = oldPrice.formatted()
            
            var string = AttributedString("\(priceFormatted) \(oldPriceFormatted)")
            
            if let range = string.range(of: oldPriceFormatted) {
                string[range].inlinePresentationIntent = .strikethrough
            }
            
            return string
        }
    }
}

let anotherProduct = Product(title: "Another app", price: 0.99, oldPrice: 1.99)
Text(anotherProduct.formatted(.price))
```

This week we discussed encapsulating formatting logic for our model types by reusing the lovely approach from the Swift Foundation.
