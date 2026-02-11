---
title: 如何将目录添加到你的 PATH
date: 2025-02-13T12:27:56.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/
translator: ""
reviewer: ""
---

• [terminal][1] •

<!-- more -->

今天我和一个朋友讨论如何将目录添加到你的 PATH。对于我这种长期使用终端的人来说，这感觉很"显而易见"，但当我搜索如何做这件事的说明时，我实际上找不到解释所有步骤的内容——很多只是说"将这个添加到`~/.bashrc`"，但如果你不使用 bash 怎么办？如果你的 bash 配置实际上在不同的文件中怎么办？而且你应该如何确定要添加哪个目录呢？

所以我想尝试写下一些更完整的指导，并提及我多年来遇到的一些陷阱。

以下是目录：

*   [步骤 1：你使用的是什么 shell？][2]
*   [步骤 2：找到你的 shell 配置文件][3]
    *   [关于 bash 配置文件的说明][4]
*   [步骤 3：确定要添加哪个目录][5]
    *   [步骤 3.1：再次检查它是正确的目录][6]
*   [步骤 4：编辑你的 shell 配置][7]
*   [步骤 5：重启你的 shell][8]
*   问题：
    *   [问题 1：它运行了错误的程序][9]
    *   [问题 2：程序不是从你的 shell 运行的][10]
    *   [问题 3：PATH 条目重复使调试更加困难][11]
    *   [问题 4：更新 PATH 后丢失历史记录][12]
*   注释：
    *   [关于 source 的说明][13]
    *   [关于 fish_add_path 的说明][14]

### [步骤 1：你使用的是什么 shell？][2]

如果你不确定你使用的是什么 shell，这里有一种方法可以找出来。运行这个：

```bash
ps -p $$ -o pid,comm=
```

*   如果你使用的是 **bash**，它会打印出 `97295 bash`
*   如果你使用的是 **zsh**，它会打印出 `97295 zsh`
*   如果你使用的是 **fish**，它会打印出一个错误，比如 "In fish, please use $fish\_pid"（`$$` 在 fish 中不是有效的语法，但无论如何错误消息告诉你你正在使用 fish，这你可能已经知道了）

另外，bash 是 Linux 的默认 shell，而 zsh 是 Mac OS 的默认 shell（截至 2024 年）。在这些指导中，我只会涵盖 bash、zsh 和 fish。

### [步骤 2：找到你的 shell 配置文件][3]

*   在 zsh 中，它可能是 `~/.zshrc`
*   在 bash 中，它可能是 `~/.bashrc`，但这很复杂，请参阅下一节中的说明
*   在 fish 中，它可能是 `~/.config/fish/config.fish`（如果你想 100% 确定，可以运行 `echo $__fish_config_dir`）

### [关于 bash 配置文件的说明][4]

Bash 有三个可能的配置文件：`~/.bashrc`、`~/.bash_profile` 和 `~/.profile`。

如果你不确定你的系统设置使用哪一个，我建议通过这种方式测试：

1.  在你的 `~/.bashrc` 中添加 `echo hi there`
2.  重启你的终端
3.  如果你看到 "hi there"，这意味着 `~/.bashrc` 正在被使用！太好了！
4.  否则，删除它并尝试对 `~/.bash_profile` 做同样的事情
5.  如果前两个选项不起作用，你也可以尝试 `~/.profile`。

（有很多[复杂的流程图][15]解释了 bash 如何决定使用哪个配置文件，但在我看来，没有必要去内化它们，直接测试是确定的最快方法）

### [步骤 3：确定要添加哪个目录][5]

假设你正在尝试安装并运行一个名为 `http-server` 的程序，但它不起作用，像这样：

```plain
$ npm install -g http-server
$ http-server
bash: http-server: command not found
```

你如何找到 `http-server` 在哪个目录中？老实说，一般来说这并不容易——通常答案是"取决于 npm 的配置方式"。一些想法：

*   通常在设置新的安装程序（如 `cargo`、`npm`、`homebrew` 等）时，当你首次设置它时，它会打印出一些关于如何更新你的 PATH 的指导。所以如果你注意的话，你可以在那时获得指导。
*   有时安装程序会自动更新你的 shell 配置文件以更新你的 `PATH`
*   有时只是谷歌搜索"npm 在哪里安装东西？"就会找到答案
*   一些工具有一个子命令，告诉你它们配置安装东西的位置，比如：
    *   Node/npm：`npm config get prefix`（然后附加 `/bin/`）
    *   Go：`go env GOPATH`（然后附加 `/bin/`）
    *   asdf：`asdf info | grep ASDF_DIR`（然后附加 `/bin/` 和 `/shims/`）

### [步骤 3.1：再次检查它是正确的目录][6]

