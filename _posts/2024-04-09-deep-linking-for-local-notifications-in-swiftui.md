---
title: Deep linking for local notifications in SwiftUI
layout: post
---

Notifications are crucial for keeping users engaged with your app. Almost all of my apps provide notifications that not only launch the app but also navigate to different parts of my apps. Today, I want to share how I build deep links for local notifications in my apps.

SwiftUI introduced the onOpenURL view modifier to handle universal links in our apps. Here is a quick example showing how to use the onOpenURL view modifier.

=====================================================

As you can see in the example above, we use the onOpenURL view modifier to parse the opened URL and provide navigation inside the app by displaying the offer sheet.

Before using the onOpenURL view modifier, you must register the URL scheme you want to open in the app. Go to project settings, choose your target, and register a URL scheme in the URL types section.

Now, you are ready to handle links in your app. You can try to test how the app handles links by opening a link in Safari, for example, myapp://offer.

I want to schedule a notification that displays a special offer to the user after the first launch in 30 minutes. I wrote an extension for UNUserNotificationCenter to make it easier.

=====================================================

As you can see in the example above, I've created the addOfferNotification function to schedule a notification in 30 minutes. I provide the title and body of the notification and include the URL field in the userInfo dictionary.

The idea is to intercept notifications with the URL field in the userInfo dictionary and open the URL instead of launching the app. The UNUserNotificationCenter type allows us to provide a delegate to handle the moment the user taps the notification.

The only place where you should set the delegate for the UNUserNotificationCenter type is the AppDelegate's willFinishLaunchingWithOptions method. So, we need to define AppDelegate for our SwiftUI app. Fortunately, it is possible.

=====================================================

As you can see, we define the AppDelegate type conforming to the UIApplicationDelegate protocol. We also conform to the UNUserNotificationCenterDelegate protocol and implement the didReceive function. Our app will call this function whenever the user taps a notification while the app is in the background.

It is the best place to verify whenever our notification contains the URL to launch and open the URL using a shared instance of the UIApplication type. We can ignore other notifications that don't include the URL field.

=====================================================

In the example above, we use the UIApplicationDelegateAdaptor property wrapper to define an AppDelegate for the SwiftUI app. We also observe the scene phase to schedule the offer notification after the first launch.

