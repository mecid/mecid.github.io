---
title: Introducing UIGestureRecognizerRepresentable protocol in SwiftUI
layout: post
---

SwiftUI provides the UIViewRepresentable and UIViewControllerRepresentable protocols since its inception. As you might know, we can use them to wrap any UIKit view or controller and place it in the SwiftUI hierarchy. The UIGestureRecognizerRepresentable protocol is another addition to the collection of representable protocols. This week, we will learn how to use the new UIGestureRecognizerRepresentable protocol.

The UIGestureRecognizerRepresentable works similarly to other representable protocols and allows us to wrap any instance of the UIGestureRecognizer type to introduce it in the SwiftUI views.

```swift
struct MyGestureRecognizerRepresentable: UIGestureRecognizerRepresentable {
    let action: () -> Void
    
    func makeUIGestureRecognizer(context: Context) -> some UIGestureRecognizer {
        let recognizer = UITapGestureRecognizer()
        recognizer.numberOfTouchesRequired = 2
        return recognizer
    }
    
    func handleUIGestureRecognizerAction(_ recognizer: UIGestureRecognizerType, context: Context) {
        switch recognizer.state {
        case .possible:
        case .ended:
            action()
        default:
            break
        }
    }
}

struct ContentView: View {
    var body: some View {
        Circle()
            .gesture(MyGestureRecognizerRepresentable {
                print("two fingers tap happened")
            })
    }
}
```

As you can see in the example above, we define the MyGestureRecognizerRepresentable type conforming to the UIGestureRecognizerRepresentable protocol. It has the only requirement, the makeUIGestureRecognizer function, where you have to initialize and return an instance of the UIGestureRecognizer type.

The handleUIGestureRecognizerAction is not required by the protocol and provides a default implementation, but you should override it and provide your implementation to handle the state of the gesture and decide when to call the action closure.

In the handleUIGestureRecognizerAction function, you have access to the instance of the particular UIGestureRecognizer, which means you can verify its state, location in the view, etc.

The SwiftUI framework offers a variety of pre-built gesture types, but the UIGestureRecognizerRepresentable protocol becomes particularly useful when you want to create your own custom gestures. For instance, checkmark gestures, circle gesture, etc.

Usually, to use attached SwiftUI gestures, we use the gesture view modifier; the same is true for types conforming to the UIGestureRecognizerRepresentable protocol.

In almost every case when you build the custom gesture, you need to set up a gesture delegate. Similarly to other representable protocols, we can use the coordinator.

```swift
struct MyGestureRecognizerRepresentable: UIGestureRecognizerRepresentable {
    let action: () -> Void
    
    func makeUIGestureRecognizer(context: Context) -> some UIGestureRecognizer {
        let recognizer = UITapGestureRecognizer()
        recognizer.numberOfTouchesRequired = 2
        recognizer.delegate = context.coordinator
        return recognizer
    }
    
    func makeCoordinator(converter: CoordinateSpaceConverter) -> Coordinator {
        Coordinator(converter: converter)
    }
    
    func handleUIGestureRecognizerAction(_ recognizer: UIGestureRecognizerType, context: Context) {
        switch recognizer.state {
        case .possible:
        case .ended:
            action()
        default:
            break
        }
    }
    
    final class Coordinator: NSObject, UIGestureRecognizerDelegate {
        let converter: CoordinateSpaceConverter
        init(converter: CoordinateSpaceConverter) {
            self.converter = converter
        }
        
        func gestureRecognizerShouldBegin(_ gestureRecognizer: UIGestureRecognizer) -> Bool {
            // your logic here
        }
    }
}

struct ContentView: View {
    var body: some View {
        Circle()
            .gesture(MyGestureRecognizerRepresentable {
                print("two fingers tap happened")
            })
    }
}
```

As you can see, we define the Coordinator type conforming to the UIGestureRecognizerDelegate protocol and set it in the makeUIGestureRecognizer function. We also should define the makeCoordinator function where we create and return our instance of the coordinator.

While creating the coordinator, we have access to the instance of the CoordinateSpaceConverter type, which will be useful in your delegate. It allows us to convert coordinates between different spaces, for example, SwiftUI view and gesture recognizer touch location. You can also access the fresh instance of the CoordinateSpaceConverter via the context parameter in the makeUIGestureRecognizer and handleUIGestureRecognizerAction functions.
