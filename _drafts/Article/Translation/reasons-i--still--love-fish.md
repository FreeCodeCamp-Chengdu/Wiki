---
title: 我（仍然）喜欢fish的原因
date: 2024-09-21T02:20:19.154Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/09/12/reasons-i--still--love-fish/
translator: ""
reviewer: ""
---

我在[2017年的这篇博文][1]中写过我有多喜欢[fish][2]，现在经过7年的每日使用，我发现了更多喜欢它的理由。所以我想写一篇新文章，既包含我之前喜欢它的原因，也包含一些新的理由。

今天之所以想到这个话题，是因为我在试图弄清楚为什么我的终端不再因为cat一个二进制文件而崩溃了，答案是"fish修复了终端！"，我觉得这真的很贴心。

### 1. 无需配置

在使用fish的10年里，我从未发现任何需要配置的地方。它就是按照我想要的方式工作。我的fish配置文件只包含：

* 环境变量
* 别名（`alias ls eza`，`alias vim nvim`等）
* 偶尔需要的`direnv hook fish | source`来集成direnv这样的工具
* 一个用来设置我的[终端颜色][3]的脚本

我听说如果你真的想配置什么的话，在fish中配置东西也非常容易。

### 2. 来自shell历史记录的自动建议

我最喜欢fish的一点是，当我输入时，它会自动建议（以浅灰色显示）我最近运行过的匹配命令。我可以按右箭头键接受补全，或继续输入来忽略它。

下面是它的样子。在这个例子中，我只输入了"v"键，它就猜到我想再次运行之前的vim命令。

