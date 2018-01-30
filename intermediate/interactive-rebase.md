# How to safely rebase a messy feature branch and merge onto master

There are many times when you are developing a feature and you're just trying to figure things out. You're making a lot of changes, things are breaking and getting fixed, but you also want the ability to safely rewind to previous checkpoints so you're making lots of commits. However, you realize that a commit history of 70 commits that are minor changes with commit messages filled with curses is not going to be helpful to anyone, so you would like to cleanup your work before you merge in back to master.

We can do this by interactively rebasing our feature branch to squash the commits and then merging onto master.

>N.B.: This walkthrough assumes that you have defined the git aliases<sup id="ref1"><a href="#footnote1">1</a></sup> used in the ["basics" walkthrough](https://github.com/matthewfeickert/git-basics).

## Interactive rebase

First, it goes without saying that you should read the Git book sections on ["Rebasing"](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) and ["Rewriting History"](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History).

Okay, now let's walkthrough keeping everything you read in mind.

We start with our `master` branch, and our `feature` branch

```
$ git branch
* feature
  master
```

We have been working on our feature and add a few commits to `feature`

```
$ git glog
* 9a1bafc (HEAD -> feature) extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

We now do a lot of testing and make a bunch of commits to `feature` as we resolve the issue we're having

```
$ git glog
* 78c8cda (HEAD -> feature) Commit D
* d8334fd Commit C
* 731e142 Commit B
* 1171d25 Commit A
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

However, we realize that our commit messages are horrible and that all the changes we made really boil down to a few lines of code once implemented correctly. So our commit history isn't very helpful at this point. It would be nice to clean things up.

We can do this, but we know that `rebase` is a destructive command in that it **rewrites our commit history**. So we want to be very careful when using it. To be careful, we then just checkout a new branch from our `feature` branch

```
$ git checkout -b cleaned-feature
Switched to a new branch 'cleaned-feature'
```

### Squashing commits

Let's say that we want to combine commits `Commit A` through `Commit D` into a single commit that fixes our problem. To do this, we find the commit SHA of the commit immediately **before** the commit we want to rebase to. In our case, as we want to squash down our last 4 commits to just `Commit A` then the SHA of the commit before is `9a1bafc`

```
$ git glog
* 78c8cda (HEAD -> feature, cleaned-feature) Commit D
* d8334fd Commit C
* 731e142 Commit B
* 1171d25 Commit A
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

We then enter into an interactive rebase with that commit

```
$ git rebase -i 9a1bafc
```

> **Alternative:**
> We can also count the number of commits from (**and including**) `HEAD` to the commit that we want to `rebase` to
>
>```
>$ git glog
>* 78c8cda (HEAD -> feature) Commit D
>* d8334fd Commit C
>* 731e142 Commit B
>* 1171d25 Commit A
>* 9a1bafc extend out class X
>* ced2fea Add class X
>* 3deb85b (master) Add feature f
>```
>
> In our case, this is 4: `78c8cda` through `1171d25`. We then would enter
>
>```
>git rebase -i HEAD~4
>```

which brings us into our default text editor

```
pick 1171d25 Commit A
pick 731e142 Commit B
pick d8334fd Commit C
pick 78c8cda Commit D

# Rebase 9a1bafc..78c8cda onto 9a1bafc (4 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

> **N.B.:** Our commits are being shown to us in **reverse** order. So when you squash you're squashing _back_ in time which is _up_ in our view here.

We want to "pick" the commits that we want to keep, and then "squash" the commits that we won't want. So using our text editor commands we change the rebase log to be

```
pick 1171d25 Commit A
squash 731e142 Commit B
squash d8334fd Commit C
squash 78c8cda Commit D

# Rebase 9a1bafc..78c8cda onto 9a1bafc (4 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

In your text editor save and exit. As we have chosen "squash" we will be brought back into our default text editor to edit the commit log. By default the commits that were squashed have their own commit messages as new lines

```
# This is a combination of 4 commits.
# The first commit's message is:
Commit A

# This is the 2nd commit message:

Commit B

# This is the 3rd commit message:

Commit C

# This is the 4th commit message:

Commit D

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sat Jan 27 21:38:55 2018 +0100
#
# interactive rebase in progress; onto 9a1bafc
# Last commands done (4 commands done):
#    squash d8334fd Commit C
#    squash 78c8cda Commit D
# No commands remaining.
# You are currently editing a commit while rebasing branch 'cleaned-feature' on '9a1bafc'.
```

Using your text editor you can then edit the commit messages to be whatever you want. As we're trying to clean things up then we can just have there be a single commit message.

```
# This is a combination of 4 commits.
# The first commit's message is:
Patch bug alpha

Bug alpha existed which caused issues. This patch fixes it by doing
these things.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sat Jan 27 21:38:55 2018 +0100
#
# interactive rebase in progress; onto 9a1bafc
# Last commands done (4 commands done):
#    squash d8334fd Commit C
#    squash 78c8cda Commit D
# No commands remaining.
# You are currently editing a commit while rebasing branch 'cleaned-feature' on '9a1bafc'.
```

> **Alternative:**
> As we know that we don't care about the commit messages of the intermediate commits that we're rewriting with the `rebase` we can pick the "fixup" command instead
>
>```
>pick 1171d25 Commit A
>fixup 731e142 Commit B
>fixup d8334fd Commit C
>fixup 78c8cda Commit D
>
># Rebase 9a1bafc..78c8cda onto 9a1bafc (4 command(s))
>#
># Commands:
># p, pick = use commit
># r, reword = use commit, but edit the commit message
># e, edit = use commit, but stop for amending
># s, squash = use commit, but meld into previous commit
># f, fixup = like "squash", but discard this commit's log message
># x, exec = run command (the rest of the line) using shell
># d, drop = remove commit
>#
># These lines can be re-ordered; they are executed from top to bottom.
>#
># If you remove a line here THAT COMMIT WILL BE LOST.
>#
># However, if you remove everything, the rebase will be aborted.
>#
># Note that empty commits are commented out
>```
>
> Upon save and exit we have the same commit message that we started with
>```
>$ git rebase -i 9a1bafc
>[detached HEAD 4fbeb63] Commit A
> Date: Sat Jan 27 21:38:55 2018 +0100
> 1 file changed, 4 insertions(+)
> create mode 100644 test.txt
>Successfully rebased and updated refs/heads/cleaned-feature.
>```
>Which can then be edited with `commit --ammend` to also have our helpful commit message.

We then save and exit into a detached `HEAD` state

```
$ git rebase -i 9a1bafc
[detached HEAD 4fbeb63] Patch bug alpha
 Date: Sat Jan 27 21:38:55 2018 +0100
 1 file changed, 4 insertions(+)
 create mode 100644 test.txt
Successfully rebased and updated refs/heads/cleaned-feature.
```

```
$ git glog
* 623bf11 (HEAD -> cleaned-feature) Patch bug alpha
| * 78c8cda (feature) Commit D
| * d8334fd Commit C
| * 731e142 Commit B
| * 1171d25 Commit A
|/  
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

### Working by yourself

We can now checkout `master` and merge our feature in

```
$ git checkout master
Switched to branch 'master'
$ git merge cleaned-feature
Updating 3deb85b..623bf11
Fast-forward
 <usual stuff that happens here>
```

```
$ git glog
* 623bf11 (HEAD -> master, cleaned-feature) Patch bug alpha
| * 78c8cda (feature) Commit D
| * d8334fd Commit C
| * 731e142 Commit B
| * 1171d25 Commit A
|/  
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b Add feature f
```

Now that our feature is in `master` we can safely remove both the `feature` branch and the `cleaned-feature` branch.

### Working with a team

If you are working on a team then you probably don't want to merge into `master`, as you are most likely going to be pushing your feature branch up to GitHub for pull request and code review. In that case you can use the [`--strategy=ours` feature in `merge`](https://git-scm.com/docs/git-merge#git-merge-ours) to force a rewrite of `feature` with `cleaned-feature`

```
$ git merge -s ours feature
Merge made by the 'ours' strategy.
$ git checkout feature
Switched to branch 'feature'
$ git merge cleaned-feature
Updating 78c8cda..a3ab52d
Fast-forward
```

```
$ git glog
*   a3ab52d (HEAD -> feature, cleaned-feature) Merge branch 'feature' into cleaned-feature
|\  
| * 78c8cda Commit D
| * d8334fd Commit C
| * 731e142 Commit B
| * 1171d25 Commit A
* | a0aa59f Patch bug alpha
|/  
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

As the latest commit doesn't really need to exit as it is an artifact of the merge you can also just do a `reset --hard` to the patch commit

```
$ git reset --hard a0aa59f
HEAD is now at a0aa59f Patch bug alpha
$ git glog
*   a3ab52d (cleaned-feature) Merge branch 'feature' into cleaned-feature
|\  
| * 78c8cda Commit D
| * d8334fd Commit C
| * 731e142 Commit B
| * 1171d25 Commit A
* | a0aa59f (HEAD -> feature) Patch bug alpha
|/  
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

Now that your `feature` branch is cleaned up and reflecting your work you can push to GitHub and remove the WIP status from your pull request. Once the PR is accepted then you can of course remove the branches `feature` and `cleaned-feature`.

For reference this is what the commit graph looks like after `cleaned-feature` is removed

```
$ git glog
* a0aa59f (HEAD -> feature) Patch bug alpha
* 9a1bafc extend out class X
* ced2fea Add class X
* 3deb85b (master) Add feature f
```

---

<a name="footnote1">1</a>) `glog` [is an alias](https://github.com/matthewfeickert/git-basics/blob/master/README.md#helpful-aliases) for `git log --graph --oneline --decorate --all` [↩](#ref1)
