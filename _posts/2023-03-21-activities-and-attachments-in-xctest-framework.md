---
title: Activities and attachments in the XCTest framework
layout: post
---

I love UI testing and protecting the primary user flows in my app. Today I want to continue the topic of UI testing in Swift by covering another great set of tools the XCTest framework provides us. This week we will talk about the hidden gems of the XCTest framework, which are activities and attachments.

#### Activity
Let's start with the concept of the XCTest activity. When you run a UI test in the Report navigator section of the Xcode, you can find the log of every action you run in the particular UI test.

=========================image========================

It is excellent that Xcode logs every action, and we can review them later, but it is hard to understand who is who here. And for this particular case, we can use XCTest activities. An XCTest activity allows us to group and name a set of actions inside the test. Let's take a look at how we can use this API.

====================================================

As you can see in the example above, we use the runActivity function on the XCTContext type. This function accepts two parameters: the name of the activity and the closure defining the activity. It allows us to easily group and name actions in the test and review them in the report navigator much more easily.


===========================image======================

#### Attachment
The second great feature of the XCTest framework is the concept of attachments. You can add attachments to any test case or activity. For example, it might be a screenshot of the current state of your app while running a particular action. Let's see how we can use the XCTest activity to add some screenshots.

=====================================================

I'm a big fan of the Page Object pattern and build my UI tests suite using it. I define the Screen protocol that every Page Object in my app conforms to. It contains many useful functions, one of which is the takeScreenshot function.

As you can see in the example above, we use the XCTContext to run an activity and use the add function on the XCTActivity type to attach the screenshot. But keep in mind that the XCTAttachment type allows us to attach screenshots and any instance of String and Data types or the content of a file.

And finally, we can review and save all of these attachments in the Report navigator section of the Xcode. It allows us to do many exciting things. For example, you can run a UI tests suite that goes through the main flows of your app and collects screenshots for the App Store page. And you can run it under different locales to automate collecting app screenshots.
