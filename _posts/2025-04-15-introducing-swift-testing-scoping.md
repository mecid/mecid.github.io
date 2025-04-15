---
title: Introducing Swift Testing. Scoping
layout: post
category: Testing
image: /public/testing.png
---

Apple recently released Swift 6.1, with most of the changes being cosmetic. However, I particularly like the scoping feature introduced in the Swift Testing framework. This week, we’ll delve into the new Test Scoping feature and explore how to effectively utilize it in Swift.

We already talked about test lifecycle in Swift Testing framework in previous posts. We can utilize init and deinit methods on class types to define setup and teardown function using Swift Testing framework.

======================================================

Assume that you have a bunch of test cases depending on the same ModelContainer, or predefined data. I means you have to write initialization code again and again for every test case. Fortunately, the Swift Testing framework solves it for us by introducing test scoping functionality. Test scoping works in pair with testing traits. Let me revise you memory with the example of testing traits.

======================================================

As you can see in the example above, we can enable or disable particular test or test suites using testing traits. With the most recent update of the Swift Testing framework we can go further and annotate with test scopes allowing us to run custom code before and after the test suite or test functions execution.

It means, the test scoping features allows us to build powerful and reusable mechanisms allowing us to customize the test or test suite environment.

======================================================

As you can see in the example above, we define the Environment struct holding all the app dependencies. We also define the task local value for the actual instance of the environment. Task local values is not only a great way of providing shared information with default values, but it also allows us to easily replace one value with another when needed.

In our example, we use task local values with default production-ready dependencies, but our final goal is to provide a mocked version of the environment for testing purposes.

======================================================

Here we define the MockEnvironmentTrait type conforming to the TestTrait, SuiteTrait and TestScoping protocols. The TestScoping protocol has the only requirement which is the provideScope function. It has a few parameters providing you information about calling test function and test suite.

The last parameter is the performing function that describes the particular test function or the whole test suite. You should run the performing function to allow test function or test suite execution.

In our example, we run the performing function inside the closure that overrides the task local value. This technique allows us to run the test suite or test function and provide it mocked environment.

