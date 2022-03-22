---
title: Getting started with CloudKit
layout: post
---

CloudKit is an easy way to store data in the cloud, sync between multiple devices, and share it between the app's users. This week we will learn how to start using CloudKit in the app to save and fetch data from the cloud and sync between multiple user devices.

#### Basics
First, to start using CloudKit in the app, we need to enable it in the project Signing and Capabilities section. Xcode creates a default container for the app. A container is a space in the cloud that stores all of your saved data. You can use a container per application or a single container to share data between multiple apps.

Every container has a public, private and shared database. The public database is accessible for any user of the app. Every user of the app has a private database that lives in the personal iCloud and counts towards iCloud storage. You can use the shared database to fetch data shared with the user.

Before saving or fetching data from the CloudKit, we should check if the user has logged in an Apple ID and enabled iCloud Drive in the settings.

=====================================================

#### Saving data
We need to define a schema for record types we want to store on CloudKit. Go to the Signing and Capabilities tab on the project settings page and press the CloudKit Console button. It should open the browser with CloudKit dashboard, where you can find schema setup in the navigation menu. Press the record types button, and let's create a new one that we want to store and fetch.

=====================================================

In the example above, you see the simple Fasting value type that I want to store on CloudKit. CloudKit provides us CKRecord type representing items in the CloudKit database. Usually, we need to implement a converter from/to CKRecord for our custom types.

=====================================================

And now, we can finally create a form to populate fasting record data and save it to the private database of the current use on CloudKit.

=====================================================

#### Fetching data
Now we can learn how to fetch data from the CloudKit. First, we should update our schema by adding indexes marking fields in our records fetchable and sortable. Let's open the CloudKit dashboard and go to Schema -> Indexes. Here we should create indexes for all the fields we want to fetch or sort. 

=====================================================

In the example above, we implemented another converter from an instance of the CKRecord type. Let's move forward and implement a method on the CloudKitService type to fetch the records in the provided date interval.

=====================================================

And now we are ready to implement a view showing the fasting history.

=====================================================

#### Conclusion
Remember, CloudKit provides us with development and production environments. While developing an app and running it in debug mode, you automatically use the development environment. Before publishing the on TestFlight or App Store, you should deploy schema to the production environment in the CloudKit dashboard.

This week we learned the basics of storing and fetching data in the CloudKit. Now you know how to sync the data between the user devices. Next week we will learn how to implement data sharing between your app users via CloudKit.
