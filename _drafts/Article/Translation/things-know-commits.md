---
title: Jamie Tanna | Software Engineer
date: 2024-07-12T19:01:09.000Z
author: Jamie Tanna
authorURL: https://www.jvt.me/
originalURL: https://www.jvt.me/posts/2024/07/12/things-know-commits/
translator: ""
reviewer: ""
---

Written by Jamie Tanna  
on   
[CC-BY-NC-SA-4.0][1] [Apache-2.0][2]  
7 mins

<!-- more -->

# 89 things I know about Git commits

![Featured image for sharing metadata for article](https://media.jvt.me/53239026de.png)

This is an article that's been swimming around in my head for ~5 weeks now, and may become a "living post" that I keep updated over time.

In no particular order, some things I've learned about Git commits and commit history, over the last 12 years. This is a mix of experience in companies with teams of 2-12 people, as well as in Open Source codebases with a vast range of contributors.

1.  Git has different uses - a collaboration tool, a backup tool, a documentation tool
2.  Git commit messages are excellent
3.  I've never met anyone who likes reading commit messages as much as me
4.  Finding out why a change was made is easier sorting through commits than it is through an issue/bug tracker
5.  It's better to have a commit that says "Various fixes. DEV-123" than it is to have "Various fixes"
6.  It's worse to have a commit that says "Various fixes. DEV-123" when the issue itself has no useful information
7.  Rebase-merging is my preference. Then squash-merge, then merge
8.  If you don't learn how to rebase, you're missing out on a good skill
9.  People who say "just delete the repo" when things go wrong really frustrate me
10.  Learn how to use `git reflog`, and it'll save you deleting a repo and work that was recoverable
11.  Learn how to use `git reflog`, and you'll be able to save yourself from mistakes that aren't that bad
12.  No amount of learning fancy tools and commands saves you from fucking up every once in a while
13.  My most recent botched rebase was last week, and I needed `git reflog` to help un-fuck it
14.  Learn how to [undo a force push][3] and then how to [more safely force push][4] (remember the `=ref`!!)
15.  Squashing is a waste of well-written atomic commits
16.  Squashing is better than 100 crap commits
17.  Squashing, and writing a good commit message at the time of merge is good
18.  Squashing, and then not re-editing the message is the worst
19.  Squashing, when you have 100 crap commits, and then not re-editing the message is **a crime**
20.  Squashing, and then not re-editing the message is worse than a merge commit from a branch with 100 crap commits
21.  Writing a well documented PR/MR description but not using that to inform the squash-merge message is a waste of time
22.  Writing commit messages have helped me pick up on missing test cases, missing documentation, or invalid thought processes, as it helps me rewrite _why_ the changes are being made
23.  Using your `git log` as an indication for standup updates is valid
24.  I can't be bothered to sign my commits (unless I'm forced to)
25.  If I have to sign my commits, SSH key signing makes it almost not awful
26.  If you're moving files between repos, you need to keep the history intact, [using `git subtree`][5]
27.  Commits should be atomic - all the code and tests, and configuration changes should all be in there
28.  I spend a good chunk of time ensuring that each commit passes CI checks atomically
29.  Some people do horrific things, like split their implementation and test code from each other
30.  It's OK to put documentation in a separate commit - we don't have to have a whole end-to-end feature delivery in a single commit
31.  Repos that use squash-merges suck
32.  As a maintainer of Open Source projects, I like squash-merge, so I can rewrite contributors' commit messages
33.  Sometimes it's not worth coaching how to write a given commit message
34.  The people around you shape the way you write commits
35.  Do the work up front to make your history atomic
36.  It's much more painful to split a mega-commit into atomic commits after the fact
37.  Splitting work atomically can be good for improving your reward drive - you can get many more things done
38.  Atomic commits work really well with [prefactoring][6]
39.  Sometimes prefactor commits can go into separate PRs (especially if squash-merge is used)
40.  Writing commit messages can take longer than the implementation
41.  The commit message can be an order of magnitude larger than the number of lines changed in a commit
42.  If you end up writing a lot of "and"s or "also" in a commit message, you may be trying to do too many things
43.  Trawling through Git commit history in the past has helped unlock a number of cases where I could then understand the _why_ without the original authors there to answer my questions
44.  Commit messages are a great point to reflect on not just what you've done, but **why**
45.  Why is more important than what - anyone can look at the diff and generally work out what changes were made, but the intent behind it is the special sauce
46.  If you only write what changed, you're annoying, and I dislike you
47.  A commit that explains what is better than a commit that just has "fixes"
48.  [Chris Beams][7]' article, [_How to Write a Git Commit Message_][8] is still an excellent post and a great place to start, just under 10 years later!
49.  Commits are a point-in-time explanation of the assumptions and the state of the world for the committer. Don't be too hard on them
50.  I don't want to read AI/LLM rewrites of your changes - either write it yourself, or call it `Various fixes`
51.  There needs to be a way to add an annotation (maybe using `git notes`) to a previous message to correct assumptions
52.  I won't write perfect commit messages up front - they'll sometimes be as much as `rew! add support for SBOMs` or `sq`, or [use `git commit --fixup`][9]
53.  I will, generally, break off very good atomic chunks of work into commits
54.  I will split atomic commits into multiple commits, sometimes
55.  Make sure you [review your own code changes][10] before you send it out for review to your collaborators
56.  Reviewing your commit messages should be as important as reviewing the code changes
57.  Getting all your contributors to invest the same amount of care into the commit history is a losing battle
58.  Trying to police commit history is going to be painful
59.  Trying to mandate reviews of commit messages as part of code review is going to be painful
60.  Trying to police commit history does lead to a greater level of documentation and consideration around changes int he codebase
61.  Making implicit assumptions explicit is really useful
62.  Introducing `commitlint` can be useful, but also frustrating
63.  It's nicer to have your collaborators want to write good commit messages than you having to force them
64.  Some people don't write, and that's alright
65.  Writing is a skill
66.  I'm not perfect at writing (commit messages)
67.  Sometimes I can't be arsed to write the perfect message
68.  Sometimes I write some really great commit messages, and impress myself
69.  Using a [template for your Git commit messages][11] is a good nudge to doing it right
70.  [`fixup` commits][12] and `git rebase --autosquash` has been one of the best Git tips I've learned
71.  I value working on a team with a diverse set of perspectives, skills and approaches to work
72.  But I also really value having a team who writes atomic commits with well-written commit messages
73.  Commit message writing is as useful as writing well-refined user stories/tickets
74.  `git commit -m sq` is probably my most-run command
75.  Using `git add -p` and `git commit -p` are hugely important for atomic commits
76.  Never use `git add -u` or `git add .`
77.  Learn when you can use `git add -u` or `git add .`
78.  I really need to look into tools like Graphite, `git-branchless` and other means to provide a stacked PR setup
79.  Using [conventional commits][13] with [semantic-release][14] or [go-semantic-release][15] can make a huge difference when wanting to release automagically and often
80.  Using [conventional commits][16] as a framework for your commits can be really useful
81.  Using [conventional commits][17], as someone with ADHD, reduces the need for thinking at times and can allow you to focus more on what the changes are
82.  Using [conventional commits][18] helps you work out when you're trying to do too much in a commit
83.  I [think through writing][19] so commit messages help understand [why I did the thing][20]
84.  It can be better to write a good commit message than a piece of documentation, stored elsewhere
85.  It can be better to write a good commit message than a code comment
86.  Give people space to learn
87.  Give people space to fail
88.  Remember you weren't so great at one point in time
89.  Documentation is rad. Do more of it

