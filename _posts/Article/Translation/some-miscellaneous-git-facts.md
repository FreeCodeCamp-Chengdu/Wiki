---
title: Julia Evans
authorURL: ""
originalURL: https://jvns.ca/blog/2023/10/20/some-miscellaneous-git-facts/
translator: ""
reviewer: ""
---

# 一些杂七杂八的 git 知识

我一直在慢慢地写关于 Git 工作原理的文章。我以为自己已经对 Git 相当了解了，但像往常一样，当我试图解释一些东西时，我又学到了一些新东西。

现在回想起来，这些事情都不会让人感到惊讶，但我之前并没有想清楚。

事实是:

- [一些杂七杂八的 git 知识](#一些杂七杂八的-git-知识)
  - [`index`， `staging area` 和 `–cached` 都是一回事](#index-staging-area-和-cached-都是一回事)
  - [储藏包含多个提交](#储藏包含多个提交)
  - [并非所有引用都是分支或标签](#并非所有引用都是分支或标签)
  - [merge commits（合并提交）不是空的](#merge-commits合并提交不是空的)
  - [就这样](#就这样)

让我们来谈谈它们！

<!-- more -->

## `index`， `staging area` 和 `–cached` 都是一回事

运行 `git add file.txt`，然后运行 `git status`，就会看到这样的内容：

```shell
git add content/post/2023-10-20-some-miscellaneous-git-facts.markdown
git status
```

```text
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   content/post/2023-10-20-some-miscellaneous-git-facts.markdown
```

人们通常称之为 `staging a file（暂存文件）` or `adding a file to the staging area（将文件添加到暂存区域）`.

当你用 `git add` 添加文件时，git 会在幕后将文件添加到它的对象数据库（在 `.git/objects`）中，并更新一个名为 `.git/index` 的文件来引用新添加的文件。

在 Git 中，这个 `staging area(暂存区)`实际上有三个不同的名字。所有这些都指向同一个东西（文件 `.git/index`）：

-   `git diff --cached`
-   `git diff --staged`
-   这个文件对应 `.git/index`

我觉得我应该早点意识到这一点，但我没有，所以就这样了。

## 储藏包含多个提交

当我运行 `git stash` 来储藏我的更改时，我一直对这些更改的去向感到困惑。事实证明，当你运行 `git stash` 时，git 会用你的更改创建一些提交，并用一个名为 `stash` 的引用标记它们（位于 `.git/refs/stash` 中）。

我们先把这篇博文储藏起来，然后查看 `stash` 引用的日志：

```shell
git log stash --oneline
```

```text
6cb983fe (refs/stash) WIP on main: c6ee55ed wip
2ff2c273 index on main: c6ee55ed wip
... some more stuff
```

现在我们可以查看 `2ff2c273` 提交，看看它包含了什么内容：

```shell
git show 2ff2c273  --stat
```

```text
commit 2ff2c273357c94a0087104f776a8dd28ee467769
Author: Julia Evans <julia@jvns.ca>
Date:   Fri Oct 20 14:49:20 2023 -0400

    index on main: c6ee55ed wip

 content/post/2023-10-20-some-miscellaneous-git-facts.markdown | 40 ++++++++++++++++++++++++++++++++++++++++
```

不出所料，它包含了这篇博文。这很合理！

实际上，`git stash` 会创建两个独立的提交：一个是索引提交，另一个是你尚未暂存的改动提交。我发现这有点振奋人心，因为我一直在开发一个工具来快照和还原 git 仓库的状态（我可能会发布，也可能不会发布），我想出了一个非常类似的设计，所以这让我对我的选择感觉更好。

显然，stash 中的旧提交会保存在 reflog 中。

## 并非所有引用都是分支或标签

Git 文档中经常泛泛地提到 `references（引用）`，有时我觉得有点混乱。就我个人而言，在 Git 中，99% 的情况下，`references（引用）`指的是`branch（分支）`或 `HEAD`，而另外 1% 的情况下，`references（引用）`指的是 `tags（标签）`。实际上，我以前并不知道有什么引用不是分支、标签或 `HEAD` 的例子。

但是现在我知道了一个例子, `stash（储藏）`是一个引用，它不是分支或标签！所以这很酷。

以下是我的博客 git 仓库中的所有引用（除了 HEAD）：

```shell
find .git/refs -type f
```

```text
.git/refs/heads/main
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/main
.git/refs/stash
```

显然，还有一个名为 [`git notes`](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/) 的 git 命令，它可以在 `.git/refs/notes` 下创建引用。

## merge commits（合并提交）不是空的

下面是我创建的两个分支 `x` 和 `y`，每个分支都有一个文件（`x.txt` 和 `y.txt`），并将它们合并。让我们看看合并提交。

```shell
git log --oneline
```

```text
96a8afb (HEAD -> y) Merge branch 'x' into y
0931e45 y
1d8bd2d (x) x
```

如果运行 `git show 96a8afb`，提交看起来是空白的：没有差异！

```shell
git show 96a8afb
```

```text
commit 96a8afbf776c2cebccf8ec0dba7c6c765ea5d987 (HEAD -> y)
Merge: 0931e45 1d8bd2d
Author: Julia Evans <julia@jvns.ca>
Date:   Fri Oct 20 14:07:00 2023 -0400

    Merge branch 'x' into y
```

但是如果我将合并提交分别与它的两个父提交进行比较，你会发现确实存在差异：

```shell
git diff 0931e45 96a8afb   --stat
```

```text
 x.txt | 1 +
 1 file changed, 1 insertion(+)
```

```shell
git diff 1d8bd2d 96a8afb   --stat
```

```text
 y.txt | 1 +
 1 file changed, 1 insertion(+)
```

现在回想起来，合并提交实际上并非“空的”（它们是仓库当前状态的快照，就像任何其他提交一样），这似乎很明显，但我从未想过为什么它们看起来是空的。

显然，这些合并差异为空的原因是合并差异只显示冲突。如果我创建一个包含合并冲突的仓库（一个分支在同一个文件中添加了 `x`，另一个分支添加了 `y`），并显示我解决冲突的合并提交，它看起来像这样：

```shell
git show HEAD
```

```text
commit 3bfe8311afa4da867426c0bf6343420217486594
Merge: 782b3d5 ac7046d
Author: Julia Evans <julia@jvns.ca>
Date:   Fri Oct 20 15:29:06 2023 -0400

    Merge branch 'x' into y

diff --cc file.txt
index 975fbec,587be6b..b680253
--- a/file.txt
+++ b/file.txt
@@@ -1,1 -1,1 +1,1 @@@
- y
 -x
++z
```

看起来它试图告诉我一个分支添加了 `x`，另一个分支添加了 `y`，而合并提交通过改为放置 `z` 来解决它。但在前面的例子中，没有冲突，所以 Git 根本没有显示差异。

（感谢 Jordi 告诉我合并差异是如何工作的）

## 就这样

我会尽量保持这篇博文的简短，也许等我学到更多 Git 知识后会再写一篇博文。
