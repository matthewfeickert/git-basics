# How to sync a fork with an upstream repository

If you are working on a project where you do not have direct commit permissions (e.g., you're not a core developer or for organizational reasons) then you will be working with a fork of the main project repository.
While you are working you will want to keep your fork up to date ("synced") with the "upstream" repository.
Here we will start assuming that you have already forked the project repository and have cloned the fork to your local working machine.

## Adding the main project repository as a remote (`upstream`)
First, the main project repository (the `upstream` repository) needs to be added as a remote to your local repo.
To do this, you need to get the address of the main project (which is just the address that you would use to clone it).
We can start by first seeing what remotes we have

```
$ git remote -v
origin	git@github.com:user/fork-of-project-name.git (fetch)
origin	git@github.com:user/fork-of-project-name.git (push)
```

and then adding the main project repository as a remote

```
$ git remote add upstream git@github.com:project-org-name/project-name.git
```

Here we chose to name the remote `upstream` but we could have named it anything (`upstream` is just [common](https://stackoverflow.com/questions/2739376/definition-of-downstream-and-upstream)).
If we now check our remotes again we can see that we have successfully added the `upstream` repo

```
$ git remote -v
origin	git@github.com:user/fork-of-project-name.git (fetch)
origin	git@github.com:user/fork-of-project-name.git (push)
upstream	git@github.com:project-org-name/project-name.git (fetch)
upstream	git@github.com:project-org-name/project-name.git (push)
```

this can also be seen by checking what branches now exist

```
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/upstream/master
```

## Syncing the local repository with the `upstream`

Now that the main project repo is added as a remote for our local repo we can `fetch` the `upstream` state

```
$ git fetch upstream master
```

assuming that there are changes that have been `fetch`ed we would now want to `merge` those changes into our local repo

```
$ git merge upstream/master
```

This could of course also have been done in one go if we knew that we wanted to accept everything (which you probably do) with

```
$ git pull upstream master
```

## Pushing to the fork remote (`origin`)

Now the fork repo on your local machine is up to date with the `upstream` repo, but we still need to `push` these changes to the fork remote (which has so far been assumed to have the name `origin`).
This is easily seen by `git status` but if you are confused as to why this is necessary simply check the output of `git glog`<sup id="ref1"><a href="#footnote1">1</a></sup>.
To fix this we just need to `push` the changes on the local machine to the fork remote

```
$ git push origin master
```

Your fork is now fully in sync with with the main project repository.
If you visit the web view of your fork (on something like GitHub or GitLab) then there should even be a message displayed telling you this.

## Keeping your fork synced

In the future to keep everything synced just repeat the `fetch` and `merge` of the `upstream` with your local repo and then `push` changes to your fork remote.

In brief:
1. `fetch` and `merge` changes from the main project repository into your local repository of your fork
   - `git pull upstream master`
2. `push` these changes to your fork's remote
   - `git push origin master`

## Additional references
- [GitHub's "Syncing a fork" reference](https://help.github.com/articles/syncing-a-fork/)

---

<a name="footnote1">1</a>) `glog` [is an alias](https://github.com/matthewfeickert/git-basics/blob/master/README.md#helpful-aliases) for `git log --graph --oneline --decorate --all` [↩](#ref1)
