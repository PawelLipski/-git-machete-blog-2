
# `git machete` strikes again! Custom annotations, branch dependency inference and more


## Intro

Good news - git machete 2.0 has been released!
For those of you not familiar with this nifty tool, I recommend quick look at the [previous part of this series (link)](https://virtuslab.com/blog/make-way-git-rebase-jungle-git-machete) -
or at least a look at the screenshots to get a TL;DR-y kind of understanding what machete actually does.

To get the latest git machete release, you can get it directly from [the git machete repo (github.com/PawelLipski/git-machete)](https://github.com/PawelLipski/git-machete):

```bash
$ git clone https://github.com/PawelLipski/git-machete.git
$ cd git-machete
$ sudo make install
```

`make install` copies the `git-machete` Python 2.7 executable to `/usr/local/bin` and sets up a corresponding Bash completion script in `/etc/bash_completion.d`.

The previous post met with quite ???sizable interest, especially in comments on a Reddit ???share.
It was exactly there where some of the improvements (especially automatic dependency inference) have been suggested - many thanks for the feedback, esp. for the Reddit user ????(TODO).
Other recent tweaks to git machete were introduced simply to make the day-to-day use of the tool even more convenient.
Also, special thanks for @### (github.com/) who raised an issue (TODO link) regarding the crashes of git machete when run from a git submodule.

As in the first part of the series, there is a script that sets up a demo repository with a couple of branches -
[you can download it directly from GitHub (link)](https://raw.githubusercontent.com/PawelLipski/git-machete-blog/master/sandbox-setup.sh).

One tricky thing with this script is that it actually sets two repos - one at `~/machete-sandbox` and another one at ~/machete-sandbox-remote`.
The latter is just a _bare_ repository (i.e. created with `git init --bare`) - that is, a one that doesn't have working tree, just `.git` folder.
This will serve as a local dummy remote repo for the one under `~/machete-sandbox`.
The script runs `git remote add origin ~/machete-sandbox-remote` so as to establish an actual local/remote relation between the repos, with all the push/pull capabilities that are available over https or ssh.

Structure of branches in the demo (the contents of the definition file, modulo annotations that we'll cover soon) is as follows:

```
develop
    allow-ownership-link
        build-chain
    call-ws
master
    hotfix/add-trigger
```


## Which PR was that... custom annotations and improved remote sync-ness status

A common problem is that for most locally developed branches (possibly other than dependency tree roots, like `develop` or `master`), at some point you'll have a pull request on e.g. GitHub or Bitbucket.
It can get pretty inconvenient to "optically" match branch names to PRs... especially in case of bitbucket which by default doesn't display source branch names in the PR list, only the destination branch.
To make it easier, `git machete status` received a simple tweak recently: custom annotations.
Simply put any phrase after the branch name, separated with a single space (there's an implicit assumption here that you never put spaces in git branch names... if you do, think it over twice!),
and it will be displayed in output of `status` subcommand.
To be precise, anything can be placed as annotation, not only PR number specifially... but PR number seems the most natural use case.

The annotations were already set up by the `sandbox-setup` script.
Let's print the `status`:

// Img here

The PR numbers are here completely random here, since the script obviously didn't set up any actual PRs anywhere.

You could also notice that the output slighly changed in terms of remote-syncness message.
What was especially inconvenient in the earlier versions of Machete was that you couldn't really distinguish between an untracked branch and a one that is tracked but is not in sync with its upstream.
As of v2.0.0, `git machete status` now distinguishes between the following cases:
* `untracked`
* `ahead of origin`
* `behind origin`
* `diverged from origin`,
pretty much just like `git status` does.

Now it's also more consistent with `git status` in that it uses the remote tracking branch information (as set up via `git branch --set-upstream` or `git push -u`) to determine the remote counterpart branch,
rather than simply matching branches by name.


## Don't remember what depended on what... branch dependency inference

If you plan to add some existing local branch to the dependency tree, but you don't remember what it actually depended on in the first place...

TODO add a branch that's not listed
In the demo we have a branch ??? that is not yet listed in the definition file.

Now let's just do `git machete add ????`

TODO insert screenshot

In case the desired upstream branch isn't specified with `--onto` option, `add` subcommand tries infers this upstream by some log/reflog magic similar to the one used for `fork-point`
(as described in the first part of the series).

TODO remove {
	That's too complicated to outline in details, but in general it is based on a similar trick as the algorithm for determining the fork point:

	To infer the upstream branch for a branch X
	reflogs of all other local branches Y are compared to the logs of (roughly) the reflog-wise earliest commits ever done on X.

	Then, if ...

	Some extra measures are taken to make sure that no cycles occur (so that we actually end up with a tree of branches and not some arbitrary graph).
}

What's more, this inference is not limited to just a single branch - it can even be performed on repository where there is not `.git/machete` file yet to infer the entire dependency tree in a single pass!

Let's now remove the `.git/machete` file (so that we make sure `git machete` doesn't have any hint on the inferred result) and run `git machete infer`:

TODO screenshot

In this case `infer` guessed the entire tree properly basically by just looking at branch reflogs (and also doing some tricks to prevent cycles from happening in the inferred graph).

`infer` gives the choice to either accept the inferred tree with `y[es]`, `e[dit]` the tree or reject the suggested version with `n[o]`.

In case of `yes`/`edit`, the old definition file (if it already exists) will be saved under `.git/machete~` (note the added tilde).

At this point one can ask a question: why then do we even need the definition file since we can always infer the upstreams on the fly when doing `status`, `update` etc.?
The reason against that is that ???we don't want everything to happen ?????, we need to leave a sensible amount of control in the hands of the developer while still helping ???


## Too lazy to think what to update next... dependency tree traversal

The sequence of steps suggested in the first blog post (TODO link to section!), namely:

* check out a branch X
* rebase it on the top of its upstream
* push with force to remote if needed
* check out another branch - a child of X

is actually quite repetitive in a daily work with `git machete`, esp. when you receive a lot of remarks on review and push fixes in rounds.

To free yourself from thinking about what to check out next, you can check a kind of wizard that walks (or rather, traverses) the branch dependency tree and
suggests what needs to be done next to restore sync of branches with their parent branches and remotes - it's called `traverse`.

The traversal is performed by moving to `next` of each branch (just like doine of `git machete go next`).
More specifically, this is equivalent to a pre-order depth-first search of the tree - each node (i.e. each git branch) visited (and possibly synced) before any of its children are visited.
This ### makes sense since you definitely want to put each branch `X` in sync with its parent branch first before syncing `X`'s children to `X` itself.

Let's check out the `develop` branch (which is a root of the dependency tree) and then iterate through the branches.

TODO screenshot

We see that 

for ??? we weren't asked to rebase onto ??? since the branches were already aligned.
Similiarly, `traverse` didn't suggest to push ??? since it was already in sync with origin/???

TODO: screenshot of interaction for the set up env


