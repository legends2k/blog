+++
title = "Git Wizardry"
description = "Obscure but useful Git incantations"
date = 2020-05-13T19:56:56+05:30
tags = ["tech", "vcs", "tools"]
toc = true
+++

Git has its oddities with unusual commands and flags.  I keep learning something new every now and then.

A decent understanding of [basic Git concepts][git-concepts] is a prerequisite.

[git-concepts]: {{< relref "git_concepts.md" >}}

# `show` time!

The easiest way to see the diff introduced by a [reference][git-concepts-ref] (fancy name for commit) is not `git diff`!

{{< highlight basic >}}
git show REF

# changes introduced in a particular file by a commit
git show REF -- path/to/a/file
{{< /highlight >}}

It shows details about the commit along with the changes it brings about.  Of course, you can control the amount of information thrown at you with the usual options `show` shares with `log`: `--name-status`, `--shortstat`, `--stat`, `--summary`, etc.

`show` can also give you a file’s clean copy from a commit --- very handy:

{{< highlight basic >}}
git show REF:path/to/a/file > file_copy
{{< /highlight >}}

`show` is a versatile command that can explain _any_ [Git object][git-object], not just references.

[git-concepts-ref]: {{< relref "git_concepts.md#references" >}}
[git-object]: https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

# Man’s search for ~~meaning~~ config 😛

Ever been annoyed by a Git setting you didn’t set; clueless about where it’s coming from?

{{< highlight cfg >}}
git config --show-origin  --list

file:/usr/local/etc/gitconfig       credential.helper=osxkeychain
file:/Users/my_user/.gitconfig      user.name=John Q. Public
file:/tmp/Cool_Project/.gitconfig   core.pager=less --quit-if-one-screen --no-init
{{< /highlight >}}

lists all settings along with the corresponding `.gitconfig` it’s coming from.

# You're on stage!

`git add` has some [good tricks][git-add-variations] up its sleeves:

{{< highlight basic >}}
git add .    # stage changes (untracked and tracked)
git add -u   # stage changes (tracked-only)
{{< /highlight >}}

`-n` dry runs `add` without actually staging anything; `-v` makes `add` tell what is staged.  If you want to stage only parts of a file, pass `-p` to interactively stage hunks of patch.

If you’re sure of your changes and just want to `commit`, skipping the intermediate `add`

{{< highlight basic >}}
git commit -a
{{< /highlight >}}

This will auto-`add` all tracked, changed files and `commit`.

[git-add-variations]: https://stackoverflow.com/q/572549/183120

# Stuff that matters!

{{< highlight basic >}}
git ls-files [<pathspec>]
{{< /highlight >}}

lists only tracked files of the repository; it optionally takes a [pathspec][] argument.  Quite useful when you’ve to separate the wheat from the chaff in a directory with many untracked files.

[pathspec]: https://stackoverflow.com/q/27711924/183120

# Pull another branch

Contrary to popular belief, [`git pull` can be run on a branch that’s not current][pull-non-HEAD].  Say `HEAD` is attached to `master`, you can still pull in changes to `my-topic` branch from a remote.

{{< highlight basic >}}
git fetch origin my-topic:my-topic
{{< /highlight >}}

If `my-topic` and `origin/my-topic` have diverged, this gets reduced to an ordinary `fetch` i.e. only `origin/my-topic` gets updated.

[pull-non-HEAD]: https://stackoverflow.com/a/42902058/183120

# Please [mind the dots][mind-gap]

`git diff A..B` = diff [(`A`, `B`\]][intervals] i.e. the diff includes changes made by `B` but not by `A`.  Since a diff-ing utility just takes a pair of file sets, with these references we simply denote the repository at different states -- after committing `A` and after committing `B`.  This diffs the repository at _two points_.

{{< highlight bash >}}
git diff A B    # same as below; only nicer
git diff A..B   # diff commits A and B
git diff A...B  # diff commits ancestor(A, B) and B
{{< /highlight >}}

`git diff A...B` = `git diff $(git merge-base A B) B` i.e. difference between the common ancestor of both references and `B`.  `A` is usually `HEAD` and `B` is commonly a branch head, so this shows work done independently in a branch.  _Memory aid_: triple dots ≈ branch.

However, the [meanings feel reversed for `git log`][log-dots]!  🤦  Also `A..B` and `A B` mean the same in `diff` while not in `log`!  Before reading `log`’s nuances, remember that `log` operates on a range of commits while `diff` only does on two.  [Gitolite’s nice observation][triple-dot] might also help you internalise `...` better:

> `...` somehow involves the common ancestor for both `diff` and `log`

{{< highlight bash >}}
git log A B    # (1) show A ∪ B commits (till root)
git log A..B   # (2) show B-only commits
git log A...B  # (3) show A-only and B-only commits
{{< /highlight >}}

`log` takes a commit and lists all the way to its root, unless an end point is given.  This explains (1); for (2) since `A` wouldn’t be reachable from `B` it stops at the common ancestor.  (3) has `...` so it involves the common ancestor; since both `A` and `B` can be reached from it, it lists both’s history up-to the ancestor.

[Pro Git v2 explains][pro-git-log] these in `git log`’s context.

[pro-git-log]: https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection
[triple-dot]: https://gitolite.com/tips-2.html#how-to-remember-what-...-does

# Overloaded `checkout`

Keep getting confused between the various tasks that the overloaded `checkout` command performs?  It’s used to create new branches, switch branches and restore files from the index or a reference.  You don’t have to use it much these days; [Git 2.23 added two very useful commands][switch-restore-release] to remedy the situation[^experimental]:

