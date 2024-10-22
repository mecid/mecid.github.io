---
title: Introducing Swift Testing. Basics.
layout: post
---

Swift Testing is a new framework with expressive and intuitive APIs that improve your testing experience. It is powered by macros that allow you to organize and assert your tests. This week, we will learn about the basics of the Swift Testing framework and how we can use it side-by-side with XCTest.

To start using the Swift Testing framework, we only need to import it. Then, we will gain access to all macro types provided by the framework.

=====================================================

As you can see in the example above, we use the @Test macro to annotate our verifyAdd function. You don't need to name your test functions with any prefix; you only need to annotate tests with the @Test macro.

You can annotate the functions with @Test macro throwing and async. Throwing tests will fail whenever an unhandled error appears.

The expect macro allows you to assert the values in your tests. It replaces the whole collection of the assert functions with a single one. You place the boolean expression inside the expect macro, which will pass whenever it is true and fail whenever it is false.

=====================================================

You can use the expect macro to verify any boolean expression that you can imagine. As you can see, it replaces many of the XCAssert* functions that the XCTest framework introduced.



Assume that you have a throwing function that might throw an error in a particular case and want to test that behavior. You can still use the expect macro to verify the error throw.

=====================================================

Another overload of the expect macro allows us to verify the function's throwing behavior and inspect the error itself to confirm that it is the error case you actually desire.

=====================================================

Another macro introduced by the Swift Testing framework is the require. The require macro has the same API as the expect macro with a single difference. It is a throwing function that throws an error as soon as the boolean expression you pass into it is false. It allows us to stop the test when a required condition doesn't meet our expectations.

=====================================================

Another way to stop the test and document the issue is to use the record function of the Issue type. This allows us to log the issues in the Test Navigator.

The Swift Testing framework integrates well with Xcode, providing a rich, inline representation of test results. It also provides a new way to organize, group, and structure test suites, which I will cover in the next few posts.