一旦你找到了一个你认为可能是正确的目录，确保它确实是正确的！例如，我发现在我的机器上，`http-server` 在 `~/.npm-global/bin` 中。我可以通过尝试在该目录中运行程序 `http-server` 来确保它是正确的目录，像这样：

```bash
$ ~/.npm-global/bin/http-server
Starting up http-server, serving ./public
```

它工作了！现在你知道你需要添加到 `PATH` 的目录是什么，让我们进入下一步！

### [步骤 4：编辑你的 shell 配置][7]

现在我们有了需要的 2 个关键信息：

1.  你试图添加到 PATH 的目录（如 `~/.npm-global/bin/`）
2.  你的 shell 配置在哪里（如 `~/.bashrc`、`~/.zshrc` 或 `~/.config/fish/config.fish`）

现在你需要添加的内容取决于你的 shell：

**bash 指导：**

打开你的 shell 配置文件，添加一行像这样的内容：

```bash
export PATH=$PATH:~/.npm-global/bin/
```

（显然，用你实际尝试添加的目录替换 `~/.npm-global/bin`）

**zsh 指令:**

你可以做与 bash 相同的事情，但 zsh 也有一些稍微更高级的语法，如果你喜欢的话：

```bash
path=(
  $path
  ~/.npm-global/bin
)
```

**fish 指令:**

在 fish 中，语法不同：

```bash
set PATH $PATH ~/.npm-global/bin
```

（在 fish 中你也可以使用 `fish_add_path`，关于这个的一些说明[在下面][14]）

### [步骤 5：重启你的 shell][8]

现在，一个非常重要的步骤：如果你不重启 shell，更新 shell 配置不会生效！

有两种方法可以做到这一点：

1.  打开一个新的终端（或终端标签），也许关闭旧的以避免混淆
2.  运行 `bash` 启动一个新的 shell（如果你使用的是 zsh，则运行 `zsh`，如果你使用的是 fish，则运行 `fish`）

我发现这两种方法通常都可以正常工作。

你应该完成了！尝试运行你想运行的程序，希望它现在能工作。

如果不行，这里有一些你可能遇到的问题：

### [问题 1：它运行了错误的程序][9]

如果运行了错误**版本**的程序，你可能需要将目录添加到 PATH 的_开头_而不是结尾。

例如，在我的系统上，我安装了两个版本的 `python3`，我可以通过运行 `which -a` 看到：

```bash
$ which -a python3
/usr/bin/python3
/opt/homebrew/bin/python3
```

你的 shell 将使用的是**列出的第一个**。

如果你想使用 Homebrew 版本，你需要将该目录（`/opt/homebrew/bin`）添加到 PATH 的**开头**，在你的 shell 配置文件中放入这个（它是 `/opt/homebrew/bin/:$PATH` 而不是通常的 `$PATH:/opt/homebrew/bin/`）

```bash
export PATH=/opt/homebrew/bin/:$PATH
```

或者在 fish 中：

```bash
set PATH ~/.cargo/bin $PATH
```

### [问题 2：程序不是从你的 shell 运行的][10]

所有这些指导只有在你**从你的 shell** 运行程序时才有效。如果你从 IDE、GUI、cron 作业或其他方式运行程序，你需要以不同的方式将目录添加到你的 PATH，具体细节可能取决于情况。

**在 cron 作业中**

一些选项：

*   使用你正在运行的程序的完整路径，如 `/home/bork/bin/my-program`
*   将你想要的完整 PATH 作为 crontab 的第一行（类似于 PATH=/bin:/usr/bin:/usr/local/bin:……）。你可以通过运行 `echo "PATH=$PATH"` 获取你在 shell 中使用的完整 PATH。

我老实说不确定如何在 IDE/GUI 中处理它，因为我很久没有遇到这种情况了，如果有人指点我正确的方向，我会在这里添加指导。

### [问题 3：PATH 条目重复使调试更加困难][11]

如果你编辑你的 path 并通过运行 `bash`（或 `zsh`，或 `fish`）启动一个新的 shell，你通常会得到重复的 `PATH` 条目，因为每次你启动 shell 时，shell 都会继续向你的 `PATH` 添加新东西。

就我个人而言，我认为我没有遇到过这种重复会破坏任何东西的情况，但如果你试图理解 `PATH` 的内容，重复可能会使调试 `PATH` 发生的事情变得更加困难。

一些处理方法：

1.  如果你正在调试你的 `PATH`，打开一个新的终端来做，这样你就会得到一个"新鲜"的状态。这应该避免重复。
2.  在你的 shell 配置的末尾去重你的 `PATH`（例如在 zsh 中，显然你可以用 `typeset -U path` 做到这一点）
3.  添加目录时检查它是否已经在你的 `PATH` 中（例如在 fish 中，我相信你可以用 `fish_add_path --path /some/directory` 做到这一点）

