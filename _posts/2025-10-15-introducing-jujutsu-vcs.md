---
title: Introducing Jujutsu VCS
layout: post
---

I’ve decided to share my experience with Jujutsu VCS, which is not a martial art but a Git-compatible version control system. I’ve switched to Jujutsu VCS for all my projects and can talk a lot about it.

First and foremost, I must emphasize that it is a git-compatible version control system. This means you can use it with a git repository, and nobody in your team will notice that you’re using it, even if they continue using git.

For the rest of the article, I will refer to Jujutsu VCS as JJ. To install JJ follow these [instructions](https://jj-vcs.github.io/jj/latest/install-and-setup/). To start working with JJ you have to clone a git repository or init an empty one using next commands in terminal.

> jj git clone repo_url
or
> jj git init

Let’s talk about how JJ is different from git. In JJ we have changes instead of commits and they are mutable till you push them to the remote. It means you can easily move between changes and edit them. 

Instead of commiting your code, you create an empty change before you start writing the code. There is no staging area and JJ automatically stores everything you have changed. Change contain a bunch of snapshots of your code. Everytime you run JJ in the terminal it make a snapshot of your code and append to the recent change.

> jj new 

Whenever you fill that you have finished work on the current change and want to create another one you can use following commands.

> jj desc -m “describe the work you have done”
> jj new 

Changes simplify the history manipulations. Rebase, squash, and split are very basic operations here. Here is the log of my repo. Assume that I want to create a change between last two changes. Nothing can be easier.

> jj new -A px

The -A argument means after and it creates an empty change after the change with id px. You can also use -B to create an empty change before px. The great thing about creating changes in between that JJ automatically rebases next changes. This is my favorite thing about JJ, it automatically runs rebase when you change something in the history. JJ runs rebase in many situations and it almost never has conflicts.

> jj squash

Squash is another simple and powerful tool in JJ appends the recent change to the previous change.

> jj squash —into px

You can use to into argument to specify change that you want to squash into and it will automatically rebase next changes as needed.

> jj undo

Another great utility of the JJ is the undo command. You can always call undo to revert the most recent operation: snapshot, rebase, squash, merge, etc.

> jj new master
> jj new master

It is enough to create more than one change with the same parent to start branching in JJ. Branches in JJ anonymous which means they don’t have names. It might be strange first, but it works really well. You can set a bookmark on the change to create a git-compatible branch.

> jj bookmark set new-feature

Finally, whenever you need to delete a change you can run 

> jj abandon px

It will remove the change and its code then rebase the next changes.

JJ reimagines version control, making history manipulation effortless. It’s fully compatible with Git but offers a much more flexible and fluid workflow. I’ll share more advanced workflows—such as rebasing across branches—in future posts.
