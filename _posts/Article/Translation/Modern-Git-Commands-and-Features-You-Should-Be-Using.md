---
title: 你应该使用的现代 Git 命令和功能
updated: 2024-04-05 19:25:00
authors:
    - luojiyin1987
original: https://martinheinz.dev/blog/109
reviewer: TechQuery
categories:
    - Article
    - Translation
toc: true
---

我们所有软件工程师每天都在使用 `git`，但大多数人只接触过最基本的命令，如 `add`、`commit`、`push` 或者 `pull`, 好像还停留在 2005 年。

不过，Git 从那时起引入了许多功能，使用它们能让你的生活变得更轻松，下面就让我们来了解一下最近添加的一些现代 Git 命令。

## Switch

自 2019 年以来，或者更准确地说，自 Git 2.23 版引入以来，我们可以使用 `git switch` 来切换分支：

```bash
git switch other-branch
git switch -  # 切换回上一个分支，类似于 "cd -"
git switch remote-branch  # 直接切换到远程分支并开始跟踪
```

`git checkout` 是一个非常灵活的命令。它可以（除其他外）签出或恢复特定文件甚至特定提交，而新的 `git switch` 只能切换分支。此外，`switch` 还会执行额外的正确性检查，而 `checkout` 则不会，例如，如果会导致本地改动丢失，`switch` 就会中止操作。

## Restore

Git 2.23 版新增的另一个子命令/功能是 `git restore`，我们可以用它将文件恢复到上次提交的版本：

```bash
# 取消对文件的修改，与 "git reset some-file.py" 相同
git restore --staged some-file.py

# 取消并丢弃对文件所做的更改，与 "git checkout some-file.py" 相同
git restore --staged --worktree some-file.py

# 将文件恢复到之前的某个提交，与 "git reset commit -- some-file.py" 相同
git restore --source HEAD~2 some-file.py
```

上述代码段中的注释解释了各种 `git restore` 使用。一般来说， `git restore` 替换和简化 `git reset` 和 `git checkout` 的使用场景，它们的功能过于复杂。关于 `revert`、`restore` 和 `reset` 的比较，请参阅本文档。

## Sparse Checkout

下一个是 `git sparse-checkout`，这是在 2020 年 1 月 13 日发布的 Git 2.25 中添加的一个不起眼的功能。

比方说，你有一个大的 `monorepo`，其中的微服务被分隔到各个目录中，由于版本库太大，`checkout` 或 `status` 等命令执行起来超级慢，但也许你真的只需要处理单个子树/目录。那么，`git sparse-checkout` 就能帮到你：

```bash
$ git clone --no-checkout https://github.com/derrickstolee/sparse-checkout-example
$ cd sparse-checkout-example
$ git sparse-checkout init --cone  # 配置 git， 只匹配根目录下的文件
$ git checkout main  # 只检出根目录中的文件
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

在上面的例子中，我们首先 `clone repo`，但并没有 `checkout` 所有文件。然后使用 `git sparse-checkout init --cone` 配置 git 只匹配仓库根目录下的文件。这样，在运行签出后，我们只有 3 个文件，而不是整棵树。要下载/检出特定目录，我们使用 `git sparse-checkout set ....`

如前所述，这在本地处理庞大的版本库时非常方便，但在 CI/CD 中，当你只想构建/部署 `monorepo` 的一部分，而不需要检出所有内容时，这对提高流水线性能同样有用。

有关 `sparse-checkout` 的详细介绍，请参阅本文。

## Worktree

一个人可能需要同时在单个应用程序（版本库）中开发多个功能，或者当你正在处理某个功能请求时，可能会出现一个 `critical` 级别的错误，这种情况并不少见。

在这种情况下，您要么需要克隆多个版本/分支的版本库，要么就需要隐藏/丢弃当时正在处理的内容。2018 年 9 月 24 日发布的 `git worktree` 就是解决这些情况的办法：

```bash
git branch
# * dev
# master

git worktree list
# /.../some-repo  ews5ger [dev]

git worktree add -b hotfix ./hotfix master

# 准备 worktree (new branch 'hotfix')
# HEAD 现在是 5ea9faa 已签名提交。

git worktree list
# /.../test-repo         ews5ger [dev]
# /.../test-repo/hotfix  5ea9faa [hotfix]

cd hotfix/  # 干净的 worktree, 你可以在其中进行修改并推送
```

该命令允许我们同时签出同一版本库的多个分支。在上面的例子中，我们有两个分支 dev 和 master。假设我们正在开发分支中开发功能，但有人告诉我们要进行紧急错误修复。与其将更改存储起来并重置分支，不如在主分支的 ./hotfix 子目录下创建一个新的 worktree。然后，我们就可以移动到该目录，进行修改、推送并返回到原始 worktree。

更多详细内容，请参阅本文。

## Bisect

最后但并非最不重要的是 `git bisect`，它并不新鲜（Git 1.7.14，2012 年 5 月 13 日发布），但大多数人只使用 2005 年左右的 git 功能，所以我觉得还是值得展示一下。

正如文档页面所描述的：`git bisect`，使用二进制搜索查找引入错误的提交：

```bash
git bisect start
git bisect bad HEAD  # 提供出问题的提交
git bisect good 479420e  # 提供你知道正常运行的提交
# Bisecting: 2 revisions left to test after this (roughly 1 step)
# [3258487215718444a6148439fa8476e8e7bd49c8] Refactoring.

# 测试当前提交...
git bisect bad  # If the commit doesn't work
git bisect good # If the commit works

# 根据最后一条命令，Git 在 bad 与 good 之间进行二分查找
# 继续测试直到找原因

git bisect reset  # 重置为初始提交
```

我们先用 `git bisect start` 显式启动分段会话，然后提供不工作的提交（bad 的提交，很可能是 HEAD）和最后一次已知的正常运行提交或标记。有了这些信息，git 就会检查出介于 `bad` 和 `good` 提交之间的一个提交。这时，我们需要测试该版本是否存在漏洞，然后用 git bisect good 告诉 git 它能正常工作，或用 git bisect bad 告诉 git 它不能正常工作。我们不断重复这个过程，直到没有提交，git 就会告诉我们哪个提交引入了问题。

我建议你去文档页面看看，那里有更多关于 `git bisect` 的选项，包括可视化、重放或跳过提交。

如果你搜索一些与 `git` 相关的问题，你很可能会在 StackOverflow 上找到有几千个向上投票的答案的问题。虽然这个答案很可能仍然有效，但很可能已经过时，因为它是 10 年前写的。因此，可能还有更好、更简单、更容易的方法。因此，当遇到一些 `git` 问题时，我建议查看 `git` 文档，了解最新的命令，这些命令都有很多很好的示例，或者浏览 `man` 页面，了解多年来添加到老命令中的很多标记（flags）和选项（options）。
