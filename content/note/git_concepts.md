+++
title = "I Git it!"
description = "Summary of my ‘Aha!’ moments with Git"
date = 2019-02-20T19:56:56+05:30
tags = ["tech", "vcs", "tools"]
+++

Basics like initializing (a repository), staging and commiting files aren’t explained here; they simply make sense; no ‘Aha!’s there.  Moving references, branching and merging --- coupled with Git’s arcane command names[^3] --- are the confusing parts.

# Basics

- Git is a distributed VCS; each repo can be both a server/client
- Honestly, `git` (sub)commands are just graph manipulating commands
- Every codebase is made of a graph; each commit is a node with edges to parent(s)
  - Git diagrams often have arrows backwards (←) for this reason
- Git stores [snapshots not differences][] i.e. entire file contents --- as a blob
  - Every commit is a complete [snapshot of tracked repo contents + (0 or more) parent ID(s)][commit composition] identified with a 40-byte SHA1-digested hash
  - This way, the exact state of your project can be referred to, copied, or restored at any time

> "finally figuring out that git commands are strangely named graph manipulation commands -- creating/deleting nodes, moving around pointers"
>    -- Kent Beck

- Nodes of the graph are created by your commits
- Nodes are never really deleted in the traditional sense; they’re made unreachable (see below)
  - These unreachable nodes eventually get garbage collected by Git
- A commit’s parent(s) --- merge commits have more than one parent --- form the edges of the graph

[snapshots not differences]: https://git-scm.com/book/en/v1/Getting-Started-Git-Basics#_snapshots_not_differences
[commit composition]: http://think-like-a-git.net/sections/graphs-and-git/references.html

# Reachablity

{{< highlight cfg >}}
      A---B---C
     /
D---E---F---G
     \
      H---I
{{< /highlight >}}

- An important (linked-list) concept that applies to Git too
  - If `void* first` is lost, the list, too, is lost
- Since a commit also has (1+) parent commits (except root), following the chain of parents will eventually take you back to the beginning of the project
- In a well-branched graph, depending on the leaf node you start from, [different parts of the graph will be _reachable_][reachability]
- [Commit X is "reachable" from commit Y if commit X is an ancestor of commit Y][reachability-pro-git]
  - In the above example, `A`, `B` and `C` are unreachable from `G`, so are `F` and `G` when starting from `C` or `B` or `A`
- The `gc` subcommand walks through the graph, building a list of every commit it can reach; removes unreachable ones
  - Will clear-up disk space; no good reason to run it often
  - Some Git subcommands [may run it automatically][gc-man-page] too!
[reachability]: http://think-like-a-git.net/sections/graph-theory/reachability.html
[reachability-pro-git]: https://git-scm.com/docs/user-manual#understanding-reachability
[gc-man-page]: https://git-scm.com/docs/git-gc

# Refs

> "References make commits reachable"
>   -- [Think like a Git][references-commits]

- Plainly, [references][Refs] are "meaningful" names to some commits
  - Branches and tags are references too
    - Creating a branch is a way to "nail down" part of the graph that you want to return to later `[reachability]`
- They facilitate easy git-speak with your friends/colleagues 😜
- [Internally][ref-internal] just a 40-byte file containing a commit ID
  - They’re specific to a single repository
  - Remote references point to commits in remote repositories
- There’re many more ways of referring to commits: `man gitrevisions` is your friend

[ref-internal]: https://git-scm.com/book/en/v2/Git-Internals-Git-References
[references-commits]: http://think-like-a-git.net/sections/experimenting-with-git.html

## Commands Affecting Refs

These are the primary subcommands that allow you to move refs directly:

- `commit`
- `merge`
- `rebase`
- `reset`

Subcommands that affect moving remote refs:

- `fetch`
- `push`

Commands like `cherry-pick` seem to work atop one of these.

# Checkout vs Reset

To understand both commands, you first need to understand `HEAD`[^1].  Most people know about working tree and stating area but not `HEAD`.