[1]: http://creativecommons.org/licenses/by-nc-sa/4.0/legalcode
[2]: http://www.apache.org/licenses/LICENSE-2.0
[3]: https://www.jvt.me/posts/2021/10/23/undo-force-push/
[4]: https://www.jvt.me/posts/2018/09/18/safely-force-git-push/
[5]: https://www.jvt.me/posts/2018/06/01/git-subtree-monorepo/
[6]: https://www.jvt.me/posts/2022/04/12/prefactor/
[7]: https://cbea.ms/
[8]: https://cbea.ms/git-commit/
[9]: https://www.jvt.me/posts/2019/01/10/git-commit-fixup/
[10]: https://www.jvt.me/posts/2019/01/12/self-code-review/
[11]: https://www.jvt.me/posts/2017/04/17/commit-templates/
[12]: https://www.jvt.me/posts/2019/01/10/git-commit-fixup/
[13]: https://www.conventionalcommits.org/en/v1.0.0/
[14]: https://github.com/semantic-release/semantic-release
[15]: https://github.com/go-semantic-release/semantic-release
[16]: https://www.conventionalcommits.org/en/v1.0.0/
[17]: https://www.conventionalcommits.org/en/v1.0.0/
[18]: https://www.conventionalcommits.org/en/v1.0.0/
[19]: https://www.jvt.me/posts/2023/10/04/blogging-neurodiversity/
[20]: https://www.youtube.com/watch?v=1qdyNoe3q2A