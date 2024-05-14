---
title: Julia Evans
authorURL: ""
originalURL: https://jvns.ca/blog/2024/04/10/notes-on-git-error-messages/
translator: ""
reviewer: ""
---

# 关于 git 错误信息的说明

<!-- more -->

在写关于 Git 的文章时，我注意到很多人都在纠结 Git 的错误信息。我已经习惯这些错误信息很多年了，所以花了很长时间才明白大家为什么会困惑：

1.  有时我确实被错误信息弄糊涂了，我只是习惯了被弄糊涂而已
2.  当 git 给我的错误信息不是很有参考价值时，我有很多策略来获取更多信息。

所以，在这篇文章里，我将逐一分析 Git 的错误信息，列出每条信息中我认为容易混淆的地方，并谈谈当我被错误信息弄糊涂时该怎么做。

### 改进错误信息并不容易

在开始之前，我想说的是，通过思考这些错误信息令人困惑的原因，让我对维护 Git 的难度肃然起敬。几个月来我一直在思考 Git 的问题，但对于其中一些错误信息，我真的不知道该如何改进。

在我看来，改进错误信息有以下几点困难：

-   如果你想出一个新信息的创意，很难说它是否真的更好！
-   改进错误信息之类的工作经常[得不到资助][1]
-   错误信息必须翻译（git 的错误信息被翻译成 [19 种语言][2]）

也就是说，如果你觉得这些消息令人困惑，希望这些注释能够在一定程度上帮助澄清它们。

## [error: `git push` on a diverged branch][3]

```shell
$ git push
To github.com:jvns/int-exposed
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'github.com:jvns/int-exposed'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 1 different commits each, respectively.
```
我觉得有些事情令人困惑:

1.  无论分支是 **behind（落后）** 还是 **diverged（偏离）**，您都会收到完全相同的错误信息。从这条信息中无法判断是哪个分支：您需要运行 `git status` 或 `git pull` 才能知道。
2.  它说 `failed to push some refs（推送某些引用失败）`，但并不完全清楚推送失败的是哪些引用。我相信所有推送失败的引用都会以 `！[rejected]`在前一行，在本例中，只有 `main` 分支。

**困惑时我喜欢做的事:**

- 我会运行 `git status` 来了解当前分支的状态。
- 我想我几乎从来没有尝试过一次推送多个分支，所以我通常会完全忽略 git 关于哪个分支推送失败的说明 - 我只是假设它就是我的当前分支

## [error: `git pull` on a diverged branch][4]
```shell
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
```
我认为这里最令人困惑的是，git 给你提供了大量的选项：它说，你可以任选其一：

1.  在本地配置 `pull.rebase false`, `pull.rebase true`, 或者 `pull.ff only` 
2.  或全局配置
3.  或运行 `git pull --rebase` 或者 `git pull --no-rebase`

很难想象一个初学 git 的人如何能轻松地利用这个提示，自己整理出所有这些选项。

如果我向朋友解释这个问题，我会说 "你可以用`git pull --rebase` 或`git pull --no-rebase` 来解决这个问题。如果你想设置一个永久的偏好，你可以用`git config pull.rebase false` 或`git config pull.rebase true`。

我觉得 `git config pull.ff only` 有点多余，因为这是 git 的默认行为（虽然并不总是这样）。

**我喜欢在这里做:**

-   运行 `git status` 查看当前分支的状态
-   也许运行 `git log origin/main` 或 `git log` 查看分支提交的情况
-   通常会运行 `git pull --rebase` 来解决它
-   如果我想丢弃本地提交（local work）或远程提交（remote work ），有时会运行 `git push --force` 或 `git reset --hard origin/main`（例如，因为我不小心提交到了错误的分支，或者因为我在一个只有我在用的个人分支上运行了 `git commit --amend` 并想强制推送）。

## [error: `git checkout asdf` (a branch that doesn't exist)][5]

```shell
$ git checkout asdf
error: pathspec 'asdf' did not match any file(s) known to git
```
这有点奇怪，因为我的意图是检出一个**分支**，但`git checkout`却在抱怨一个不存在的**文件路径（path）**。

出现这种情况是因为 `git checkout` 的第一个参数既可以是分支也可以是文件路径，而 git 无法知道你的意图是哪个。要改进这一点似乎很棘手，但我可能会期待类似 `No such branch, commit, or path: asdf（没有这样的分支、提交或路径：asdf）`这样的提示。

**我喜欢在这里做:**

-   理论上，用 `git switch` 代替会更好，切换分支，但我还是一直用 `git checkout`。
-   一般来说，我只记得我需要把它理解为分支 `asdf` 不存在。

## [error: `git switch asdf` (a branch that doesn't exist)][6]
```shell
$ git switch asdf
fatal: invalid reference: asdf
```
`git switch` 只接受分支作为参数（除非你传递了 `-d`），那它为什么会说 `invalid reference: asdf` 而不是 `invalid branch: asdf` 呢？

我认为原因在于，在内部，`git switch` 试图在其错误信息中提供帮助：如果你运行 `git switch v0.1` 来切换到一个标签，它会说：

