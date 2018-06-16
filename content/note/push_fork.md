+++
tags = ["vcs", "git", "sync"]
date = "2018-04-19T13:57:03-07:00"
description = "the original origin"
title = "Git: Pushing to a Fork"

+++


# Fork? Food!?

When you fork[^1] a repository in GitHub, from your repository's viewpoint, the one you forked from is called the "[upstream][]"; a term used commonly in the [FOSS][] world for a long time, to denote the original authors (or their repository) of a software.  _Fork_ is a concept introduced by GitHub (the code hosting platform) for separation of concerns; Git (the version control system) itself doesn't have a concept of fork; from its viewpoint, it's just another remote repository.  So both the _upstream_ (original repo at GitHub from where you forked your repo) and _origin_ (your fork at GitHub) are two remotes to your local repo.

Between your `commit`, `push` to origin and the pull request approval, more commits could've gotten pushed to the upstream.  To resolve you sync

1. local to upstream
    * bringing your local up to date (the heavy-lifting a.k.a *merge*)
2. origin to local
    * bringing your own remote (origin) up to date
3. upstream to local
    * finally making your changes

[upstream]: https://en.wikipedia.org/wiki/Upstream_(software_development)
[FOSS]: https://en.wikipedia.org/wiki/Free_and_open-source_software

# Steps

1. Make sure you've the upstream as one of your remotes, if not add
{{< highlight cfg >}}
git remote add Upstream git@github.com:user/project.git
{{< /highlight >}}

1. Pull changes from upstream by rebasing instead of a merge
{{< highlight cfg >}}
git pull --rebase Upstream <branch_name>
{{< /highlight >}}

    1. To verify you've done this correctly, you can do `git log --oneline -10`
    2. You should see your commit at `HEAD` followed by the most recent commit in `Upstream/<branch_name>`
    3. This confirms that your change would be the latest commit atop all previous commits in upstream

3. Force push your commit to origin (your fork's remote)
{{< highlight cfg >}}
git push --force origin <branch_name>
{{< /highlight >}}
    1. Though you'd have earlier pushed to your origin for creating the pull request, you redo it to have these new changes gotten from upstream to be synced into your fork's history
    2. You can verify with the above log command again; both origin and your local `HEAD` should point to the commit you got approval for; upstream should be at the next commit
4. Push to upstream
{{< highlight cfg >}}
git push Upstream <branch_name>
{{< /highlight >}}

[^1]: No, not the culinary or chess parlance, silly! The [software meaning](https://en.wikipedia.org/wiki/Fork_(software_development)).
