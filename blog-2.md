
# `git machete` strikes again! Custom annotations, branch dependency inference and more


## Intro

Link to machete as such + install instructions again

Link to machete-sandbox-2.sh - another one, simpler!

```
develop
    allow-ownership-link
        build-chain
    call-ws
master
    hotfix/add-trigger
```

Address the problems that readers of the previous part suggested via Reddit, and solve some other issues that I came across in the day-to-day use.


## Which PR was that... custom annotations and improved remote sync-ness status

A common problem is that you have some PRs

Simply put any phrase after the branch name, separated with a single space (there's an implicit assumption here that you never put spaces in git branch names... if you do, think it over twice :D).

Now when printing the `status`:

// Img here

// TODO setup a local remote for the a repo

You could also see that the output slighly changed.

What was especially inconvenient with earlier versions of machete was that you couldn't really distinguish between a untracked branch and a one that is tracked but is not in sync with its upstream.

Once it even made me forget about one of my branches ###

`git machete status` now distinguishes between the following:
* `untracked`
* `ahead of remote`
* `behind remote`
* `diverged from origin`

Now also more consistent with `git status` in that it uses the remote tracking branch set with `git branch --set-upstream` or `git push -u`.


## Don't remember what depended on what... branch dependency inference

If you plan to add some existing local branch, but you don't remember what it actually depended on in the first place...

`add` subcommand infers the upstream branch

That's too complicated to outline in details, but in general it is based on a similar trick as the algorithm for determining the fork point:

To infer the upstream branch for a branch X
reflogs of all other local branches Y are compared to the logs of (roughly) the reflog-wise earliest commits ever done on X.

Then, if ...

Some extra measures are taken to make sure that no cycles occur (so that we actually end up with a tree of branches and not some arbitrary graph).

We set up the repository according to the script: ####

Let's now run `git machete infer` to see what is being suggested:

// Insert the pic

Now you can either accept the inferred tree with `y[es]`, `e[dit]` the tree or reject the suggested version with `n[o]`.

In case of `yes`/`edit`, the old definition file will be saved under `.git/machete~` (note the added tilde).


## Too lazy to think what to update next... dependency tree traversal

The sequence of steps suggested in the first blog post (TODO link to section!), namely:

* check out a branch X
* rebase it on the top of its upstream
* push with force to remote if needed
* check out another branch - a child of X

is actually quite repetitive in a daily work with `git machete`, esp. when you receive a lot of remarks on review and push fixes in rounds.

To free yourself from thinking about what to check out next, you can check a kind of wizard that walks (or rather, traverses) the branch dependency tree and
suggests what needs to be done next to restore sync of branches with their parent branches and remotes - it's called `traverse`.

The traversal is performed by moving to `next` of each branch ###
More specifically, this is equivalent to a pre-order depth-first search of the tree - each node (i.e. each git branch) visited (and possibly synced) before any of its children are visited.
This ### makes sense since #####

Let's

TODO: screenshot of interaction for the set up env