[`HEAD`][HEAD definition SO] references the currently checked out commit; your working tree will mostly be from this snapshot i.e. the commit pointed to by `HEAD`.  [Pro Git][Reset Demystified] summarizes this nicely

> `HEAD` will be the parent of the next commit that is created.

[HEAD definition SO]: https://stackoverflow.com/q/5772192/183120

## Checkout

{{< highlight basic >}}
git checkout HEAD -- file
{{< /highlight >}}

When you `checkout` a `file` from `HEAD`, what you do is get a clean copy of `file` from the commit `HEAD` is pointing to; this _replaces_ your working tree copy.  Of course, one could use other refs too, `HEAD` is just a convenient default, you can replace it with any ref; if `HEAD` is omitted, it’ll be from index --- the stage.

{{< highlight basic >}}
git checkout topic
{{< /highlight >}}

When you checkout a branch (reference to a commit/node) e.g. `topic`, `HEAD` will be moved to that branch and hence the entire working tree, not just a file, will be from the snapshot of the commit that branch is pointing to.

## Reset

Plainly, `reset` moves `HEAD` around.  It’s used to move `HEAD` to a given commit.  There’re different flavours of doing this --- depending on what happens to the index and working tree (`--hard`, `--soft`, `--mix` …) --- but the crux is to move `HEAD`.

When moving, `HEAD` will also move the branch reference along with it, if it’s _attached_.

### Detached `HEAD`

Whoa! Slow down there, cowboy.  Before talking about detached, what’s the attached state of `HEAD`?  We already know that `HEAD` is just a reference to a commit.  Say this commit also has another reference pointing to it: a branch name.

{{< highlight basic >}}
C1 <-- C2 <-- C3 <-- C4 <-- C5 <-- master
                             ^
                             |
                            HEAD
{{< /highlight >}}

> When `HEAD` is moved by `reset`, if it’s attached to a branch, that reference will move along with `HEAD`.

{{< highlight basic >}}
git reset --hard C3
{{< /highlight >}}

would move _both_ `HEAD` and `master` to `C3`[^2].  `HEAD` would continue to be attached.  Now if it weren’t attached, it’ll only move `HEAD` leaving `master` behind, hence the _detached_ `HEAD` state.

There’re a couple of ways to identify if `HEAD` is detached.  `git status`’s very first line will tell you:

{{< highlight basic >}}
> git status
On branch master
…
> git status
# HEAD detached at 847fe59
{{< /highlight >}}

Another way is to use `git log`; I learnt from this actually.

{{< highlight basic >}}
> git log --oneline --all -5
847fe59 (HEAD -> master) Initial commit
…
> git log --oneline --all -5
847fe59 (HEAD, master) Initial commit
{{< /highlight >}}

Note that when `HEAD` is attached, you see an arrow (→) pointing to the branch it’s attached to.  However, in the detached state they’re list as independent items.

### Attach/Detaching `HEAD`

How do we attach or detach `HEAD` to a reference?  Both are done with `checkout`, but with a subtle difference.  To attach `HEAD`, you’d `checkout`

{{< highlight basic >}}
> git checkout master
{{< /highlight >}}

_When you checkout a commit by using anything other than a branch name (commit ID, HEAD~1, branch~3, HEAD^^, etc.), you’d detach `HEAD`_ i.e. Git wouldn’t know what to associate `HEAD` with.  When you want to poke around and see how the repository looked like at a particular unnamed commit (except for its commit ID), this is what you would normally do.

{{< highlight basic >}}
> git checkout lk3nw7ef
{{< /highlight >}}

Here, it doesn’t matter if this commit has other branch references to it.  Since you referred to it using the raw commit ID, Git takes it as a cue to detach `HEAD`.

I highly recommend playing around in [Visualizing Git][] with `checkout`, `reset`; get your hands dirty with the whole attach/detach business.  Here’s a small snippet to get you started; see what happens as each command gets executed:

