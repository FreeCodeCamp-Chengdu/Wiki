---
title: 安装 "现代化 "终端需要什么？
date: 2025-01-11T09:46:01.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/
translator: ""
reviewer: ""
---





你好！最近我进行了一项终端调查，询问了人们在使用终端时遇到的困扰。有人评论道：
>  要获得现代化的终端体验，需要配置的东西太多了。我希望这些功能都能开箱即用。

我的第一反应是"哦，获得现代化的终端体验并不难，你只需要……"，但越想越觉得这个"你只需要……"的清单越来越长，而且我不断想到更多的注意事项。
所以我想写下一些笔记，记录下我个人认为什么是"现代化"的终端体验，以及为什么人们可能难以实现这一点。

### [什么是"现代化终端体验"？][1]

以下是一些对我来说很重要的功能，以及它们由系统的哪部分负责：

*   **多行复制粘贴支持**: 如果你在 shell 中粘贴了 3 条命令，它不应该立即全部执行！这很危险！ (**shell**, **终端模拟器**)
*   **无限 shell 历史记录**: 如果我在 shell 中运行了一条命令，它应该被永久保存，而不是在 500 条历史记录后被删除。此外，我希望命令在运行时立即保存到历史记录中，而不是仅在退出 shell 会话时保存 (**shell**)
*   **有用的提示符**: 我无法忍受提示符中没有**当前目录**和**当前 git 分支** (**shell**)
*   **24 位色支持**: 这对我来说很重要，因为我发现使用 24 位色支持来配置 neovim 比在仅支持 256 色的终端中要容易得多(**终端模拟器**)
*   **vim 和操作系统之间的剪贴板集成**, 这样当我在 Firefox 中复制时，我可以在 vim 中按 `p` 粘贴 (**文本编辑器**, 可能还有操作系统/终端模拟器)
*   **良好的自动补全**: 例如，像 git 这样的命令应该有特定于命令的自动补全(**shell**)
*   **在 `ls` 中显示颜色** (**shell 配置**)
*   **我喜欢的终端主题**: 我在终端中花费了大量时间，我希望它看起来漂亮，并且它的主题与我的终端编辑器的主题相匹配。(**终端模拟器** **文本编辑器**)
*   **自动终端修复**: 如果程序打印了一些奇怪的转义代码，搞乱了我的终端，我希望它能自动重置，这样我的终端就不会被搞乱 (**shell**)
*   **快捷键绑定**: 我希望 `Ctrl+左箭头` 能正常工作 (**shell** 或者 **应用程序**)
*   **能够在像 `less` 样的程序中使用滚轮**: (**终端模拟器r** and **应用程序**)

还有无数其他终端便利功能，不同的人看重不同的东西，但这些都是我无法接受没有的功能。

### [我如何实现"现代化体验"][2]

我的基本方法：

1.  使用 `fish` shell。基本上不配置它，除了：
    *   将 `EDITOR` 环境变量设置为我最喜欢的终端编辑器
    *   将 `ls` 别名为 `ls --color=auto`
2.  使用任何支持 24 位色的终端模拟器。过去我使用过 GNOME Terminal、Terminator 和 iTerm，但我不太挑剔。除了选择字体外，我基本上不配置它。
3.  使用 `neovim`，并配置一个我过去 9 年左右一直在慢慢构建的配置（我上次删除 vim 配置并从头开始是 9 年前）
4.  使用 [base16 框架][3] 来为主题化一切

影响我方法的一些因素：

*   我不经常 SSH 到其他机器
*   我宁愿使用鼠标一点，也不愿想出基于键盘的方法来做所有事情
*   我从事许多小型项目，而不是一个大型项目

### [一些"开箱即用"的"现代化"体验选项][4]

如果你想要一个良好的体验，但不想花太多时间在配置上呢？弄清楚如何配置 vim 让我满意确实花了我大约十年的时间，这太长了！

我关于如何以最少的配置获得合理的终端体验的最佳想法是：

*   shell：`fish` 或带有 [oh-my-zsh][5] 的 `zsh`
*   终端模拟器：几乎任何支持 24 位色的东西，例如以下这些都很受欢迎：
    *   linux：GNOME Terminal、Konsole、Terminator、xfce4-terminal
    *   mac：iTerm（Terminal.app 不支持 256 色）
    *   跨平台：kitty、alacritty、wezterm 或 ghostty
