---
title: The many faces of buttons in SwiftUI
layout: post
image: /public/buttons1.png
category: Mastering SwiftUI view
---

Button is one of the crucial components of any app. We use buttons to provide actions in the user interface of the app. SwiftUI 3 released a bunch of new view modifiers that allow us to style buttons in different ways. The new bordered button style in conjunction with controlProminence and controlSize view modifiers can change button presentation drastically.

{% include friends.html %}

#### Button role
New in SwiftUI Release 3, you can provide an optional button role. By default, it is nil and uses a standard one, but you can set the predefined role provided by ButtonRole enum. The role can be destructive or cancel.
In this case, SwiftUI will set a specified button style. For example, SwiftUI changes the button tint to red for destructive buttons.

```swift
Button("Delete", role: .destructive) {
    viewModel.delete()
}
```
![button-destructive](/public/buttons-destructive.png)

Button roles change the appearance in many places across the app, like context menus, toolbar menus, etc.

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            Text("Hello, World!")
                .toolbar {
                    Menu("Actions") {
                        Button("New action") {}
                        Button("Delete", role: .destructive) {}
                }
            }.navigationTitle("Buttons")
        }
    }
}
```

![button-toolbar](/public/buttons-toolbar.png)

#### Bordered button style
There is a new BorderedButtonStyle type that allows us to display buttons with rounded corners. You can set the button style for a particular button or the full view hierarchy using the buttonStyle view modifier.

```swift
Button("New action") {}
    .buttonStyle(.bordered)
```

![button-bordered](/public/buttons-bordered.png)

BorderedButtonStyle provides you a bordered button appearance with rounded corners that you can see in many places across the iOS system. 

#### Button tint
There is a new tint view modifier that we should use to override the default accent color. Unlike an app's accent color, which can be overridden by user preference, the tint color is always respected.

```swift
Button("New action") {}
    .buttonStyle(.bordered)
    .tint(.green)
```

![button-tint](/public/buttons-tint.png)

#### Control size
We can't directly control the corner radius of the bordered button, but we can affect it using the controlSize view modifier. The controlSize view modifier allows us to set the size of controls within the view. There is a ControlSize enum with four cases: mini, small, regular, and large. We can use one of them and pass it via the controlSize modifier.

```swift
 Button("New action") {}
    .tint(.green)
    .buttonStyle(.bordered)
    .controlSize(.large)
```

![button-bordered-tint](/public/buttons-bordered-tint.png)

In the example above, we set the large size for controls in our view hierarchy. As you can see, it affects the size of our button and changes its corner radius.

#### Control prominence
Control prominence defines the dominance of control in the user interface. You can set the importance using the controlProminence view modifier that accepts one of two possible prominence cases: standard or increased.

```swift
Button("New action") {}
    .tint(.green)
    .buttonStyle(.bordered)
    .controlSize(.large)
    .controlProminence(.increased)
```

![button-bordered-tint-important](/public/buttons-tint-fill.png)

As you can see in the example above, SwiftUI changes button appearance whenever we set the increased prominence. SwiftUI displays buttons with increased prominence by filling them with tint color.

#### Conclusion
There are a lot of new opportunities for customizing buttons both in SwiftUI and UIKit. New control size and prominence APIs will play a crucial role in styling SwiftUI buttons without implementing custom button styles. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!