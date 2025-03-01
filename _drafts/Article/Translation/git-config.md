---
title: 我是如何配置 Git 身份认证
date: 2024-11-22T00:00:00.000Z
author: benji
authorURL: https://benji.dog/
originalURL: https://www.benji.dog/articles/git-config/
translator: ""
reviewer: ""
---

> **注意**：这篇文章我已经写了 3 年！！！终于到了发布的时候了。

我喜欢折腾我的 [dotfiles](https://github.com/benjifs/dotfiles)，时不时会发现一些新的方法，然后花上比预期更多的时间去学习如何使用它。

几年前，我了解到 Git 的 [includeIf](https://git-scm.com/docs/git-config#_includes) 功能，它可以在满足某些条件时包含特定的文件。我最初看到的例子是这样的：

```
[includeIf "gitdir:~/code/**"]
  path = ~/.config/git/personal
[includeIf "gitdir:~/work/**"]
  path = ~/.config/git/work
```

这样，`~/.config/git/personal` 只会被包含在 `~/code` 下的 Git 目录中，而 `~/.config/git/work` 则只会被包含在 `~/work` 下的目录中。这些被包含的文件内容各不相同，但通常包含你的 Git 身份、签名密钥等。以下是一个示例：

```
[user]
  name = benji
  email = benji@work.com
  signingkey = ~/.ssh/work.id_ed25519.pub
```

这种方式效果不错，但我通常将所有代码都放在 `~/workspace` 中，无论是个人项目、**工作项目-1** 还是 **工作项目-2** 等。我希望能够根据仓库的实际位置来配置 Git，而不是根据它在我的机器上的目录位置。然后我发现了 [hasconfig:remote.\*.url:](https://git-scm.com/docs/git-config#Documentation/git-config.txt-codehasconfigremoteurlcode)！

这让我可以根据当前工作目录的远程 URL 来条件性地配置 Git。

以下是我做的一些示例：

```
[includeIf "hasconfig:remote.*.url:git@github.com:orgname/**"]
  path = ~/.config/git/config-gh-org

[includeIf "hasconfig:remote.*.url:git@github.com:*/**"]
  path = ~/.config/git/config-gh

[includeIf "hasconfig:remote.*.url:git@gitlab.com:*/**"]
  path = ~/.config/git/config-gl

[includeIf "hasconfig:remote.*.url:git@git.sr.ht:*/**"]
  path = ~/.config/git/config-srht
```

现在，如果我在一个远程 URL 匹配 `github.com:orgname/**` 的目录中，它会使用 `~/.config/git/config-gh-org`，否则它会使用适用于其他 GitHub 仓库的通用配置文件。

* * *

虽然这解决了 Git 身份的问题，但我仍然需要单独配置 SSH 密钥以便能够 `pull` 和 `push` 到远程仓库。我的 `~/.ssh/config` 的简化版本如下：

```
Host gitlab.com
Hostname gitlab.com
User git
IdentityFile ~/.ssh/gitlab.id_ed25519

Host github.com
Hostname github.com
User git
IdentityFile ~/.ssh/github.id_ed25519
```

唯一的问题是，如果我想为同一个 `Hostname` 使用不同的 `IdentityFile`，以便为 `github.com/orgname` 下的仓库使用不同的密钥，我必须为 `Host` 使用不同的值。因此，我会在我的 `~/.ssh/config` 中添加以下内容：

```
Host gh-work
Hostname github.com
User git
IdentityFile ~/.ssh/work.id_ed25519
```

最后，为了在查找 `github.com/orgname` 下的仓库时使用这个 `Host`，我会在我的 Git 配置中添加以下内容：

```
[url "gh-work:orgname"]
  insteadOf = git@github.com:orgname
```

这样，当我 `clone`、`pull` 或 `push` 一个属于我工作组织账户的仓库时，我可以这样做：

```
git clone git@github.com:orgname/project
```

而 `insteadOf` 会将 `github.com:orgname` 替换为 `gh-work:orgname`，从而使用 SSH 配置中的正确信息。这是一个我在 [这篇文章](https://www.kenmuse.com/blog/ssh-and-multiple-git-credentials/#git) 中看到的巧妙技巧。

* * *

这种方法有什么问题吗？有没有更好的方式？我不确定，所以请告诉我，我很乐意学习，并会相应地更新这篇文章。

参考资料
----------

*   [https://fundor333.com/post/2021/advance-git-config-and-ssh-config/](https://fundor333.com/post/2021/advance-git-config-and-ssh-config/)
*   [https://www.kenmuse.com/blog/ssh-and-multiple-git-credentials/#git](https://www.kenmuse.com/blog/ssh-and-multiple-git-credentials/#git)
*   [https://garrit.xyz/posts/2023-10-13-organizing-multiple-git-identities](https://garrit.xyz/posts/2023-10-13-organizing-multiple-git-identities)
*   [https://stevenharman.net/configure-ssh-keys-for-multiple-github-accounts](https://stevenharman.net/configure-ssh-keys-for-multiple-github-accounts)