*   shell 配置：
    *   将 `EDITOR` 环境变量设置为你最喜欢的终端文本编辑器
    *   可能将 `ls` 别名为 `ls --color=auto`
*   文本编辑器：这是一个棘手的问题，也许是 [micro][6] 或 [helix][7]？我没有认真使用过它们，但它们看起来都是非常酷的项目，我认为在 micro 中你可以使用所有常见的 GUI 编辑器命令（`Ctrl-C` 复制，`Ctrl-V` 粘贴，`Ctrl-A` 全选）并且它们会按预期工作，这真是太棒了。我可能会尝试切换到 helix，只是重新训练我的 vim 肌肉记忆似乎太难了。此外，helix 还没有 GUI 或插件系统。

我个人**不会**使用 xterm、rxvt 或 Terminal.app 作为终端模拟器，因为我过去发现它们缺少一些核心功能（例如 Terminal.app 不支持 24 位色），这让我更难使用终端。

不过，我不想假装获得"现代化"终端体验比实际更容易——我认为有两个问题让它变得困难。让我们来谈谈它们！

### [获得"现代化"体验的第一个问题：shell][8]

bash 和 zsh 是迄今为止最流行的两种 shell，它们都没有提供我开箱即用就能满意的默认体验，例如：

*   你需要自定义你的提示符
*   它们默认不提供 git 补全，你必须自己设置
*   默认情况下，bash 只存储 500 行历史记录，而（至少在 Mac OS 上）zsh 只配置为存储 2000 行，这仍然不多
*   我发现 bash 的 tab 补全非常令人沮丧，如果有多个匹配项，你无法通过 tab 键浏览它们

尽管[我喜欢 fish][9]，但它不是 POSIX 兼容的事实确实让很多人难以切换。

当然，学习如何在 bash 或其他 shell 中自定义提示符是完全可能的，而且它甚至不需要那么复杂（在 bash 中，我可能会从类似 `export PS1='[\u@\h \W$(__git_ps1 " (%s)")]\$ '` 的东西开始，或者使用 [starship][10]）。但每一个"不复杂"的事情确实会累积起来，如果你需要在多个系统上保持配置同步，这尤其困难。

获得"现代化" shell 体验的一个非常流行的解决方案是 [oh-my-zsh][5]。它看起来是一个很棒的项目，我知道很多人非常高兴地使用它，但过去我在使用类似的配置系统时遇到了困难——看起来现在基本的 oh-my-zsh 增加了大约 3000 行配置，而且我经常发现，拥有一个额外的配置系统会让调试出问题时变得更加困难。我个人倾向于使用该系统添加大量额外的插件，使我的系统变慢，对它的缓慢感到沮丧，然后完全删除它并从头开始编写一个新的配置。

### [获得"现代化"体验的第二个问题：文本编辑器][11]

在我最近进行的终端调查中，最受欢迎的终端文本编辑器是 `vim`、`emacs` 和 `nano`。

我认为终端文本编辑器的主要选择是：

*   使用 vim 或 emacs 并按照你的喜好进行配置，如果你投入精力，你可能会拥有任何你想要的功能
*   使用 nano 并接受你将拥有一个相当有限的体验（例如，我认为你无法在 nano 中用鼠标选择文本然后"剪切"它）
*   使用 `micro` 或 `helix`，它们似乎提供了相当好的开箱即用体验，但可能会偶尔遇到使用不太主流的文本编辑器的问题
*   尽量避免使用终端文本编辑器，也许使用 VSCode，使用 VSCode 的终端来满足你所有的终端需求，并且基本上不在终端中编辑文件。或者我知道很多人使用 `code` 作为他们在终端中的 `EDITOR`。

### [问题 3：个别应用程序][12]

最后一个问题是，有时我使用的个别程序有点烦人。例如，在我的 Mac OS 机器上，`/usr/bin/sqlite3` 不支持 `Ctrl+左箭头` 快捷键。为了解决这个问题以获得在 SQLite 中的合理终端体验，我不得不：

*   意识到为什么会发生这种情况（Mac OS 不会提供 GNU 工具，而"Ctrl-左箭头"支持来自 GNU readline）
*   找到一个解决方法（从 homebrew 安装 sqlite，它确实支持 readline）
*   调整我的环境（将 Homebrew 的 sqlite3 放入我的 PATH 中）