{{< highlight basic >}}
git commit
git commit
git commit
git commit
git commit
# create topic branch and checkout; HEAD now attached to topic
git checkout -b topic
# move HEAD one commit behind topic; this will also move topic with HEAD
git reset topic~1
# detach HEAD!
git checkout HEAD~2
# attach to master
git checkout master
# move back master by 3
git reset master~3
# move master forward/backward with commit ID
git reset f08ad6
{{< /highlight >}}

[Reset Demystified]: https://www.git-scm.com/book/en/v2/Git-Tools-Reset-Demystified

# Rebase

`rebase`  seems to have a scary reputation on the web, with good reason of course, since it’s infamous for rewriting history; something your teammates mightn’t take kindly.  However, when you’re doing this only within your repo, before pushing, it’s a great tool.

> The crux of a `rebase`: given a subgraph’s root node, `rebase` changes its parent pointer from one node to another; thereby _rebasing_ the entire subgraph to a new parent.

Interactive rebase (`rebase -i`) is quite useful.  I frequently use it to amend (not just the recent commit), fix, reword, edit, drop or squash commits.  During an interactive rebase, one can even create multiple commits as usual and continue with the rebase; things will be taken care of!  This is normal when [dividing][divide commit] a commit into smaller parts.

[divide commit]: https://stackoverflow.com/q/6217156/183120

## pull = fetch + ~~merge~~ rebase? 🤔

When pulling from a remote branch, you might know that your changes are unrelated to the ones coming down.  In this case, to avoid a merge commit and have a linear commit history, you’d pass `--rebase` do override the default merge strategy of `pull`: merge.

{{< highlight basic >}}
git pull --rebase origin master
{{< /highlight >}}

`git pull` is just `git fetch` followed by `git merge` which creates a new merge commit.  `git pull --rebase`, however, is `git fetch` and `git rebase`; it pulls commits from remote to your current branch and then replay your commits atop your current branch’s tip -- this works if there’re no merge conflicts; otherwise you’ve to resolve conflicts as you’d normally.  The resolution (changes) become a part of one of your commits where rebase halted; you’d end up [re-writing your commit][rebase conflict].  However, you don’t have to force push your changes to the remote since the resolution just happened in your local commits.  Rewriting of (commit) history, as long as it is not public, is OK 😉

A counter point to pull-with-rebase: if you want logical separation of a set of commits, say for a completely new feature, then rebase --- which makes them inline, muddled with unrelated history --- isn’t the right tool; use `merge` instead.

> Use `git pull --rebase` when your changes do not deserve a separate branch.

seems to be the appropriate answer to [when should I `git pull --rebase`][when pull rebase].

[rebase conflict]: https://stackoverflow.com/a/35025978/183120
[when pull rebase]: https://stackoverflow.com/q/2472254/183120

# Learn by Doing

[try.github.io][] for good DIY resources.

- [Visualizing Git][] -- lets you visualize your git commands
- [Visualizing Git Concepts with D3][] -- explains commands with interactive images
- A [Visual Git Reference][] -- explains commands with images

[try.github.io]: https://try.github.io/
[Visualizing Git]: https://git-school.github.io/visualizing-git/
[Visualizing Git Concepts with D3]: http://onlywei.github.io/explain-git-with-d3/
[Visual Git Reference]: http://marklodato.github.io/visual-git-guide/index-en.html

# References

1. [Think like a Git][]
2. [Pro Git][]
3. [Git Tutorials by Atlassian][]
4. [Git Ready][]

[Think like a Git]: http://think-like-a-git.net/
[Refs]: http://think-like-a-git.net/sections/graphs-and-git/references.html
[Pro Git]: https://git-scm.com/book/en/v2
[Git Tutorials by Atlassian]: https://www.atlassian.com/git/tutorials
[Git Ready]: http://gitready.com/

[^1]: case-sensitive
[^2]: using `C3` for readability; substitute with proper commit ID
[^3]: I use [Magit][] to interact with Git but knowing them helps


[Magit]: https://magit.vc/

