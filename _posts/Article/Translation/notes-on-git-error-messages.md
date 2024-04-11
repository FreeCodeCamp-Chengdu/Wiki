---
title: Julia Evans
authorURL: ""
originalURL: https://jvns.ca/blog/2024/04/10/notes-on-git-error-messages/
translator: ""
reviewer: ""
---

# Notes on git's error messages

<!-- more -->

While writing about Git, I’ve noticed that a lot of folks struggle with Git’s error messages. I’ve had many years to get used to these error messages so it took me a really long time to understand _why_ folks were confused, but having thought about it much more, I’ve realized that:

1.  sometimes I actually _am_ confused by the error messages, I’m just used to being confused
2.  I have a bunch of strategies for getting more information when the error message git gives me isn’t very informative

So in this post, I’m going to go through a bunch of Git’s error messages, list a few things that I think are confusing about them for each one, and talk about what I do when I’m confused by the message.

### improving error messages isn’t easy

Before we start, I want to say that trying to think about why these error messages are confusing has given me a lot of respect for how difficult maintaining Git is. I’ve been thinking about Git for months, and for some of these messages I really have no idea how to improve them.

Some things that seem hard to me about improving error messages:

-   if you come up with an idea for a new message, it’s hard to tell if it’s actually better!
-   work like improving error messages often [isn’t funded][1]
-   the error messages have to be translated (git’s error messages are translated into [19 languages][2]!)

That said, if you find these messages confusing, hopefully some of these notes will help clarify them a bit.

## [error: `git push` on a diverged branch][3]

$ git push
To github.com:jvns/int-exposed
! \[rejected\]        main -> main (non-fast-forward)
error: failed to push some refs to 'github.com:jvns/int-exposed'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 1 different commits each, respectively.

Some things I find confusing about this:

1.  You get the exact same error message whether the branch is just **behind** or the branch has **diverged**. There’s no way to tell which it is from this message: you need to run `git status` or `git pull` to find out.
2.  It says `failed to push some refs`, but it’s not totally clear _which_ references it failed to push. I believe everything that failed to push is listed with `! [rejected]` on the previous line– in this case just the `main` branch.

**What I like to do if I’m confused:**

-   I’ll run `git status` to figure out what the state of my current branch is.
-   I think I almost never try to push more than one branch at a time, so I usually totally ignore git’s notes about which specific branch failed to push – I just assume that it’s my current branch

## [error: `git pull` on a diverged branch][4]

$ git pull
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint:
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.

The main thing I think is confusing here is that git is presenting you with a kind of overwhelming number of options: it’s saying that you can either:

1.  configure `pull.rebase false`, `pull.rebase true`, or `pull.ff only` locally
2.  or configure them globally
3.  or run `git pull --rebase` or `git pull --no-rebase`

It’s very hard to imagine how a beginner to git could easily use this hint to sort through all these options on their own.

If I were explaining this to a friend, I’d say something like “you can use `git pull --rebase` or `git pull --no-rebase` to resolve this with a rebase or merge _right now_, and if you want to set a permanent preference, you can do that with `git config pull.rebase false` or `git config pull.rebase true`.

`git config pull.ff only` feels a little redundant to me because that’s git’s default behaviour anyway (though it wasn’t always).

**What I like to do here:**

-   run `git status` to see the state of my current branch
-   maybe run `git log origin/main` or `git log` to see what the diverged commits are
-   usually run `git pull --rebase` to resolve it
-   sometimes I’ll run `git push --force` or `git reset --hard origin/main` if I want to throw away my local work or remote work (for example because I accidentally commited to the wrong branch, or because I ran `git commit --amend` on a personal branch that only I’m using and want to force push)

## [error: `git checkout asdf` (a branch that doesn't exist)][5]

$ git checkout asdf
error: pathspec 'asdf' did not match any file(s) known to git

This is a little weird because we my intention was to check out a **branch**, but `git checkout` is complaining about a **path** that doesn’t exist.

This is happening because `git checkout`’s first argument can be either a branch or a path, and git has no way of knowing which one you intended. This seems tricky to improve, but I might expect something like “No such branch, commit, or path: asdf”.

**What I like to do here:**

-   in theory it would be good to use `git switch` instead, but I keep using `git checkout` anyway
-   generally I just remember that I need to decode this as “branch `asdf` doesn’t exist”

## [error: `git switch asdf` (a branch that doesn't exist)][6]

$ git switch asdf
fatal: invalid reference: asdf

`git switch` only accepts a branch as an argument (unless you pass `-d`), so why is it saying `invalid reference: asdf` instead of `invalid branch: asdf`?

I think the reason is that internally, `git switch` is trying to be helpful in its error messages: if you run `git switch v0.1` to switch to a tag, it’ll say:

