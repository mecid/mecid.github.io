---
title: Introducing Jujutsu VCS
layout: post
category: Meta
image: /public/jj.svg
---

I’ve decided to share my experience with Jujutsu VCS, which is not a martial art but a Git-compatible version control system. I’ve switched to Jujutsu VCS for all my projects and can talk a lot about it.

{% include friends.html %}

First and foremost, I must emphasize that Jujutsu VCS is a Git-compatible version control system. This means you can use it with a Git repository, and nobody on your team will notice that you’re using it, even if they continue using Git.

For the rest of the article, I will refer to Jujutsu VCS as JJ. To install JJ, follow these [instructions](https://jj-vcs.github.io/jj/latest/install-and-setup/). To start working with JJ, clone a Git repository or initialize an empty one using the following commands in the terminal.

> jj git clone repo_url

or

> jj git init

Let’s talk about how JJ is different from Git. In JJ, we have changes instead of commits, and they remain mutable until you push them to a remote. That means you can easily move between changes and edit them.

Instead of committing your code, you create an empty change before you start writing code. There is no staging area, and JJ automatically tracks everything you modify. A change contains a series of snapshots of your code. Every time you run JJ in the terminal, it makes a snapshot of your code and appends it to the most recent change.

> jj new

Whenever you feel that you have finished work on the current change and want to create another one, you can use the following commands.

> jj desc -m "describe the work you have done"

> jj new

Changes simplify history manipulation. Rebase, squash, and split are very basic operations here. Here is the log of my repo. Assume that I want to create a change between the last two changes. Nothing could be easier.

![jj-terminal](/public/jj1.png)

> jj new -A px

The -A argument means **after**, and it creates an empty change after the change with ID `px`. You can also use -B to create an empty change **before** `px`. The great thing about creating changes in between is that JJ automatically rebases subsequent changes. This is my favorite thing about JJ: it automatically runs a rebase when you change something in the history. JJ runs rebases in many situations, and it almost never results in conflicts.

![jj-terminal](/public/jj2.png)

> jj squash

The squash command is another simple and powerful tool in JJ; it appends the most recent change to the previous change.

> jj squash --into px

You can use the --into argument to specify the change that you want to squash into, and JJ will automatically rebase subsequent changes as needed.

> jj undo

Another great utility in JJ is the undo command. You can always call undo to revert the most recent operation—snapshot, rebase, squash, merge, etc.

> jj new master

> jj new master

Creating more than one change with the same parent is enough to start branching in JJ. Branches in JJ are anonymous, which means they don’t have names. It might be strange at first, but it works really well. You can set a bookmark on a change to create a Git-compatible branch.

> jj bookmark set new-feature

Finally, whenever you need to delete a change, you can run:

> jj abandon px

It will remove the change and its code, then rebase the subsequent changes.

JJ reimagines version control, making history manipulation effortless. It’s fully compatible with Git but offers a much more flexible and fluid workflow. I’ll share more advanced workflows—such as rebasing across branches—in future posts. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
