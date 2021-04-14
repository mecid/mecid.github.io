---
title: Composable styling in SwiftUI
layout: post
category: View Composition
---

This week I want to talk about the styling of views in SwiftUI. SwiftUI provides a pretty composable architecture for building your apps. Every screen in terms of SwiftUI is a function on some data which returns a view. So let's talk today about composable and highly reusable styling options which we have in SwiftUI.

{% include friends.html %}

#### Branding
Whenever I begin a project, I start with defining my brand color for my User Interface. I use brand color as a tint for my buttons, switches, slider, etc. We can easily set the tint color on every view in the app by using *accentColor* modifier on the root view. Here is a quick example. 

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        window = UIWindow(windowScene: scene as! UIWindowScene)
        window?.rootViewController = UIHostingController(
            rootView: RootView()
                .accentColor(.purple)
        )

        window?.makeKeyAndVisible()
    }
```

SwiftUI uses *Environment* feature to pass the values implicitly inside any child view. This is how we can give an accent color to every view across the app. To learn more about *Environment* feature of SwiftUI, check my dedicated post ["The power of Environment in SwiftUI"](/2019/08/21/the-power-of-environment-in-swiftui/).

Another must-have option which I want to enable on every view in my app is line limit. I want to make every text in my app multi-lined in the case when it is too long. I also need it when a user enables extra large font size for Dynamic Type. It is also straightforward to achieve by adding *lineLimit* modifier to my root view.

```swift
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        window = UIWindow(windowScene: scene as! UIWindowScene)
        window?.rootViewController = UIHostingController(
            rootView: RootView()
                .accentColor(.purple)
                .lineLimit(nil)
        )

        window?.makeKeyAndVisible()
    }
```

#### Button styles
I often have a few button types which I reuse across the app. Before SwiftUI, I was using inheritance to apply my styling to *UIButtons* in *UIKit*. But SwiftUI relies on composition instead of inheritance, that's why it provides us *ButtonStyle* protocol which we can implement to reuse our button styles across the app. 

```swift
import SwiftUI

struct OutlineStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .frame(minWidth: 44, minHeight: 44)
            .padding(.horizontal)
            .foregroundColor(Color.accentColor)
            .background(RoundedRectangle(cornerRadius: 8).stroke(Color.accentColor))
    }
}

struct FillStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .frame(minWidth: 44, minHeight: 44)
            .padding(.horizontal)
            .foregroundColor(configuration.isPressed ? .gray : .white)
            .background(Color.accentColor)
            .cornerRadius(8)
    }
}
```

As you can see in the example above, we implement two button styles: *OutlinedButton* and *FilledButton*. To apply them to a button in SwiftUI, we have to use button style modifier. Let's see how we can use them.

```swift
HStack {
    store.monthly.map { product in
        Button("\(product.localizedPrice) / \(product.localizedPeriod)") {
            self.store.buyProduct(product)
            self.presentation.wrappedValue.dismiss()
        }.buttonStyle(OutlineStyle())
    }

    store.annually.map { product in
        Button("\(product.localizedPrice) / \(product.localizedPeriod)") {
            self.store.buyProduct(product)
            self.presentation.wrappedValue.dismiss()
        }.buttonStyle(FillStyle())
    }
}.padding()
```

**I want to note that you can use *buttonStyle* modifier on any view in SwiftUI and it utilizes *Environment* feature to share the style with any button inside it.** 

#### Text styles
Similar to buttons, I have a few different styling options for my text representation. Unfortunately, SwiftUI doesn't provide something like *TextStyle* protocol. But instead, it gives us a much more powerful composition concept, and it is *ViewModifier*. Let's take a look at how we can use *ViewModifiers* to style our Text views.

```swift
struct TitleStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.title)
            .lineSpacing(8)
            .foregroundColor(.primary)
    }
}

struct ContentStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.body)
            .lineSpacing(4)
            .foregroundColor(.secondary)
    }
}

extension Text {
    func textStyle<Style: ViewModifier>(_ style: Style) -> some View {
        ModifiedContent(content: self, modifier: style)
    }
}
```

In the example above, we implement two style modifiers. We also provide an extension for *Text* component, which allows us easily apply any modifier to a *Text*. Now we can use our styles by adding *textStyle* modifier to any *Text* component.

```swift
VStack {
    Text("title")
        .textStyle(TitleStyle())
    Text("content")
        .textStyle(ContentStyle())
}
```

*ViewModifiers* allow us to encapsulate and reuse any logic across the *Views*. To learn more about *ViewModifiers*, take a look at my dedicated post ["ViewModifiers in SwiftUI"](/2019/08/07/viewmodifiers-in-swiftui/).

#### Conclusion
Today we learned how to create highly reusable styling components for SwiftUI by using features like *ViewModifiers* and *Environment*. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 