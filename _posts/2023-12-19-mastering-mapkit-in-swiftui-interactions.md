---
title: Mastering MapKit in SwiftUI. Interactions.
layout: post
---

MapKit provides us with a very rich API as part of the next iteration of the SwiftUI framework. This week, we will continue the topic by learning how to handle interactions using the new MapKit API in SwiftUI.

In the previous post, we discussed the map view's camera position. Let me update your memory with the quick code example.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .camera(
        .init(centerCoordinate: .newYork, distance: 10_000_000)
    )
    
    var body: some View {
        Map(position: $position) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
            Marker("Seattle", monogram: Text("SE"), coordinate: .seattle)
            Marker("San Francisco", monogram: Text("SF"), coordinate: .sanFrancisco)
        }
        .onChange(of: position) {
            print(position.camera?.centerCoordinate)
            print(position.positionedByUser)
        }
    }
}
```

As you can see in the example above, we use the *onChange* view modifier to track changes in the two-way binding of the camera position. Unfortunately, we can't get the direct camera position from the binding in the case of user drag. For this particular case, MapKit API introduces the *onMapCameraChange* view modifier.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .camera(
        .init(centerCoordinate: .newYork, distance: 10_000_000)
    )
    
    var body: some View {
        Map(position: $position) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
            Marker("Seattle", monogram: Text("SE"), coordinate: .seattle)
            Marker("San Francisco", monogram: Text("SF"), coordinate: .sanFrancisco)
        }
        .onMapCameraChange(frequency: .continuous) { context in
            print(context.camera)
            print(context.region)
            print(context.rect)
        }
    }
}
```

In the example above, we use the *onMapCameraChange* view modifier to track camera changes as soon as the camera position changes. MapKit API allows us to set the frequency of the *onMapCameraChange* listener by passing an instance of the *MapCameraUpdateFrequency* type. The *MapCameraUpdateFrequency* enum provides us with two options: *continuous* and *onEnd*. The first defines nearly real-time changes in the camera position. The second fires whenever the camera position drags finish.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .camera(
        .init(centerCoordinate: .newYork, distance: 10_000_000)
    )
    
    var body: some View {
        Map(position: $position) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
            Marker("Seattle", monogram: Text("SE"), coordinate: .seattle)
            Marker("San Francisco", monogram: Text("SF"), coordinate: .sanFrancisco)
        }
        .onMapCameraChange(frequency: .onEnd) { context in
            print(context.camera)
            print(context.region)
            print(context.rect)
        }
    }
}
```

The second parameter of the *onMapCameraChange* view modifier is the action closure, which can handle camera position updates. The action closure provides us with an instance of the *MapCameraUpdateContext* type defining the current map camera, rectangle, and region.

The new MapKit API also introduces the *mapCameraKeyframeAnimator* view modifier, allowing us to animate the map camera using a keyframe animator.

```swift
struct ContentView: View {
    @State private var trigger = false
    @State private var position: MapCameraPosition = .camera(
        .init(centerCoordinate: .newYork, distance: 10_000_000)
    )
    
    var body: some View {
        Map(position: $position, selection: $selection) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
            Marker("Seattle", monogram: Text("SE"), coordinate: .seattle)
            Marker("San Francisco", monogram: Text("SF"), coordinate: .sanFrancisco)
        }
        .mapCameraKeyframeAnimator(trigger: trigger) { camera in
            KeyframeTrack(\MapCamera.centerCoordinate) {
                LinearKeyframe(.newYork, duration: 2)
                LinearKeyframe(.seattle, duration: 2)
                LinearKeyframe(.sanFrancisco, duration: 2)
            }
            
            KeyframeTrack(\MapCamera.distance) {
                LinearKeyframe(camera.distance, duration: 2)
                LinearKeyframe(camera.distance * 2, duration: 2)
                LinearKeyframe(camera.distance, duration: 2)
            }
        }
        .task {
            trigger.toggle()
        }
    }
}
```

As you can see in the example above, we use the *mapCameraKeyframeAnimator* view modifier to define a trigger value. Trigger value allows us to animate the map camera whenever the trigger value changes.

The second parameter of the *mapCameraKeyframeAnimator* view modifier is the *KeyframesBuilder* closure, which allows us to define a set of keyframe tracks. Inside these tracks, we describe the transition states to iterate our animation.

As you can see, we can animate all the properties of the *MapCamera* type. In our example, we animate the map camera's center location and distance. The *KeyframesBuilder* closure also provides us with the initial value of the map camera, allowing us to read the value of the map camera before animation. 

Last, the topic to cover is the map selection feature. The Map view provides an initializer with a *selection* parameter, allowing us to offer a two-way binding for map content selection.

```swift
struct ContentView: View {
    @State private var selection: Int?
    @State private var position: MapCameraPosition = .camera(
        .init(centerCoordinate: .newYork, distance: 10_000_000)
    )
    
    var body: some View {
        Map(position: $position, selection: $selection) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
                .tag(1)
            Marker("Seattle", monogram: Text("SE"), coordinate: .seattle)
                .tag(2)
            Marker("San Francisco", monogram: Text("SF"), coordinate: .sanFrancisco)
                .tag(3)
        }
        .onChange(of: selection) {
            print("selection changed:", selection)
        }
    }
}
```

In the example above, we define a state property to store the currently selected value of the map. We also annotate our markers using the *tag* view modifier. Remember that the type of the *selection* property must be the same as the *tag* you provide to the map content.
