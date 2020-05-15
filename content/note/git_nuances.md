+++
title = "Git Wizardry"
description = "Obscure but useful Git incantations"
date = 2020-05-13T19:56:56+05:30
tags = ["tech", "vcs", "tools"]
+++

Git has its oddities with unusual commands and flags.  I keep learning something new every now and then.

A decent understanding of [basic Git concepts][git-concepts] is a perquisite.

[git-concepts]: {{< relref "git_concepts.md" >}}

# `show` time!

The easiest way to see the diff introduced by a [reference][git-concepts-ref] is not `git diff`!

{{< highlight basic >}}
git show REF
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

Ever been annoyed by a Git setting you didn’t set and didn’t know where it comes from?

{{< highlight cfg >}}
git config --show-origin  --list

file:/usr/local/etc/gitconfig       credential.helper=osxkeychain
file:/Users/my_user/.gitconfig      user.name=John Q. Public
file:/tmp/Cool_Project/.gitconfig   core.pager=less --quit-if-one-screen --no-init
{{< /highlight >}}

will list all the settings along with the corresponding `.gitconfig` file it’s coming from.

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

Contrary to popular belief, [`git pull` can be run on a branch that’s not current][pull-non-HEAD].  Say `HEAD` is attached to `master`, you can still pull in changes for `topic` branch from a remote.

{{< highlight basic >}}
git fetch origin topic:topic
{{< /highlight >}}

[pull-non-HEAD]: https://stackoverflow.com/a/42902058/183120

# Please [mind the dots][mind-gap]

`git diff A..B` = diff [(`A`, `B`\]][intervals] i.e. the diff includes changes made by `B` but not by `A`.  Since a diff-ing utility just takes a pair of file sets, with these references we simply denote the repository at different states -- after committing `A` and after committing `B`.  This diffs the repository at _two points_.  `git diff A B` is a nicer way of saying the same thing.

`git diff A...B` = `git diff $(git merge-base A B) B` i.e. difference between the common ancestor of both references and `B`.  `A` is usually `HEAD` and `B` is commonly a branch head, so this shows work done independently in a branch.  _Memory aid_: triple dots ≈ branch.

However, the [meanings feel reversed for `git log`][log-dots]! 🤦 Also `A..B` and `A B` mean the same in `diff` while not in `log`!

{{< highlight bash >}}
git log A...B  # show A-only and B-only commits
git log A..B   # show B-only commits
git log A B    # show A ∪ B commits
{{< /highlight >}}

[Pro Git v2 explains][pro-git-log] these in `git log`’s context.

[pro-git-log]: https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection

# Overloaded `checkout`

Keep getting confused between the various tasks that the overloaded `checkout` command performs?  It’s used to create new branches, switch branches and restore files from the index or a reference.  You don’t have to use it much these days; [Git 2.23 added two very useful commands][switch-restore-release] to remedy the situation[^1]:

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


[mind-gap]: https://en.wikipedia.org/wiki/Mind_the_gap
[intervals]: https://en.wikipedia.org/wiki/Interval_(mathematics)#Including_or_excluding_endpoints
[log-dots]: https://stackoverflow.com/a/7256391/183120
[switch-restore-release]: https://hub.packtpub.com/git-2-23-released-with-two-new-commands-git-switch-and-git-restore-a-new-tutorial-and-much-more/

[^1]: `switch` and `restore` are still experimental as of Git 2.26.2