如何去重你的 `PATH` 是特定于 shell 的，并且并不总是有内置的方法来做到这一点，所以你需要查找如何在你的 shell 中完成它。

### [问题 4：更新 PATH 后丢失历史记录][12]

这里有一个在 bash 或 zsh 中容易陷入的情况：

1.  运行一个命令（它失败了）
2.  更新你的 `PATH`
3.  运行 `bash` 重新加载你的配置
4.  按几次向上箭头重新运行失败的命令（或打开一个新的终端）
5.  失败的命令不在你的历史记录中！为什么？

这是因为在 bash 中，默认情况下，历史记录直到你退出 shell 才会保存。

一些修复选项：

*   不要运行 `bash` 重新加载你的配置，而是运行 `source ~/.bashrc`（或在 zsh 中运行 `source ~/.zshrc`）。这将在你当前会话中重新加载配置。
*   配置你的 shell 持续保存你的历史记录，而不是只在 shell 退出时保存历史记录。（如何做到这一点取决于你是使用 bash 还是 zsh，zsh 中的历史选项有点复杂，我不太确定最好的方法是什么）

### [关于 source 的说明][13]

当你第一次安装 `cargo`（Rust 的安装程序）时，它给你这些关于如何设置 PATH 的指导，这些指导根本没有提到特定的目录。

```bash
This is usually done by running one of the following (note the leading DOT):

. "$HOME/.cargo/env"        	# For sh/bash/zsh/ash/dash/pdksh
source "$HOME/.cargo/env.fish"  # For fish
```

这个想法是你将该行添加到你的 shell 配置中，他们的脚本会自动为你设置 `PATH`（可能还有其他东西）。

这是相当常见的（例如 [Homebrew][16] 建议你执行 `brew shellenv`），有两种方法可以处理这个：

1.  只做工具建议的事情（比如将 `. "$HOME/.cargo/env"` 添加到你的 shell 配置中）
2.  找出他们告诉你运行的脚本会将哪些目录添加到你的 PATH 中，然后手动添加这些目录。以下是我的做法：
    *   在我的 shell 中运行 `. "$HOME/.cargo/env"`（如果使用 fish，则运行 fish 版本）
    *   运行 `echo "$PATH" | tr ':' '\n' | grep cargo` 来确定它添加了哪些目录
    *   看到它说 `/Users/bork/.cargo/bin` 并将其缩短为 `~/.cargo/bin`
    *   将目录 `~/.cargo/bin` 添加到 PATH（使用本文中的指导）

我认为按照工具建议做没有什么问题（它可能是"最好的方法"！），但个人而言，我通常使用第二种方法，因为我更喜欢确切地知道我正在更改什么配置。

### [关于 fish_add_path 的说明][14]

fish 有一个方便的函数叫做 `fish_add_path`，你可以运行它来将目录添加到你的 `PATH`，像这样：

```bash
fish_add_path /some/directory
```

这很酷（这是一个如此简单的命令！）但我已经停止使用它，原因有几个：

1.  有时 `fish_add_path` 会更新未来每个会话的 `PATH`（使用"通用变量"），有时它只会更新当前会话的 `PATH`，我很难分辨它会做哪一个。理论上文档解释了这一点，但我无法理解它们。
2.  如果你几周或几个月后需要从你的 `PATH` 中_移除_目录，因为也许你犯了一个错误，这有点难做到（不过在这个 github issue 的评论中有[指导][17]）。

### [就是这些][18]

希望这能帮助一些人。如果你在将目录添加到 PATH 时遇到了其他主要陷阱，或者你对这篇文章有疑问，请让我知道（在 Mastodon 或 Bluesky 上）！

[2]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-1-what-shell-are-you-using
[3]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-2-find-your-shell-s-config-file
[4]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#a-note-on-bash-s-config-file
[5]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-3-figure-out-which-directory-to-add
[6]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-3-1-double-check-it-s-the-right-directory
[7]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-4-edit-your-shell-config
[8]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#step-5-restart-your-shell
[9]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#problem-1-it-ran-the-wrong-program
[10]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#problem-2-the-program-isn-t-being-run-from-your-shell
[11]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#problem-3-duplicate-path-entries-making-it-harder-to-debug
[12]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#problem-4-losing-your-history-after-updating-your-path
[13]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#a-note-on-source
[14]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#a-note-on-fish-add-path
[15]: https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html
[16]: https://github.com/Homebrew/install/blob/deacfa6a6e62e5f4002baf9e1fac7a96e9aa5d41/install.sh#L1072-L1087
[17]: https://github.com/fish-shell/fish-shell/issues/8604
[18]: http://jvns.ca/blog/2025/02/13/how-to-add-a-directory-to-your-path/#that-s-all
