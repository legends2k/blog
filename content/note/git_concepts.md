+++
title = "Git Concepts"
description = "Summary of my ‘Aha!’ moments"
date = 2019-02-20T19:56:56+05:30
tags = ["tech", "vcs", "tools"]
+++

- Git is a distributed VCS; each repo can be both a server/client
- Honestly, `git` (sub)commands are just graph manipulating commands
- Every codebase is made of a graph; each commit is a node with edges to parent(s)
  - Git diagrams often have arrows backwards for this reason only
- Git stores [snapshots not differences][] i.e. entire file contents --- as a blob
  - Every commit is a complete [snapshot of tracked repo contents + (0 or more) parent ID(s)][commit composition] identified with a 40-byte SHA1-digested hash
  - This way, the exact state of your project can be referred to, copied, or restored at any time

> "finally figuring out that git commands are strangely named graph manipulation commands -- creating/deleting nodes, moving around pointers"
>    -- Kent Beck

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

- An important (linked-list) concept that applies here too
  - If `void* first` is lost, the list, too, is lost
- Since a commit also has (0+) parent commits, following the chain of parents will eventually take you back to the beginning of the project
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
  - They’re specific to a single repository
- They facilitate easy git-speak with your friends/colleagues 😜
- [Internally][ref-internal] just a 40-byte file containing a commit ID
  - Remote references point to commits in remote repositories
- There’re many more ways of referring to commits: `man gitrevisions` is your friend

[ref-internal]: https://git-scm.com/book/en/v2/Git-Internals-Git-References
[references-commits]: http://think-like-a-git.net/sections/experimenting-with-git.html

## Commands Affecting Refs

Only these `git` subcommands allow you to move refs:

- `commit`
- `merge`
- `rebase`
- `reset`

Subcommands that affect moving remote refs:

- `fetch`
- `push`

Commands like `cherry-pick` internally use one of these.

# Learn by Doing

[try.github.io][] for good try-it-yourself resources.

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

[Think like a Git]: http://think-like-a-git.net/
[Refs]: http://think-like-a-git.net/sections/graphs-and-git/references.html
[Pro Git]: https://git-scm.com/book/en/v2