```
$ git switch v0.1
fatal: a branch is expected, got tag 'v0.1'`
```

So what git is trying to communicate with `fatal: invalid reference: asdf` is “`asdf` isn’t a branch, but it’s not a tag either, or any other reference”. From my various [git polls][7] my impression is that a lot of git users have literally no idea what a “reference” is in git, so I’m not sure if that’s coming across.

**What I like to do here:**

90% of the time when a git error message says `reference` I just mentally replace it with `branch` in my head.

## error: [`git checkout HEAD^`][8]

$ git checkout HEAD^
Note: switching to 'HEAD^'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 182cd3f add "swap byte order" button

This is a tough one. Definitely a lot of people are confused about this message, but obviously there's been a lot of effort to improve it too. I don't have anything smart to say about this one.

**What I like to do here:**

-   my shell prompt tells me if I’m in detached HEAD state, and generally I can remember not to make new commits while in that state
-   when I’m done looking at whatever old commits I wanted to look at, I’ll run `git checkout main` or something to go back to a branch

## [message: `git status` when a rebase is in progress][9]

This isn’t an error message, but I still find it a little confusing on its own:

$ git status
interactive rebase in progress; onto c694cf8
Last command done (1 command done):
   pick 0a9964d wip
No commands remaining.
You are currently rebasing branch 'main' on 'c694cf8'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged ..." to unstage)
  (use "git add ..." to mark resolution)
  both modified:   index.html

no changes added to commit (use "git add" and/or "git commit -a")

Two things I think could be clearer here:

1.  I think it would be nice if `You are currently rebasing branch 'main' on 'c694cf8'.` were on the first line instead of the 5th line – right now the first line doesn’t say which branch you’re rebasing.
2.  In this case, `c694cf8` is actually `origin/main`, so I feel like `You are currently rebasing branch 'main' on 'origin/main'` might be even clearer.

**What I like to do here:**

My shell prompt includes the branch that I’m currently rebasing, so I rely on that instead of the output of `git status`.

## [error: `git rebase` when a file has been deleted][10]

$ git rebase main
CONFLICT (modify/delete): index.html deleted in 0ce151e (wip) and modified in HEAD.  Version HEAD of index.html left in tree.
error: could not apply 0ce151e… wip

The thing I still find confusing about this is – `index.html` was modified in `HEAD`. But what is `HEAD`? Is it the commit I was working on when I started the merge/rebase, or is it the commit from the other branch? (the answer is “`HEAD` is your branch if you’re doing a merge, and it’s the “other branch” if you’re doing a rebase, but I always find that hard to remember)

I think I would personally find it easier to understand if the message listed the branch names if possible, something like this:

```
CONFLICT (modify/delete): index.html deleted on `main` and modified on `mybranch`
```

## [error: `git status` during a merge or rebase (who is “them”?)][11]

$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run “git commit”)
  (use “git merge –abort” to abort the merge)

Unmerged paths:
  (use “git add/rm …” as appropriate to mark resolution)
    deleted by them: the\_file

no changes added to commit (use “git add” and/or “git commit -a”)

I find this one confusing in exactly the same way as the previous message: it says `deleted by them:`, but what “them” refers to depends on whether you did a merge or rebase or cherry-pick.

-   for a merge, `them` is the other branch you merged in
-   for a rebase, `them` is the branch that you were on when you ran `git rebase`
-   for a cherry-pick, I guess it’s the commit you cherry-picked

**What I like to do if I’m confused:**

-   try to remember what I did
-   run `git show main --stat` or something to see what I did on the `main` branch if I can’t remember

## [error: `git clean`][12]

$ git clean
fatal: clean.requireForce defaults to true and neither -i, -n, nor -f given; refusing to clean

I just find it a bit confusing that you need to look up what `-i`, `-n` and `-f` are to be able to understand this error message. I’m personally way too lazy to do that so even though I’ve probably been using `git clean` for 10 years I still had no idea what `-i` stood for (`interactive`) until I was writing this down.

**What I like to do if I’m confused:**

Usually I just chaotically run `git clean -f` to delete all my untracked files and hope for the best, though I might actually switch to `git clean -i` now that I know what `-i` stands for. Seems a lot safer.

### that’s all!

Hopefully some of this is helpful!

[Making crochet cacti][13]

[1]: https://lwn.net/Articles/959768/
[2]: https://github.com/git/git/tree/master/po
[3]: #git-push-on-a-diverged-branch
[4]: #git-pull-on-a-diverged-branch
[5]: #git-checkout-asdf
[6]: #git-switch-asdf
[7]: https://jvns.ca/blog/2024/03/28/git-poll-results/
[8]: #detached-head
[9]: #rebase-in-progress
[10]: #merge-deleted
[11]: #merge-ours
[12]: #git-clean
[13]: https://jvns.ca/blog/2024/04/01/making-crochet-cacti/ "Previous Post: Making crochet cacti"