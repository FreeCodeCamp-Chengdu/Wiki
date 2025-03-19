---
title: 高速成长工程师
authorURL: ""
originalURL: https://careercutler.substack.com/p/the-top-7-software-engineering-workflow
translator: ""
reviewer: ""
---

你的时间很重要。

每天提高 1 小时的效率可以为你每年节省 1 个月。

我来证明给你看：

_每天 1 小时 x 每周 5 天 x 52 周 = 每年节省 260 小时_

_260 小时 / 每天 8 小时 = 每年 32.5 个工作日。_

这意味着你可以提前 1 个月获得晋升、放松或做任何你想做的事情。

优化我每天做的事情使我在过去几年作为软件工程师的工作中平均每天可以提交 1-2 个拉取请求。**这不是最好的衡量标准（也不是应该追求的目标），但它提供了一些参考框架。**

今天，我将分享让这成为可能的最重要的工作流程技巧。

<!-- more -->

---

## [系统设计挑战（赞助）][12]

![][1]

通过高级工程师、经理和 CTO [Ricardo Morales][13] 的真实世界挑战来学习系统设计。我最近喜欢的一个是[设计 Google Sheets][14]。

[订阅系统设计挑战][15]

感谢 Ricardo 和他的通讯赞助，让 High Growth Engineer 对你们所有人保持免费 🙏

---

## 💡 重要理念

优化你**每天**多次执行的任务。

例如，以文章开头的等式为例，如果你每个工作日通过自动化或更好的流程节省 2 分钟，每年就能收回 1 个工作日。

你可以使用下面的图表来决定是否值得优化某件事，但我不这么做。

我优化重复的日常任务，并希望得到最好的结果 🤞


![][2]

xkcd 时间优化图表。注意：它对"工作日"来说并不完全准确，因为它是基于 24 小时制的。[来源][16]

在每个部分，我将介绍软件工程师工作流程中最频繁的部分。

每个部分都包括我的做法，但你可能会发现你已经在做的事情比我更有效率！这很好。请随时告诉我！我一直在寻求改进。

## (1) 💻 Git / 终端工作流

这里有 3 个巨大的时间节省方法：

1. 自动完成过去的命令
2. 命令别名
3. 轻松管理 Git 文件

在高层次上，我建议查看[这篇终端改造文章的目录][17]。不过，我会介绍 3 个主要的时间节省方法。

### 自动完成过去的命令

我使用以下终端设置：

- [iTerm2][18] - 终端
- [Oh-my-zsh][19] 和 Zsh (bash 的替代品) - [设置指南在这里][19]
- [Starship][20] - 自定义终端提示符（这个不是必需的，但值得了解）

一旦你设置好 oh-my-zsh，你可以在`.zshrc`文件中添加这些"插件"

```zsh
# in ~/.zshrc
plugins=(
  git
  zsh-autosuggestions <--- 我们现在要讨论这个
  zsh-syntax-highlighting
)
```

使用 zsh-autosuggestions，你输入的每个命令都会开始自动完成。

看下面的演示。

![][3]

我只需要按键盘上的右箭头键，它就会填充完整。

### 别名

上面，我们看到我在`.zshrc`中添加了这个

