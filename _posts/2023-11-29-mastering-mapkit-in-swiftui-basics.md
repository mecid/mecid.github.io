---
title: Mastering MapKit in SwiftUI. Basics
layout: post
---

MapKit integration with SwiftUI significantly changed this year. In the previous version of SwiftUI, we had very basic functionality of MKMapView wrapped into the SwiftUI view named Map. Fortunately, things changed, and SwiftUI introduced a new API for MapKit integration. This week, we will learn how to use the new full-featured APIs available in the latest iteration of SwiftUI to integrate with MapKit.

As I said before, in the previous versions of the SwiftUI framework, we had a Map view providing us with basic functionality for MapKit, which is deprecated now. Using the deprecated Map view still makes sense whenever you target previous versions of Apple platforms.

> To learn more about the old MapKit integration with SwiftUI, take a look at my dedicated "Using MapKit with SwiftUI" post.

The new MapKit API introduces the MapContentBuilder result builder that looks similar to the ViewBuilder but instead uses the types conforming to the MapContent protocol. Let's start with the basic example of using the new MapKit APIs in SwiftUI.

=====================================================

As you can see in the example above, we define the map and place the content on it by using the MapContentBuilder closure. The MapContentBuilder type works with any type conforming to the MapContent protocol. 

In our example, we use Marker and Annotation types. The Marker is the essential item that allows us to place a predefined pin on the map. The Annotation type is more advanced and will enable us to place a SwiftUI on the map using the latitude and longitude.

SwiftUI provides us with a bunch of types conforming to the MapContent protocol. We already used two of them: Marker and Annotation. Many of them include MapCircle, MapPolygon, MapPolyline, UserAnnotation, etc.

You can control the initial position of the map by using another overload of the Map initializer that provides the initialPosition parameter.

=====================================================

The initialPosition parameter takes an instance of the MapCameraPosition type. The MapCameraPosition allows us to define the map position in a few ways. It can be the user location that we use in our example, or you can point it to any area on the map using the camera, region, rect, or item static functions. By default, it uses the automatic instance of the MapCameraPosition type that fits the map content.

Whenever you need constant control over the camera position, you can use another overload of the Map initializer, allowing you to provide two-way binding to the map camera position.

=====================================================

SwiftUI updates the position binding once the user drags the map. It also updates the map camera position as soon as you update the position property programmatically.

=====================================================

You can control the types of interactions allowed with the map by using the interactionModes parameters. The MapInteractionModes type defines a set of interactions like pan, zoom and all. By default, it enables all the available interaction types.

Today, we learned the basics of the MapKit integration in SwiftUI. We will continue the topic in the upcoming weeks by covering camera manipulations, map controls, and other advanced topics.
