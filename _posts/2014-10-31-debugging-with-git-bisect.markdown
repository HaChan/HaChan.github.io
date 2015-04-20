---
layout: post
title:  "Debugging with Git Bisect"
date:   2014-10-31
tags: [git]
---

Git bisect is used when bugs has introduced in the source code but you don't know what causes it, and there are a lot of commits since the last working state where you know the code worked. This is where the command `git bisect` comes handy. The `git bisect` command works by performing a "binary" search between one state (commit) of the source code (tree) and the other in your commit history in order to find which commit introduced the problem.

Let's assume that when develop a product, everything has worked fine until suddenly, your code has some major bug. Your source code now is too big and it's hard to find what caused it or which commit has caused it. Now, you might want to find which commit has introduced the bug to see what changes has caused it to fix the bug. Here's how to use `git bisect` to find the bugging commit, step by step:

###Step 1: Find a working, non-bug commit.

First, have a look through the git commit history by using `git log`, or `git log --oneline` for short and easy to see. Thus,it will produce something like this:

```
"7 character of the commit hash"   "commit message"
```

Now, to find a commit that actually work, you just choose a commit and experiment with it to see if it works or not by switching to that commit through:

```
git checkout "commit hash (or just 5 - 6 character of git commit hash)
```

If you can find the working commit, go to the next step:

###Step 2: Use git bisect to find the problem commit.

To begin, you need to start git bisect with:

    git bisect start

Then, you use:

    git bisect bad

to tell the git system that the current commit you're on is broken. Then, you use:

    git bisect good "commit hash"

to tell git the last commit state that work fine (through step 1). Or you might want to use a different approach by assign good and bad commit hash to git bisect good and bad so that git can find the problem commit between these two commmit:

```
git bisect good "commit hash good"
git bisect bad "commit hash bad"
```

Git bisect looked at all of the commits between good and bad commit hash, pick the middle one and then switched the current check out to that commit instead. Now, you can run the code or test to see if the problem is there. If it's not, you tell git that this commit is good by typing:

    git bisect good

else you tell git that this commit is bad by typing:

    git bisect bad

Then, you continue this process until there is one commit left with git bisect. If that commit is the problem one, so congrats, you found it, else if that commit is good then the previous of that commit is the problem one.

Once you're all finished up, you should run

    git bisect reset

to reset your HEAD to get back to the normal working branch.

This is a powerful tool that can help you check hundreds of commits for an introduced bug in minutes. In fact, you can automate this process by doing so:

```
git bisect start HEAD v1.0
git bisect run test-error.sh
```

Doing so automatically runs test-error.sh on each checked-out commit until Git finds the first broken commit.

Source: [git-scm](http://git-scm.com/book/en/Git-Tools-Debugging-with-Git), [Using Git bisect to figure out when brokenness was introduced](http://webchick.net/node/99)
