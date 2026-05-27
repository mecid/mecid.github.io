---
title: Swift defer: Clean Up Before You Leave
layout: post
---

You may think about defer keyword as one of the most ambiguous language features in Swift, but it is very useful in some cases. You can use it deliberately, and it will give you safety. This week we will talk about some best practices of using defer in Swift.

Defer keyword in Swift allows you to run a block of code at the end of the current scope. What does current scope mean? Usually, it is the nearest curly braces pair. Let’s take a look at a few examples.

=======================================================

As you can see in the example above, every open curly brace defines a new scope. It might be a function, for loop, calculated property, etc. So, the defer keyword defined inside a particular scope affects only this scope and runs just before the end of the scope.

=======================================================

Here I try to show where defer blocks should execute. Every closing curly brace is closing some scope, and this is exactly the place where defer blocks run.

So, you can place defer blocks anywhere in the scope, but they will run at the end of the scope. Why might we need so ambiguous behaviour? We used to read and understand the code line by line, but defer blocks are different.

I’m sure you don’t need to use defer blocks often, but there are some cases where they shine.

=======================================================

As you can see, we create a file in the temporary directory, and we should delete it as soon as we finish work with it. You can create that file and easily forget to remove it after all. Believe me, that’s happened more often than you might think.

=======================================================

By using a defer block, you can create a file and define a defer block that deletes it. You can place it right below the file creation, but it will execute at the end of the scope.

=======================================================

Let me show you another example. Here we use HealthKit observation API, that requires you to call a completion handler as soon as you finish your processing. This is another interesting usage of a defer block, because you define it at the start of the scope, but it will run in the end right after you finish all your processing.
