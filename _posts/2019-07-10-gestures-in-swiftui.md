---
title: Gestures in SwiftUI
layout: post
category: Interactions
---

SwiftUI has a powerful and easy to use approach in building *Gestures*. Today we will talk about how we can use gestures in SwiftUI. We will touch special *GestureState Property Wrapper* which is very similar to *State* but works only with gestures. Finally, we will build swipeable Tinder cards as a sample project.

{% include friends.html %}

#### Gesture modifier
SwiftUI provides a bunch of ready to use gestures like *TapGesture, DragGesture, RotationGesture, MagnificationGesture, LongPressGesture*. You can use them by attaching gesture modifier to any view. Let's take a look at a code sample. 

```swift
import SwiftUI

struct ContentView: View {
    @GestureState var isLongPressed = false

    var body: some View {
        let longPress = LongPressGesture()
            .updating($isLongPressed) { value, state, transaction in
                state = value
        }

        return Rectangle()
            .fill(isLongPressed ? Color.purple : Color.red)
            .frame(width: 300, height: 300)
            .cornerRadius(8)
            .shadow(radius: 8)
            .padding()
            .scaleEffect(isLongPressed ? 1.1 : 1)
            .gesture(longPress)
            .animation(.interactiveSpring(), value: isLongPressed)
    }
}
```

Here we use @*GestureState Property Wrapper* to bind gesture value changes to *isLongPressed* property. To attach gesture changes to @*GestureState* property, we have to call *updating* method on gesture instance and pass property wrapper with closure where we implement binding. In the current sample, we just bind a value to the state, but in more complex gestures, we can have here any calculations before assigning a new value to the state. Now we can use *isLongPressed* while declaring the view to animate changes based on the gesture. SwiftUI will rebuild the view whenever *isLongPressed* changes.

The critical point here is that SwiftUI reset property marked with @*GestureState* when gesture ended. Keep it in mind and use @*State* whenever you need to store value after gesture finish. 

> To learn more about *Property Wrappers* provided by SwiftUI, take a look at my ["Understanding Property Wrappers in SwiftUI" post](/2019/06/12/understanding-property-wrappers-in-swiftui/).

As a result, we have a red rectangle which scales and change color to purple during a long-press gesture. As soon as gesture finishes SwiftUI resets *isLongPressed* to initial false value rebuilds view to show red rectangle again. All these transitions nicely animated by adding animation modifier with interactive spring.

#### DragGesture
Let's try to create a Tinder-like swipeable card. We will use *DragGesture* to track dragging. When the user finishes the dragging we have to check if translation enough to remove the card, if not we will animate it back to the center of the screen. Here is the implementation.

```swift
import SwiftUI

struct ContentView: View {
    @State private var offset: CGSize = .zero

    var body: some View {
        let drag = DragGesture()
            .onChanged { self.offset = $0.translation }
            .onEnded {
                if $0.translation.width < -100 {
                    self.offset = .init(width: -1000, height: 0)
                } else if $0.translation.width > 100 {
                    self.offset = .init(width: 1000, height: 0)
                } else {
                    self.offset = .zero
                }
        }

        return PersonView()
            .background(Color.red)
            .cornerRadius(8)
            .shadow(radius: 8)
            .padding()
            .offset(x: offset.width, y: offset.height)
            .gesture(drag)
            .animation(.interactiveSpring(), value: offset)
    }
}

struct PersonView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Rectangle()
                .fill(Color.gray)
                .cornerRadius(8)
                .frame(height: 300)

            Text("Majid Jabrayilov")
                .font(.title)
                .foregroundColor(.white)

            Text("iOS Developer")
                .font(.body)
                .foregroundColor(.white)
        }.padding()
    }
}
```

Instead of using @*GestureState* we use @*State* here because when the gesture ends, we don't need to reset offset, we want to increase it in the right direction to animate cart move outside the screen. Instead of using the *updating* method, we use *onChanged* and *onEnded* gesture callbacks, where we can make our calculations and state updates. Now we have pleasant dragging experience which is animated by spring only by adding *animation* modifier. 

> To learn more about animations in SwiftUI, please take a look at my ["Animations in SwiftUI"](/2019/06/26/animations-in-swiftui/) post.

#### Composing gestures
Sometimes we need to add more than one gesture to a *View*, and for this special case, SwiftUI provides three ways of gesture composition.

1. Simultaneous
2. Sequenced
3. Exclusive

Let's add a dragging gesture simultaneously with a long-press gesture to our red rectangle sample.

```swift
import SwiftUI

struct ContentView: View {
    @State private var offset: CGSize = .zero
    @GestureState var isLongPressed = false

    var body: some View {
        let longPressAndDrag = LongPressGesture()
            .updating($isLongPressed) { value, state, transaction in
                state = value
        }.simultaneously(with: DragGesture()
            .onChanged { self.offset = $0.translation }
            .onEnded { _ in self.offset = .zero }
        )

        return Rectangle()
            .fill(isLongPressed ? Color.purple : Color.red)
            .frame(width: 300, height: 300)
            .cornerRadius(8)
            .shadow(radius: 8)
            .padding()
            .scaleEffect(isLongPressed ? 1.1 : 1)
            .offset(x: offset.width, y: offset.height)
            .gesture(longPressAndDrag)
            .animation(.interactiveSpring(), value: offset)
            .animation(.interactiveSpring(), value: isLongPressed)
    }
}
```

Now we can both drag and long-press our rectangle, and it changes position and scale as expected.

#### Conclusion
SwiftUI has a powerful declarative way of handling gestures. Try to add some delight to your app by adding gestures. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  