![Image 7](https://jvns.ca/images/fish-2024.png)

### 2.5 "智能"shell自动建议

我最喜欢的一个微妙的自动补全功能是fish如何处理包含路径的命令的自动补全。例如，如果我运行：

```bash
$ ls blah.txt
```

这个命令只会在包含`blah.txt`的目录中被自动补全 - 它不会在其他目录中出现。（这里是[关于它如何工作的简短说明][4]）

举个例子，如果在这个目录中我输入`bash scripts/`，它只会建议包含我博客scripts文件夹中实际存在的文件的历史命令，而不是我在其他文件夹中运行过的几十个不相关的`scripts/`命令。

直到上周我才真正理解这是如何工作的，它就感觉像是fish神奇地能够建议正确的命令。它仍然感觉有点像魔法，我很喜欢。

### 3. 粘贴多行命令

如果我复制粘贴多行命令，bash会全部运行它们，像这样：

```bash
[bork@grapefruit linux-playground (main)]$ echo hi
hi
[bork@grapefruit linux-playground (main)]$ touch blah
[bork@grapefruit linux-playground (main)]$ echo hi
hi
```

这有点令人担忧 - 如果我实际上并不想运行所有这些命令怎么办？

Fish会将它们全部粘贴到一个提示符下，这样我可以在真正想运行它们时按Enter。这样就不那么可怕了。

```bash
bork@grapefruit ~/work/> echo hi

                         touch blah
                         echo hi
```

### 4. 友好的tab补全

如果我运行`ls`并按tab键，它会以漂亮的网格显示所有文件名。我可以使用Tab、Shift+Tab或箭头键来导航网格。

另外，我可以从文件名的**中间**开始tab补全 - 如果文件名以一个奇怪的字符开头（或者如果它不是很独特），我可以输入中间的一些字符并按tab。

下面是tab补全的样子：

```bash
bork@grapefruit ~/work/> ls 
api/  blah.py     fly.toml   README.md
blah  Dockerfile  frontend/  test_websocket.sh
```

说实话，除了文件名之外我不太使用其他补全，所以我不能谈论那些，但我发现tab补全文件名的体验非常好。

### 5. 漂亮的默认提示符（包括git集成）

Fish的默认提示符包含了我想要的一切：

* 用户名
* 主机名
* 当前文件夹
* git集成
* 上一个命令的退出状态（如果上一个命令失败）

这是一个截图，展示了默认提示符的几种不同变化，包括如果上一个命令被中断（`SIGINT`）或失败的情况。

![Image 8][11]

### 6. 良好的历史记录默认设置

在bash中，默认的最大历史记录大小是500，这可能是因为以前的计算机速度慢且磁盘空间有限。此外，默认情况下，命令只有在你结束会话后才会被添加到历史记录中。所以如果你的计算机崩溃了，你会丢失一些历史记录。

在fish中：

1. 默认历史记录大小是256,000条命令。我看不出有什么理由需要更多。
2. 如果你打开一个新标签，所有你曾经运行过的命令（包括在打开的会话中的命令）都会立即可用
3. 在现有会话中，历史记录搜索只会包含当前会话中的命令，以及你启动shell时历史记录中的所有内容

我不确定我是否清楚地解释了fish的历史记录系统是如何工作的，但在实践中感觉真的很好。我的印象是，它的实现方式是命令会持续添加到历史记录文件中，但fish只在启动时加载一次历史记录文件。

我在这里要提到，如果你想在其他shell中拥有更高级的历史记录系统，可以考虑看看[atuin][5]或[fzf][6]。

### 7. 按上箭头键来搜索历史记录

我也喜欢fish的历史记录搜索界面：例如，如果我想编辑我的fish配置文件，我可以只输入：

```bash
$ config.fish
```

然后按上箭头键来找到最后一个包含`config.fish`的命令。这样就完成了：

```bash
$ vim ~/.config/fish/config.fish
```

我就完成了。这与在bash中使用`Ctrl+R`来搜索历史记录并没有太大区别，但我觉得我更喜欢fish的方式，也许是因为`Ctrl+R`有一些我觉得混乱的行为（例如你可能会意外地编辑你的历史记录，我不喜欢）。

### 8. 终端不会崩溃

我以前会遇到bash的问题，即我会不小心用`cat`命令输出一个二进制文件，然后终端就会崩溃。

每次fish显示提示符时，它都会尝试修复你的终端，以免你陷入这种奇怪的情况。我觉得[fish中有一些代码来防止终端崩溃][7]。

它做的一些事情包括：

* 启用`echo`以便你可以看到自己输入的字符
* 确保换行符正常工作，以免出现奇怪的阶梯效果
* 重置你的终端背景颜色等

我觉得很久以前我就不再遇到“我的终端崩溃了”的问题了，我甚至不知道这是因为fish的原因。我以为事情只是神奇地变好了，或者也许我不再犯那么多错误。但我觉得主要是fish在保护我免于犯错，我真的很感激。

### 9. Ctrl+S被禁用

与终端崩溃有关：fish禁用了Ctrl+S（这会冻结你的终端，然后你需要记得按Ctrl+Q来解冻它）。这是一个我从未想要的功能，我很高兴没有它。

显然你可以使用`stty -ixon`在其他shell中禁用`Ctrl+S`。

### 10. `fish_add_path`

我对这个功能有混合的感觉，但在fish中，你可以使用`fish_add_path /opt/whatever/bin`来全局、永久地、跨所有打开的shell会话添加一个路径到你的PATH中。这可能会让人困惑，因为你可能会忘记这些PATH条目是如何配置的，但总的来说我觉得我很喜欢这个功能。

### 11. 良好的语法高亮

默认情况下，不存在的命令会以红色高亮显示，像这样。

![Image 9][12]

### 12. 更容易的循环

我觉得fish的循环语法比bash的更容易输入。它看起来像这样：

```
for i in *.yaml
  echo $i
end
```

此外，它会在你的循环中添加缩进，这很好。

### 13. 更容易的多行编辑

与循环有关：你可以更容易地编辑多行命令（只需使用箭头键来导航多行命令！）。此外，当你使用上箭头键从历史记录中获取多行命令时，它会以你最初输入的方式显示整个命令，而不是像bash那样将其压缩到一行：

```bash
$ bash
$ for i in *.png
> do
> echo $i
> done
$ # 按上箭头键
$ for i in *.png; do echo $i; done ink
```

### 14. Ctrl+左箭头

这可能只是我，但我真的很喜欢fish的`Ctrl+左箭头`/`Ctrl+右箭头`快捷键，用于在输入命令时在单词之间移动。

我老实说，我有点困惑，不知道这个快捷键来自哪里（我在fish中找到的唯一记录的快捷键是`Alt+左箭头`/`Alt+右箭头`，它似乎做了同样的事情），但我相当确定这是一个fish快捷键。

关于使这个快捷键生效以及它来自哪里的一些注意事项：

* 有人说他们需要将终端模拟器从“Linux控制台”键绑定切换到“默认（XFree 4）”才能使其在fish中生效
* 在Mac OS中，`Ctrl+左箭头`默认会切换工作区，所以我需要禁用它。
* 此外，据说Ubuntu在`/etc/inputrc`中配置libreadline，使得`Ctrl+左箭头`/`Ctrl+右箭头`可以在bash中向前/向后移动一个单词，所以它会在Ubuntu和可能其他Linux发行版中的bash中生效。这里有一个[Stack Overflow问题讨论这个问题][10]。

### 一个缺点：不是所有东西都有fish集成

有时工具没有fish集成的说明。这很烦人，但：

* 我发现这在过去的10年里变得更好了，因为fish变得越来越流行。例如，Python的virtualenv已经有很长时间了fish集成。
* 如果我需要快速运行一个POSIX shell命令，我总是可以运行`bash`或`zsh`
* 我在过去的几年里变得更擅长将简单的命令从bash语法转换为fish语法，当我需要时。

我最大的日常烦恼可能是，我仍然不习惯fish设置环境变量的语法，我会混淆`set`和`export`。

### 关于POSIX兼容性

当我开始使用fish时，你不能做像`cmd1 && cmd2`这样的事情 - 它会抱怨“不，你需要运行`cmd1; and cmd2`”。 

看起来随着时间的推移，fish开始接受一些POSIX风格的语法，比如：

* `cmd1 && cmd2`
* `export a=b`来设置环境变量（虽然这似乎有一些限制，你不能做`export PATH=$PATH:/whatever`，所以我觉得最好还是学习`set`）

### 关于fish作为默认shell

将我的默认shell更改为fish总是有点烦人，我偶尔会陷入这种情况：

1. 我在某个地方安装fish，比如`/home/bork/.nix-stuff/bin/fish`
2. 我将新的fish位置添加到`/etc/shells`中作为允许的shell
3. 我使用`chsh`更改我的shell
4. 在某个时候，几个月或几年后，我会以不同的位置重新安装fish，并删除旧的
5. 哦不！我没有有效的shell！我再也无法打开新的终端标签了！

这从来不是一个主要问题，因为我总是有一个打开的终端，可以在那里修复问题并拯救自己，但这有点令人担忧。

如果你不想使用`chsh`来将你的shell更改为fish（这很合理，也许我不应该这样做），[Arch wiki页面][8]有几个很好的建议 - 要么配置你的终端模拟器来运行fish，要么在你的`.bashrc`中添加一个`exec fish`。

### 我从未真正学习过fish的脚本语言

除了偶尔在命令行上交互式地编写for循环之外，我从未真正学习过fish的脚本语言。我仍然在bash中进行所有的shell脚本编写。

我不认为我曾经编写过fish函数或`if`语句。

### fish似乎变得越来越流行

我在Mastodon上进行了一项非常不科学的投票，询问人们在[交互式会话中使用什么shell][9]。结果是（2600个回复中）：

* 46% bash
* 49% zsh
* 16% fish
* 5% 其他

我觉得fish的16%是相当令人印象深刻的，因为据我所知，没有系统将fish作为默认shell，而且我的感觉是大多数人倾向于坚持使用系统的默认shell。

这感觉像是fish项目的一个巨大成就，即使我的Mastodon粉丝可能比平均水平更有可能使用fish。

### 谁可能适合使用fish？

fish显然不是每个人的菜。我觉得我喜欢它是因为：

1. 我真的不喜欢配置我的shell（和我的开发环境），我希望它能“开箱即用”。
2. fish的默认设置对我来说感觉很好。
3. 我不花太多时间登录到其他服务器使用其他shell，所以没有太多的上下文切换。
4. 我喜欢它的功能，所以我愿意重新学习一些“基本”的shell操作，比如使用括号`(seq 1 10)`来运行命令，而不是反引号，或者使用`set`而不是`export`。

也许你也是一个可能喜欢fish的人！我希望更多适合使用fish的人能够找到它，因为我花了很多时间在终端上，它使我的时间更加愉快。

[1]: https://jvns.ca/blog/2017/04/23/the-fish-shell-is-awesome/
[2]: https://fishshell.com/
[3]: https://github.com/chriskempson/base16-shell/blob/588691ba71b47e75793ed9edfcfaa058326a6f41/scripts/base16-solarized-light.sh
[4]: https://github.com/fish-shell/fish-shell/issues/120#issuecomment-6376019
[5]: https://github.com/atuinsh/atuin
[6]: https://github.com/junegunn/fzf
[7]: https://github.com/fish-shell/fish-shell/blob/a979b6341d7fc4c466b3992f25da3209e0808aaa/src/reader.rs#L3601-L3623
[8]: https://wiki.archlinux.org/title/Fish
[9]: https://social.jvns.ca/@b0rk/112722850642874842
[10]: https://stackoverflow.com/questions/5029118/bash-ctrl-to-move-cursor-between-words-strings
[11]: https://jvns.ca/images/fish-prompt-2024.png
[12]: https://jvns.ca/images/fish-syntax-2024.png