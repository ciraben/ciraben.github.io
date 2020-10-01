---
layout: post
title:  "8 Git Tips to Get You Going"
date:   2020-09-28 16:00:00 -0700
categories: 
background: '/images/tapas.jpg'
---

Sometimes, in between projects, I lose track of all the Git tips I've learned. Here are some of my faves.

### The `master` branch

Tired of seeing `master` all over your projects? As Scott Hanselman suggests, here's a one-time fix to change the term to `main` (or `latest`, or whatever): 
```bash
$ git config --global init.defaultBranch main
```
Check [his article](https://www.hanselman.com/blog/EasilyRenameYourGitDefaultBranchFromMasterToMain.aspx) for details on amending projects in progress.

### Making a new repository

Here's the standard:
```bash
$ git init
$ touch .gitignore
$ git add .
$ git commit
```
(Since I edit in vim, I'll also add `*.swp` to `.gitignore` right off the bat.) 
Next, make a new repository on Github, grab the generated address and:
```bash
$ git remote add origin ADDRESS
$ git push -u origin main
```
If you're ready to start versioning, you can.
```bash
$ git tag -a v0.1.0
$ git push origin --tags
```
There's instructions for setting up a tag as a Github release later in this post.

### Working on an Established Repository

One feature that makes Git so powerful is branching -- the ability to put down our work on one feature & pick up work on another without hassle.

For example, we're writing a new post for our nifty tech blog, and we come across a typo in an old post.
By checking out a new Git branch, we can put our writing on hold to fix the typo & push a quick update without pushing our unfinished post.
This gets even handier when the features in question impact multiple (or overlapping!) files.
Essentially, this is the Git feature which facilitates teamwork best, and it's handy on your own too.
What does this look like in practice?

Let's illustrate a typical workflow. Say we have a stable project established on `main` and we want to work on a new (possibly build-breaking) feature. First, check out a new branch.

```bash
$ git checkout -b new-feature
```

We can see we're now on the `new-feature` branch - any commits will be recorded here instead of on `main`.

```bash
$ git branch
  main
* new-feature
```

After making some commits and successfully testing our new feature, we can add it to our `main` project.

```bash
$ git checkout main
$ git merge new-feature
```

If we're testing a new Github Pages site addition, or we're collaborating with others, we can push `new-feature` to a new remote branch:

```bash
$ git checkout new-feature
$ git push
```

To also set `new-feature` to track its remote counterpart,

```bash
$ git push -u origin
```

(This ensures our local `new-feature` is updated with colleagues' pushes whenever we pull/fetch from `remote`.)

Once we've merged our changes with `main` and we're all done with the development branch, we can delete it.

```bash
$ git branch -d new-feature
```

To delete a remote branch,

```bash
$ git push origin --delete new-feature
```

To summarize the workflow:
* start with a stable `main` branch
* make a development branch
* commit changes to it
* when the feature is complete, merge with `main`
* optionally push the development branch
* delete the development branch

### Submitting a Pull Request

The preceding workflow is also the basic process for contributing to open-source projects on Github, with a few adjustments:
* Start by forking the project repository. This gives you a remote copy to push to.
* Clone the repository locally & follow the above workflow.
* Instead of merging with local `main`, push the development branch.
* When your work is done, navigate to your remote development branch on Github and open a "new pull request", under the Pull requests tab. It'll ask for a description of the feature & begin a conversation thread on the original repository's page.

Of course, that's just the beginning, so maybe don't delete your local branch quite yet. The project owner or community may suggest additional changes to you before agreeing to merge your feature.

### Git Tagging

In Git, we use tags to flag notable commits in a project. Use `git tag` to check existing tags on a project. There are two different kinds of tags.

**Lightweight tags** (`git tag TAGNAME`) just label a specific commit. From the [Git website](https://git-scm.com/book/en/v2/Git-Basics-Tagging),
>"[They are] basically the commit checksum stored in a file — no other information is kept."

**Annotated tags** store a bit more info. It sounds like they're the most useful for versioning on Github. To declare a new version using a tag: 
```bash
$ git tag -a v1.4 -m "version 1.4"
```
This adds the tag to the most recent commit (of the branch you're working on locally). If you want to declare something a few commits ago to be a version, use:
```bash
$ git log --oneline
f6face4 (HEAD -> main, tag: v0.0.1, origin/main) Commit license.
$ git tag -a v1.4 f6face4
```

### Releasing a Version to Github

To push a new release, normally we'll push our commits to `main` as well, to advertise our new release on our project's main page:
```bash
$ git push
```
Either way, we push our commits to the new version's tag:
```bash
$ git push origin v1.4
```
or, if you like typing the same thing every time,
```bash
$ git push origin --tags
```
This will update any and all tags you've fiddled with though, including lightweight tags. To update annotated tags only, use 
```bash
$ git push origin --follow-tags`
```

To display your version as a release:
1. Go to the repository's main page on Github.
2. In the righthand panel under Releases, a text link "Create a new release" is displayed - click it!
3. From the Releases page, on the righthand side, click "Draft new release".
4. Begin typing your version's tag in the "Tag version" field, and select it from the dropdown menu. (Handily, if all your versions start with v, you can type `v` and see them all!)
5. Optionally enter a title & description for this version.
6. Scroll down & click "Publish release", and voilà!

Your new version will be displayed on your repository's main page in the righthand panel under Releases, or at the bottom of the main page, after the README. Github auto-generates `.zip`/`.tar.gz` distributions of each release version, downloadable from the releases page.

### Getting Around in Git

These commands help me observe my git "surroundings":

* `git tag` lists all the tag names on your project.
```bash
$ git tag
v0.0.1
v0.1.0
v0.1.1
```
* `git branch -avv` lists remote & local branches, and what they track.

  ```bash
  $ git branch -avv
    main                    c2f592e [origin/main] Minor adjustments.
  * new-feature             c2f592e Minor adjustments.
    remote                  c2f592e Minor adjustments.
    remotes/origin/gh-pages 8003501 jekyll build from Action c2f592e...
  ```

* `git log --oneline` lists your commits with tags & origin labeled.
```bash
$ git log --oneline
237406c (HEAD -> main, origin/main) Rearrange README contents.
002cae5 Add info on releasing versions to Github.
36e18ad (tag: v0.1.1) Change faulty code block in README.
758c7ef (tag: v0.1.0) Update changelog.
2832746 Commit changelog.
fc718b4 Commit .gitignore.
1623d3c Commit a readme.
f6face4 (tag: v0.0.1) Commit license.
```
* `git describe` shows where you are with respect to most recent tag, in the form `V-N-H`, where `V` is the most recent version/tag name, `N` is the number of commits since then, and `H` is the last commit's hash ID. Like this:
```bash
$ git describe
v0.1.1-2-g237406c
```

* `git status` shows which files have been changed since last commit.

  ```bash
  $ git status
  On branch main
  Your branch is up to date with 'origin/main'.

  Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
          modified:   README.md

  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
          modified:   README.md
  ```
* `git diff` shows line-by-line details of those changes.

  ```bash
  $ git diff
  diff --git a/README.md b/README.md
  index 190ea13..a75d153 100644
  --- a/README.md
  +++ b/README.md
  @@ -1,15 +1,15 @@
   # Learning How Git Versioning Works

  -Use `git tag` to check existing tags on a project.
  +In Git, we use tags to flag notable commits in a project. Use `git tag` to check existing tags on a project. There are two different kinds of tags.

  -Lightweight tags (`git tag TAGNAME`) just label a specific commit. They are
  ->"basically the commit checksum stored in a file — no other information is kept."
  +Lightweight tags (`git tag TAGNAME`) just label a specific commit. From the [Git website](https://git-scm.com/book/en/v2/Git-Basics-Tagging),
  +>"[They are] basically the commit checksum stored in a file — no other information is kept."

  -Annotated tags sound like they're the most useful for versioning with github.
  +Annotated tags store a bit more info. It sounds like they're the most useful for versioning with github. To declare a new version using a tag, do:
  ```

### Odds and ends

To edit the most recent commit message before pushing:

```bash
$ git commit --amend
```

However, this adds our unstaged changes to the commit as well.
More generally, we can "tidy up" small local commits (typos, etc.) by refactoring them before pushing. To rewind the last N commits,

```bash
$ git reset --soft HEAD~N
```

After soft resetting, our commit messages are deleted but all our changes remain, unstaged. From here, we can recommit them all at once,

```bash
$ git commit -a
```

or we can add files manually and re-commit them in whatever grouping makes sense to us.

```bash
$ git add FILE1
$ git commit -m "FILE-SPECIFIC MESSAGE"
$ git add FILE2 FILE3
$ git commit -m "ANOTHER MESSAGE"
```

To move a tag to the most recent commit:
```bash
$ git push origin :refs/tags/TAGNAME
$ git tag --force -a TAGNAME
$ git push origin main --tags
```

One last tidbit — please be careful with this one! Unlike other Git commands, `reset --hard` deletes changes for good. However, if we've made a mess of our project files and just want to start fresh, we can.

```bash
$ git reset --hard HEAD
```

This discards all changes since the last commit (or whichever commit `HEAD` is on), including changes staged for commit AND unadded changes.

And that's it for now! Thanks for reading. :)

## References

* [Git Basics](https://git-scm.com/book/en/v2/)
* [marco.marinangeli on Stack Overflow](https://stackoverflow.com/questions/37814286/how-to-manage-the-version-number-in-git#46434732)
* [The Stack Overflow community wiki](https://stackoverflow.com/questions/179123/how-to-modify-existing-unpushed-commit-messages)
* [Greg Hewgill on Stack Overflow](https://stackoverflow.com/questions/8044583/how-can-i-move-a-tag-on-a-git-branch-to-a-different-commit)
* [Scott Hanselman](https://www.hanselman.com/blog/EasilyRenameYourGitDefaultBranchFromMasterToMain.aspx)