{{< highlight bash >}}
# create new branch
git switch -c shiny-topic-branch

# switch BRANCH
git switch master

# restore hello from index
git restore hello

# restore hello from commit 1fe35f
git restore --source 1fe35f hello

# restore hello in index from HEAD
git restore --staged hello

# restore hello in both index and working tree from 1fe35f
git restore --WS --source 1fe35f hello
{{< /highlight >}}

# _Strings Attached_ Tags

Annotated tags are tags with an accompanying message that are displayed by `git show`.  Unlike [lightweight tags][lightweight-vs-annotated-tag] they are unintuitive.

{{< highlight bash >}}
# Lightweight tag
git tag 1.0.0

# Annotated tag
git tag -m "Tag for release after sign off from CI team" 1.0.0
{{< /highlight >}}

However, the differences go deeper.  Annotated tags themselves are separate objects while lightweight objects are just pointers i.e. they point to a commit object.  Like commit objects, apart from the message, annotated tags also carry their own _tagger_ and _date_ fields.  When you `git push --follow-tags`, only annotated tags are pushed; `git push --tags` pushes both.  Since they’re separate objects, like all [Git objects][git-object], they’ve a distinct hash; consequently they can also be tagged!  This can bite you when you [rename an annotated tag][annotated-tag-rename] by [creating the new tag with the old one][usual-tag-rename].

# Frugal Fetches

When getting a remote branch, ignore the popular advise to `git fetch` as it fetches everything[^fetch-default]; a better option is to get just what you want; saves a lot of time:

{{< highlight bash >}}
# fetch just the interesting branch
git fetch origin my_topic_branch

**Note**: This applies to `git pull` as well since `fetch` is its first step.

# checkout locally and also set remote-tracking branch
git checkout --track origin/my_topic_branch
{{< /highlight >}}

**Tip**: `git checkout my_topic_branch` is an even shorter form of above if `origin` is the only remote with `my_topic_branch`.

# Fetch Fiascos?  Shallow Repos!

Irrespective of the number of times you yell `git fetch`, Git swears that there’s only one branch in a remote!  You look at `git branch -r` in dismay 😰

Well, you forgot that it’s a shallow repro i.e. cloned with `--single-branch`.  The caveat lurks in `man git-clone`

> `--[no-]single-branch`
>
> [...]
>
> Further fetches into the resulting repository will only update the
> remote-tracking branch for the branch this option was used for the initial
> cloning.

Deepen the shallow repo with `git fetch --unshallow`.  Further details at [an SO post][shallow-fetch].  I’ve also seen a manual method of editing `.git/config`: replacing branch name with `*` and `fetch`-ing

{{< highlight cfg >}}
[remote "origin"]
	url = https://github.com/legends2k/blog
# fetch = +refs/heads/master:refs/remotes/origin/master
# fetch = +refs/tags/master:refs/tags/master
	fetch = +refs/heads/*:refs/remotes/origin/*
	fetch = +refs/tags/*:refs/tags/*
{{< /highlight >}}

# Are You My Mother?

There’re times when you want to check a commit’s ancestory.  To know if `Possible-Parent` is truly an ancestor of `Child`

{{< highlight bash >}}
# check
git merge-base --is-ancestor possible_ancestor person
# show result
echo $0
{{< /highlight >}}

Prints `0` if the lineage checks out.  On Windows `echo %ErrorLevel%` tells the return value of the last command.

Conversely, if you’re looking for all the branches containing a commit

{{< highlight bash >}}
git branch --contains some_parent
{{< /highlight >}}

lists all branches descending from `some_parent`; of course, this isn’t useful if the descendant commit your looking for isn’t a branch tip.

The general idea is _reachability_: an ancestor is reachable from a descendant, if you keep following the parental links.  When an ancestor is _reachable_, changes it introduced are _visible_ to the descendant.

# Misc

* Delete branch from remote directly: `git push origin --delete my_topic`
* Remove local branches with no remote counterparts: `git remote prune origin`
  - Doing the same while fetching newer branches: `git fetch --prune`
* Remove untracked files from working tree: `git clean`
* Clean up unnecessary house-keeping files and optimize repo: `git gc`
  - Remove unreachable objects older than some duration: `git prune --expire 2.weeks.ago`; more manual
  - Git 2.31 introduced `git maintenance start` to GC periodically in the background :)


[mind-gap]: https://en.wikipedia.org/wiki/Mind_the_gap
[intervals]: https://en.wikipedia.org/wiki/Interval_(mathematics)#Including_or_excluding_endpoints
[log-dots]: https://stackoverflow.com/a/7256391/183120
[switch-restore-release]: https://hub.packtpub.com/git-2-23-released-with-two-new-commands-git-switch-and-git-restore-a-new-tutorial-and-much-more/
[lightweight-vs-annotated-tag]: https://stackoverflow.com/q/11514075/183120
[git-object]: https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
[annotated-tag-rename]: https://stackoverflow.com/a/49286861/183120
[usual-tag-rename]: https://stackoverflow.com/a/5719854/183120
[shallow-fetch]: https://stackoverflow.com/q/1778088/183120
[pull-sans-args]: https://stackoverflow.com/a/6861747/183120

[^fetch-default]: [`git fetch` without arguments][pull-sans-args] usually fetches all branches as `remote.<origin>.fetch` defaults to `+refs/heads/*:refs/remotes/<origin>/*`!
[^experimental]: `switch` and `restore` are still experimental.