我发现调试像这样的应用程序特定问题真的不容易，而且通常感觉"不值得"——通常我最终会处理各种小不便，因为我不想花几个小时去调查它们。我之所以能够解决这个问题，唯一的原因是我最近花了大量时间思考终端。

使用终端程序获得"现代化"体验的一个重要部分就是使用更新的终端程序，例如，我懒得学习一个键盘快捷键来在 `top` 中排序列，但在 `htop` 中，我只需用鼠标点击列标题即可排序。所以我使用 htop 代替！但发现新的更"现代化"的命令行工具并不容易（尽管我在这里做了一个[列表][13]），找到我实际喜欢使用的工具需要时间，而且如果你 SSH 到另一台机器，它们并不总是存在。

### [一切都会影响其他一切][14]

我发现配置终端使一切"美好"的一个棘手之处是，改变我工作流程中看似很小的一件事可能会真正影响其他一切。例如，现在我不使用 tmux。但如果我需要再次使用 tmux（例如，因为我在 SSH 到另一台机器上做了很多工作），我需要考虑几件事，比如：

*   如果我想要 tmux 的复制通过 SSH 与我的系统剪贴板同步，我需要确保我的终端模拟器有 [OSC 52 支持][15]
*   如果我想使用 iTerm 的 tmux 集成（它将 tmux 标签变成 iTerm 标签），我需要改变我配置颜色的方式——现在我用一个 [shell 脚本][16] 在 shell 启动时设置它们，但这意味着在恢复 tmux 会话时颜色会丢失。

还有可能更多我没有想到的事情。"使用 tmux 意味着我必须改变我管理颜色的方式"听起来不太可能，但这确实发生在我身上，我决定"好吧，我现在不想改变我管理颜色的方式，所以我猜我不会使用那个功能！"

记住我依赖哪些功能也很难——例如，也许我当前的终端确实有 OSC 52 支持，而且因为从 tmux 通过 SSH 复制一直都能正常工作，我甚至没有意识到这是我需要的东西，然后当我切换终端时，它神秘地停止工作。

### [慢慢改变][17]

虽然我认为我的设置并不是那么复杂，但我花了 20 年才达到这一点！因为终端配置的变化很可能会产生意外和难以理解的后果，我发现如果我一次性改变大量终端配置，会使我更难理解如果出现问题是什么出错了，这可能会让人非常迷失方向。

所以我通常更喜欢做相当小的改变，并接受改变可能需要我很长时间才能适应。例如，一两年前我从使用 `ls` 切换到 [eza][18]，虽然我喜欢它（因为 `eza -l` 默认打印人类可读的文件大小），但我仍然不太确定。但有时做一个大的改变也是值得的，比如 10 年前我从 bash 切换到 fish，我很高兴我这样做了。

### [获得"现代化"终端并不那么容易][19]

尝试解释配置终端有多"容易"，实际上只是让我认为这有点难，而且我有时仍然会感到困惑。

我发现在终端中从来没有一种完美的配置方式，可以与其他所有东西兼容。我只需要尝试各种东西，找出一种适合我的本地稳定状态，并接受如果我开始使用一个新工具，它可能会扰乱系统，我可能需要重新思考一些事情。

[1]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#what-is-a-modern-terminal-experience
[2]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#how-i-achieve-a-modern-experience
[3]: https://github.com/chriskempson/base16
[4]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#some-out-of-the-box-options-for-a-modern-experience
[5]: https://ohmyz.sh/
[6]: https://micro-editor.github.io/
[7]: https://helix-editor.com/
[8]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#issue-1-with-getting-to-a-modern-experience-the-shell
[9]: https://jvns.ca/blog/2024/09/12/reasons-i--still--love-fish/
[10]: https://starship.rs/
[11]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#issue-2-with-getting-to-a-modern-experience-the-text-editor
[12]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#issue-3-individual-applications
[13]: https://jvns.ca/blog/2022/04/12/a-list-of-new-ish--command-line-tools/
[14]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#everything-affects-everything-else
[15]: https://old.reddit.com/r/vim/comments/k1ydpn/a_guide_on_how_to_copy_text_from_anywhere/
[16]: https://github.com/chriskempson/base16-shell/blob/588691ba71b47e75793ed9edfcfaa058326a6f41/scripts/base16-solarized-light.sh
[17]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#change-things-slowly
[18]: https://github.com/eza-community/eza
[19]: http://jvns.ca/blog/2025/01/11/getting-a-modern-terminal-setup/#getting-a-modern-terminal-is-not-that-easy