```shell
$ git switch v0.1
fatal: a branch is expected, got tag 'v0.1'`
```

所以，git 试图通过 `fatal: invalid reference: asdf` 传达的意思是“`asdf`不是一个分支，但也不是一个标签，或者其他任何引用”。从我的各种[git polls][7]来看，我的印象是很多 git 用户根本不知道什么是 git 中的 `reference（引用）`，所以我不确定他们是否理解了这一点。

**我喜欢在这里做:**

90% 的情况下，当 git 错误信息中出现 `reference` 时，我都会在脑海中把它替换成 `branch`。

## error: [`git checkout HEAD^`][8]
```shell
$ git checkout HEAD^
Note: switching to 'HEAD^'.
```
您处于 `detached HEAD（分离的 HEAD）`状态。您可以四处看看，进行试验性改动并提交，还可以通过切换回分支来放弃在此状态下所做的任何提交，而不会影响任何分支。
状态下的任何提交，而不会影响任何分支。

如果你想创建一个新的分支来保留你创建的提交，可以在 switch 命令中使用 -c 来实现（现在或以后）。示例
```shell
  git switch -c 
```  
 或者使用:
```shell
  git switch -
```
通过将配置变量 `advice.detachedHead` 设为 `false` 关闭该建议

HEAD 现在的位置是 182cd3f，添加 `swap byte order` 按键

这是一个难题。肯定有很多人对这条信息感到困惑，但显然也有很多人在努力改进它。对于这个问题，我没什么好说的。

**我喜欢在这里做:**

-   我的 shell 提示会告诉我是否处于分离的 HEAD 状态，一般来说，在这种状态下我不会提交新的内容。
-   当我看完我想看的旧提交后，我会运行 `git checkout main` 或其他命令返回到某个分支

## [message: `git status` when a rebase is in progress][9]

这不是一条错误信息，但我还是觉得它本身有点令人困惑:
```shell
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
```
我认为有两点可以说得更清楚:

1.  如果把 `You are currently rebasing branch 'main' on 'c694cf8'.（您正在重定向 ‘c694cf8’ 上的分支main）` 放在第一行而不是第五行，我觉得会更好。现在第一行并没有说明您正在重定向哪个分支。
2.  在本例中， `c694cf8` 实际在 `origin/main`， 所以我觉得 `You are currently rebasing branch 'main' on 'origin/main'（您正在‘origin/main’上重定向分支‘main’）`可能更清楚。

**我喜欢在这里做:**

我的 shell 提示包括了当前正在重置（rebasing）的分支，所以我依赖它而不是 `git status` 的输出。

## [error: `git rebase` when a file has been deleted][10]
```shell
$ git rebase main
CONFLICT (modify/delete): index.html deleted in 0ce151e (wip) and modified in HEAD.  Version HEAD of index.html left in tree.
error: could not apply 0ce151e… wip
```
我仍然感到困惑的是，`index.html` 是在 `HEAD` 中修改的。但什么是 `HEAD`？是我开始合并（merge）/重置（rebase）时的提交，还是另一个分支的提交？ 答案是如果是合并，`HEAD` 就是你的分支，如果是重置，它就是 `另一个分支`，但我总觉得很难记住。

我个人认为，如果能在信息中列出分支名称，会更容易理解，就像这样：

```
CONFLICT (modify/delete): index.html deleted on `main` and modified on `mybranch`
```

## [error: `git status` during a merge or rebase (who is “them”?)][11]
```shell
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run “git commit”)
  (use “git merge –abort” to abort the merge)

Unmerged paths:
  (use “git add/rm …” as appropriate to mark resolution)
    deleted by them: the\_file

no changes added to commit (use “git add” and/or “git commit -a”)
```
我觉得这条信息和上一条信息一样让人困惑：上面写着 `deleted by them:`，但 `they` 指的是什么，取决于你是进行了合并（merge）、重置（rebase）还是 `cherry-pick`。

-   对于 merge，`them` 是您合并进来的另一个分支
-   对于 rebase，`them` 是您运行 `git rebase` 时所在的分支
-   对于 cherry-pick， 我猜这是你 cherry-picked 的提交

**困惑时我喜欢做的事:**

-   努力回忆我做过的事
-   如果我不记得了，运行 `git show main --stat` 或别的什么，看看我在 `main` 分支上做了什么

## [error: `git clean`][12]
```shell
$ git clean
fatal: clean.requireForce defaults to true and neither -i, -n, nor -f given; refusing to clean
```
我只是觉得这有点令人困惑，你需要查一下 `-i`、`-n` 和 `-f` 是什么意思才能理解这个错误信息。我个人太懒了，所以即使我用了 10 年的 `git clean` 也不知道 `-i` 代表什么（`interactive`），直到我写下这篇文章。

**困惑时我喜欢做的事:**

通常，我只是胡乱运行 `git clean -f` 来删除所有未跟踪的文件，并寄希望于最好的结果，不过现在我知道了 `-i` 代表什么，也许会改用 `git clean -i`。看起来安全多了。

### 就这样!

希望这些内容对您有所帮助！

[编织仙人掌][13]

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