Modern Git Commands and Features You Should Be Using
MARTINMar 4, 2024

All of us - software engineers - use git every day, however most people only ever touch the most basic of commands, such as add, commit, push or pull, like it's still 2005.

Git however, introduced many features since then, and using them can make your life so much easier, so let's explore some of the recently added, modern git commands, that you should know about.

Switch
New since 2019, or more precisely, introduced Git version 2.23, is git switch which we can use to switch branches:

```bash
git switch other-branch
git switch -  # Switch back to previous branch, similar to "cd -"
git switch remote-branch  # Directly switch to remote branch and start tracking it
```
Well that's cool, but we've been switching branches in Git since ever using git checkout, why the need for a separate command? git checkout is a very versatile command - it can (among other things) check out or restore specific files or even specific commits, while the new git switch only switches the branch. Additionally, switch performs extra sanity checks that checkout doesn't, for example switch would abort operation if it would lead to loss of local changes.

Restore
Another new subcommand/feature added in Git version 2.23 is git restore, which we can use to restore a file to last committed version:

```bash
# Unstage changes made to a file, same as "git reset some-file.py"
git restore --staged some-file.py

# Unstage and discard changes made to a file, same as "git checkout some-file.py"
git restore --staged --worktree some-file.py

# Revert a file to some previous commit, same as "git reset commit -- some-file.py"
git restore --source HEAD~2 some-file.py
```
The comments in the above snippet explain the workings of various git restore. Generally speaking git restore replaces and simplifies some of the use cases of git reset and git checkout which are already overloaded features. See also this docs section for comparison of revert, restore and reset.

Sparse Checkout
Next one is git sparse-checkout, a little more obscure feature that was added in Git 2.25, which was released on January 13, 2020.

Let's say you have a large monorepo, with microservices separated into individual directories, and commands such as checkout or status are super slow because of the repository size, but maybe you really just need to work with single subtree/directory. Well, git sparse-checkout to the rescue:

```bash
$ git clone --no-checkout https://github.com/derrickstolee/sparse-checkout-example
$ cd sparse-checkout-example
$ git sparse-checkout init --cone  # Configure git to only match files in root directory
$ git checkout main  # Checkout only files in root directory
$ ls
bootstrap.sh  LICENSE.md  README.md

$ git sparse-checkout set service/common

$ ls
bootstrap.sh  LICENSE.md  README.md  service

$ tree .
.
├── bootstrap.sh
├── LICENSE.md
├── README.md
└── service
    ├── common
    │   ├── app.js
    │   ├── Dockerfile
    ... ...
```

In the above example we first clone the repo without actually checking out all the files. We then use git sparse-checkout init --cone to configure git to only match files in the root of the repository. So, after running checkout we only have 3 files rather than whole tree. To then download/checkout particular directory, we use git sparse-checkout set ....

As already mentioned, this can be very handy when working locally with huge repos, but it's equally useful in CI/CD for improving performance of a pipeline, when you only want to build/deploy part of the monorepo and there's no need to check out everything.

For detailed write-up about sparse-checkout see this article.

Worktree
It's not uncommon, that one might have to work on multiple features in single application (repository) at the same time, or maybe a critical bug comes in while you're in the middle of working some feature request.

In those situations, you either have to have multiple versions/branches of the repository cloned, or you need to stash/discard whatever you've been working on at the time. The answer to these situations is git worktree, released on September 24, 2018:

```bash
git branch
# * dev
# master

git worktree list
# /.../some-repo  ews5ger [dev]

git worktree add -b hotfix ./hotfix master

# Preparing worktree (new branch 'hotfix')
# HEAD is now at 5ea9faa Signed commit.

git worktree list
# /.../test-repo         ews5ger [dev]
# /.../test-repo/hotfix  5ea9faa [hotfix]

cd hotfix/  # Clean worktree, where you can make your changes and push them
```

This command allows us to have multiple branches of the same repository checked out at the same time. In the example above, we have 2 branches dev and master. Let's say we're working on feature in the dev branch, but we're told to make urgent bug fix. Rather than stashing the changes and resetting the branch, we create a new worktree in the ./hotfix subdirectory from the master branch. We can then move to that directory, do our changes, push them and return to the original worktree.

For a more detailed write-up see this article.

Bisect
Last but not least, git bisect, which isn't so new (Git 1.7.14, released on May 13, 2012), but most people are using only git features from around 2005, so I think it's worth showing anyway.

As the docs page describes it: git-bisect - Use binary search to find the commit that introduced a bug:

```bash
git bisect start
git bisect bad HEAD  # Provide the broken commit
git bisect good 479420e  # Provide a commit, that you know works
# Bisecting: 2 revisions left to test after this (roughly 1 step)
# [3258487215718444a6148439fa8476e8e7bd49c8] Refactoring.

# Test the current commit...
git bisect bad  # If the commit doesn't work
git bisect good # If the commit works

# Git bisects left or right half of range based on the last command
# Continue testing until you find the culprit

git bisect reset  # Reset to original commit
```

We start by explicitly starting the bisection session with git bisect start, after which we provide the commit that doesn't work (most likely the HEAD) and the last known working commit or tag. With that information, git will check out a commit halfway between the "bad" and "good" commit. At which point we need to test whether that version has the bug or not, we then use git bisect good to tell git that it works or git bisect bad that it doesn't. We keep repeating the process until no commits are left and git will tell us which commit is the one that introduced the issue.

I recommend checking out the docs page that shows couple more options for git bisect including visualizing, replaying or skipping commits.

Conclusion
If you search for some problem relating to git, you will most likely end up on StackOverflow question with answer that has couple thousand upvotes. While this answer will be mostly likely still valid, it very well might be outdated, because it was written 10 years ago. Therefore, there might be a better, simpler, easier way to do it. So, when faced with some git issue, I would recommend to check git docs for more recent commands, all of which have a lot of great examples, or to explore man pages for lots of flags and options that were added to the good old commands over the years.