```plain
# in ~/.zshrc
plugins=(
  git <--- 我们现在要讨论这个
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

git 插件为 git 命令添加了别名。我使用的一些是：

- ga => git add
- gc => git commit
- gd => git diff
- gs => git status
- gps => git push
- gpl => git pull

这每天至少为我节省一两分钟，一年下来就很可观了。

我还在`.zshrc`中添加自定义别名

这里有两个例子

```plain
# in ~/.zshrc
alias addalias='code ~/.zshrc' <-- 打开zshrc文件进行编辑
alias reload='source ~/.zshrc' <-- 重新加载zshrc文件以应用更改
```

### Git 文件的编号快捷方式

添加[SCM Breeze][21]允许你轻松处理 Git 中的文件。

这些编号快捷方式是 SCM Breeze 为你提供的：

![][4]

现在你可以像`git add 2-3`或`git reset 1`这样引用你的文件

看下面的例子：

![][5]

## (2) 🖥️ 编码

以下是在代码编辑器中优化时间的最重要领域

1. **向上或向下追踪代码堆栈** - 假设你正在查看一个方法调用。你想看看该方法内部的代码。**不要全局使用 cmd+f 搜索该函数名称来查找定义！**

   1. **这样做：** 为追踪变量、方法、类等设置键盘快捷键。
   2. VSCode 快捷键：`F12`。
   3. Jetbrains IDE 快捷键：`cmd+b`

2. **在位置之间导航** - 通常你在 2-3 个文件之间工作，在每个文件的相同位置工作，并点击顶部的导航来查找你正在处理的 15 个文件中的哪一个。或者你在一个大文件中来回滚动。但有更好的方法。

   1. **这样做：** 为在你的"位置历史"中向后和向前移动设置键盘快捷键。它可以让你在文件之间来回移动，跨文件，所以你可以轻松地在文件和位置之间切换。
   2. VSCode 快捷键：`ctrl + -` 和 `ctrl + shift + -`
   3. Jetbrains IDE 快捷键：`cmd + option + ⬅️` 和 `cmd + option + ➡️`

3. **输入** - 使用[Github Copilot][22]自从几个月前开始使用以来已经为我节省了数天的时间。几乎你写的每一行代码都可以为你自动完成，包括测试。它的智能程度令人震惊 🤯。

## (3) 📓 保存学习内容

我把所有学到的和想要保存以后用的东西都保存在 Notion 中。

[Tiago Forte][23]将这个概念普及为"构建第二大脑"。

一个关键的要点是把你的学习内容存储在你会**使用**它们的地方，而不是你学到它们的地方。

这是我的 Notion 笔记文件夹的两个截图：

![][6]

每个主题都是我从许多不同文章、视频、书籍等收集的学习内容。这有助于我将来更容易找到它们。

将来，我计划更公开地分享这些内容，但我的许多文章都是我笔记的精选版本。

## (4) ✅ 放下想法和任务

你的大脑应该用于创造力、解决问题和产生想法—**而不是存储它们。**

我使用[Todoist][24]和 Slack 的"稍后保存"提醒的组合来存储任何想法或任务，这样我就可以立即把它从我的大脑中清除。

在我开始这样做之前，我的大脑一直在"颠簸"。我会在头脑中在任务之间切换，试图记住它们，实际完成它需要花费 10 倍的时间。

这是我组织 Todoist 的方式，但你应该做最适合你的方式。

![][7]

**注意：** 我喜欢的一个功能是你可以将网站添加为任务，所以如果你遇到想稍后查看的视频或文章，你可以一键将其添加为待办事项。

彩蛋🥚如果你能注意到我的待办事项中有为我的猫安装攀爬架的项目😄

**这里的关键要点是使用什么工具并不重要。确保你可以让你的大脑专注于解决问题，而不是记住所有事情。**

## (5) 👀 通过视觉进行沟通

以下是一些你可以使用视觉来更好地沟通的日常用例：

- 记录拉取请求
- 发送 Slack 消息向 PM 解释你想做的更改
- 获得对你构建的新流程的截图的批准或签字

[CleanShot][25] ([非推广链接选项][26])在这里为我节省了大量时间。

它替代了 Mac 上的截图工具，并为你提供...

- 一种快速简便的方法来对刚刚拍摄的截图应用大量视觉编辑，并将其放在任何地方
- 一个多合一的 gif + 屏幕录制工具。这就是我如何创建本文中的其他视觉效果。
- 更多功能（如滚动捕获、模糊图像的部分等）。如果你感兴趣，我建议查看[他们的演示视频][27]。

下面是一个稍微牵强的例子，但用截图注释我的拉取请求是这个工具让我经常做的事情，这会导致更快的批准：

![][8]

另一个常见的情况是为演示制作快速屏幕录制，或向我的设计师或 PM 发送 Slack 消息，获得对功能行为的签字。

## (6) 🔑 登录账号

密码管理器因两个原因而改变游戏规则：

1. 便利性 - 再也不用输入电子邮件或密码。它会为你做这件事。
2. 安全性 - 它会为你在每个网站上创建复杂和唯一的密码。

这是使用[1Password][28]登录网站时的样子：

![][9]

## (7) 🪟 窗口管理

作为软件工程师，我们经常管理我们的浏览器窗口、终端、编辑器和消息应用。

有一段时间，我花了很多时间手动调整窗口大小，直到我开始使用窗口管理工具。

我使用[Rectangle][29]。它是免费的，专业版终身访问只需 10 美元。

它让我可以快速将一个窗口放在左侧，一个窗口放在右侧，全屏显示另一个窗口等。它为我节省了大量时间。

[

![][10]

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F604f5344-87ae-41fd-86ac-0486512a1b19_800x452.gif)

## 💭 结束语

将所有这些整合起来对我来说是一个 6 年的过程。我需要慢慢地引入新工具，否则我会感到不知所措。

**这篇文章的目的不是让任何人觉得如果他们不使用这些工具就是做错了。**

它只是分享对我有效的东西，如果你感兴趣的话可以尝试。

你不需要使用和我一样的工具。使用对你有效的工具。

欢迎在评论中分享你使用的最有帮助的工具 🙏

## 📣 本周推荐

- [情商课程：如何让某人真正感到被倾听和被看见][30] - 作者：Irina Stanescu。关于如何提高情商的扎实课程和例子。
- [我如何说服我们没有错][31] - 作者：Raviraj Achar。关于如何有效解决团队之间紧张分歧的精彩故事和启示。
- [如何在顶级科技公司获得更多面试、offer 和薪酬][32] - 即将举行的 LinkedIn 直播活动，由我和 Alan Stein（在多家 FAANG 公司担任 Director+级别）主持。点击链接参加。将在本周二，10 月 10 日太平洋时间下午 5 点举行。

---

_像往常一样，感谢阅读，感谢订阅人数增长到 16k+。_

- Jordan

_P.S. 如果你感兴趣，我正在接受以下内容：_

1. _新的辅导客户：查看[Mentorcruise 的费率][33]_
2. _通讯赞助：请回复此邮件联系我。_

![][11]

[1]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6b3f06a3-2d91-4470-94a3-317a4ca88350_500x500.webp
[2]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9c3189f5-9da9-4f2e-87d1-00ca60e5c4d2_571x464.png
[3]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F06546601-9068-4b82-9640-cdf6918b5883_800x310.gif
[4]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0d235595-37c2-4dcc-bdcb-4a106848f61c_764x340.png
[5]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F94733d79-29a2-46e5-b975-5eb5d7b01620_800x500.gif
[6]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f934cb1-e3fc-4473-abb2-0eb472ed85cb_1182x960.png
[7]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbcac5a15-b230-4099-a888-95e8d2192083_2328x1220.png
[8]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1239c71c-7928-49a2-9f85-33ebfa70a683_1614x968.png
[9]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F33e47349-d67e-42d5-a073-06a5a273ccaa_1044x828.png
[10]: https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F604f5344-87ae-41fd-86ac-0486512a1b19_800x452.gif
[11]: https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe2aed624-b6bc-45a9-b8f5-b28b519871a1_1920x1280.jpeg
[12]: https://www.systemsdesignchallenges.com
[13]: https://www.linkedin.com/in/ricardo-morales-1820/
[14]: https://www.systemsdesignchallenges.com/p/systems-design-challenge-1-of-2
[15]: https://www.systemsdesignchallenges.com/
[16]: https://xkcd.com/1205/
[17]: https://towardsdatascience.com/the-ultimate-guide-to-your-terminal-makeover-e11f9b87ac99
[18]: https://iterm2.com/
[19]: https://github.com/ohmyzsh/ohmyzsh
[20]: https://starship.rs/
[21]: https://github.com/scmbreeze/scm_breeze
[22]: https://github.com/features/copilot
[23]: https://www.youtube.com/@TiagoForte
[24]: https://todoist.com/
[25]: https://cleanshot.sjv.io/oqVmZg
[26]: https://cleanshot.com/
[27]: https://www.youtube.com/watch?v=FZbICrBKWIU
[28]: https://1password.com/personal
[29]: https://rectangleapp.com/
[30]: https://www.thecaringtechie.com/p/lessons-in-emotional-intelligence
[31]: https://ravirajachar.substack.com/p/how-i-convinced-we-werent-wrong
[32]: https://www.linkedin.com/events/7116067854558355456
[33]: https://mentorcruise.com/mentor/jordancutler/
