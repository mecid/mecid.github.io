---
title: Zone sharing in CloudKit
layout: post
---

Last week we talked about the basics of CloudKit. We learned how to save and fetch data from the cloud storage and how to sync the data between devices. This week I want to cover the only reason why I have chosen CloudKit instead of Firebase, and it is data sharing between users.

CloudKit provides you ready to use data sharing API that allows you to implement collaborative features of your app without much effort. There are two ways to share data via CloudKit: record sharing and zone sharing. In this post, we will talk about zone sharing.

A zone is a defined bucket inside the private database of the current user. You can create and assign a zone to any record in the database. You share all the records tagged with the zone by sharing the zone itself. For example, in the todo app, you can create a zone with the name of the todo list and share it with your family.

First of all, we need to create and save a zone in the private database of the current user. Then we have to assign it to a particular record and save or update it in the private database.

=====================================================

Now we have a zone saved in the private database and associated records. But to start sharing, we should create an instance of CKShare type and save it into the private database.

=====================================================

As you can see in the example above, we create a new CKShare object and set the public permission to read-only. The default value is none which blocks anyone from accessing your data without your approval.

Remember that you can have only one instance of CKShare per zone in the database. You need to check if you already have one and fetch if it is available. The next step is to present an instance of UICloudSharingController with the created CKShare object.

=====================================================

UICloudSharingController provides you with all the needed functionality to add and manage participants of the share. You can also configure available options of the UICloudSharingController instance by setting the value of the availablePermissions property to allowReadOnly, allowPrivate, allowReadWrite, allowPublic. Now let's talk about how we should implement share accepting functionality.

Before writing any code, we should add the CKSharingSupported boolean key with the value YES to the Info.plist. Whenever a user receives and opens a link shared via UICloudSharingController, the userDidAcceptCloudKitShareWith delegate method will be called by the system. Here we can call the accept method to approve the sharing.

=====================================================

Finally, we can use the shared database to fetch the content of shared zones.

=====================================================

Remember that you have to fetch all the zones from the shared database. Even when they have the same name, they have different owners. We can enumerate all the shared zones and fetch shared records in every zone.
