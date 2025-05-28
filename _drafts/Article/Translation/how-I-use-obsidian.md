---
title: 我如何使用 Obsidian
date: 2025-05-24T15:54:12.001Z
authorURL: "https://stephango.com/about"
originalURL: https://stephango.com/vault
translator: "Yiwei"
reviewer: ""
---

我使用 [Obsidian][1] 来思考、记笔记、写文章，并发布到这个网站。这是我自下而上的，用于记笔记和整理我感兴趣的事物的方法（这一句翻译还需要推敲）。它拥抱混乱和懒惰，从而创造出一种自发的结构。

<!-- more -->

在 Obsidian 中，“ vault ”只是一个文件夹。这一点很重要，因为它符合我的[文件优先于应用][2]理念。如果你想创建能够长久保存的数字作品，它们必须是你能控制的文件，采用易于检索和阅读的格式。Obsidian 给了你这种自由。

以下内容绝非教条，只是展示如何使用 Obsidian 的一个例子。取你所需即可。

## Vault 模板

1. [下载我的 vault 模板][3]或从 [GitHub 仓库][4]克隆。
2. 将 `.zip` 文件解压到你选择的文件夹中。
3. 在 Obsidian 中将该文件夹作为 vault 打开。

## 主题和相关工具

- 我的主题 [Minimal][5] 。
- [Obsidian Web Clipper][6] 用于保存网页上的文章和页面，查看我为特定网站准备的[剪藏模板][7]。
- [Obsidian Sync][8] 用于在桌面电脑、手机和平板之间同步笔记。
- [Obsidian Bases][9] 自2025年5月起已取代了我对 Dataview 的使用。
- [Leaflet][10] 插件用于我某些模板中的地图功能。

## 个人规则

我在个人 vault 中遵循的规则：

- 避免将内容分散到多个 vault 中。
- 避免使用文件夹进行组织。
- 避免使用非标准 Markdown 。
- 总是将分类和标签使用复数形式。
- 大量使用内部链接。
- 所有地方都使用 `YYYY-MM-DD` 日期格式。
- 使用 7 分制进行评分。
- 每周保持[单一待办事项列表][11]。

拥有[一致的风格][12]可以将未来数百个决定简化为一个，让我更加专注。例如，我总是使用复数形式的标签，这样我就不必考虑如何命名新标签。选择你觉得舒适的规则并写下来。制定你自己的风格指南。你随时可以在以后更改这些规则。

## 文件夹和组织

我使用的文件夹很少。我避免使用文件夹，因为我的许多条目属于多个思想领域。我的系统注重速度和便捷。我不想承担考虑将内容放在哪里的额外负担。

我不使用嵌套子文件夹。我很少使用文件浏览器进行导航。我主要通过快速切换器、反向链接或笔记内的链接进行导航。

我的笔记主要使用 `category` 属性进行组织。分类是列出相关笔记的概览笔记。

**我的大部分笔记都在 vault 的根目录中**，而不是在文件夹里。这里是我写关于个人世界的地方：日记条目、文章、[常青][13]笔记和其他个人笔记。如果一个笔记在根目录中，我知道它是我写的，或者与我直接相关。

我使用的两个参考文件夹：

- References（参考资料），我在这里记录外部世界的事物。书籍、电影、地点、人物、播客等。始终使用标题命名，例如 `Book title.md` 或 `Movie title.md` 。
- Clippings（剪藏），我在这里保存其他人写的内容，主要是随笔和文章。

我创建了三个管理文件夹，这样它们的内容就不会显示在文件导航中：

- **Attachments（附件）** 用于存放图片、音频、视频、PDF等。
- **Daily（日记）** 用于我的每日笔记，全部命名为YYYY-MM-DD.md 。我不在日记中写任何内容，它们的存在仅仅是为了从其他条目链接到它们。
- **Templates（模板）** 用于存放模板。

在我的知识库的可下载版本中有两个文件夹是为了清晰起见而存在的。在我个人的知识库中，这些笔记会在根目录下，而不是在文件夹中。

- **Categories（分类）** 包含按类别（如书籍、电影、播客等）组织的笔记顶层概览。
- **Notes（笔记）** 包含示例笔记。

## 链接

我在笔记中大量使用内部链接。我尽量总是链接第一次提到的事物。我的日记条目通常是记录近期事件的意识流，寻找事物之间的联系。通常链接是_未解析的_，意味着该链接的笔记尚未创建。未解析的链接很重要，因为它们是未来事物之间连接的面包屑。

我知识库**根目录**中的日记条目可能看起来像这样：

```
我和 [[Aisha]] 一起去 [[Vidiots]] 看了电影 [[Perfect Days]] ，然后在 [[Little Ongpin]] 吃了菲律宾美食。我很喜欢 Perfect Days 中的这句话：[[Next time is next time, now is now]]。它让我想起了那篇随笔...
```

电影、电影院和餐厅各自链接到我的 **References** 文件夹中的条目。在这些参考笔记中，我记录属性、我的评分以及对该事物的想法。我使用 [Web Clipper][14] 帮助从 IMDB 等数据库填充属性。这句引言对我很有意义，所以它成为了我根文件夹中的[常青笔记][15]。我提到的随笔在我的 **Clippings** 文件夹中，因为它不是我自己写的。

这种大量链接的风格随着时间的推移变得更加有用，因为我可以追踪想法是如何产生的，以及这些想法创造的分支路径。

## 分形日记和随机回顾

分形日记和随机化是我驯服知识库的方法，它可以让知识库成长为一片荒野。

在一天中，我使用 Obsidian 的 _unique note_ 快捷键来记录想到的个人想法。这个快捷键会自动创建一个前缀为 `YYYY-MM-DD HHmm`的笔记，我可能会添加一个描述这个想法的标题。

每隔几天，我会回顾这些日记片段并汇编重要的想法。然后我每月回顾这些回顾，并每年回顾月度回顾（使用[这个模板][16]）。结果是我生活的分形网络，我可以以不同的细节程度放大和缩小。我可以追溯个别想法的来源，以及它们如何汇集成更大的主题。

每隔几个月，我会留出时间进行"随机回顾"。我使用 _random note_ 快捷键在我的知识库中快速随机游走。我经常使用浅层深度的本地图谱来查看相关笔记。这有助于我重新审视旧想法，创建缺失的链接，并在过去的想法中找到灵感。这也是进行维护的机会，比如根据我个人风格指南中的新规则修复格式。

有人问我是否可以用语言模型自动化这个过程，但我不想这样做。我享受这个过程。进行这种维护帮助我理解自己的模式。[不要委托理解][17]。

## 属性和模板

我创建的几乎每个笔记都是从[模板][18]开始的。我大量使用模板，因为它们允许我懒惰地添加信息，这些信息将帮助我以后找到笔记。我为每个类别都有一个带有顶部[属性][19]的模板，用于捕获以下数据：

- **日期** — 创建、开始、结束、发布
- **人物** — 作者、导演、艺术家、演员、主持人、嘉宾
- **主题** — 按流派、类型、话题、相关笔记分组
- **位置** — 社区、城市、坐标
- **评分** — 下面会详细介绍

我遵循的几条属性规则：

- 属性名称和值应该尽量在各个类别中可重复使用。这允许我跨类别查找内容，例如，`genre` 在所有媒体类型中共享，这意味着我可以在一个地方查看 _科幻_ 书籍、电影和节目的档案。
- 模板应该尽量可组合，例如，_Person_ 和 _Author_ 是两个不同的模板，可以添加到同一个笔记中。
- 简短的属性名称更容易输入，例如使用 `start` 而不是 `start-date` 。
- 如果将来可能包含多个链接或值，默认使用 `list` 类型属性而不是 `text` 。

[.obsidian/types.json][20] 文件列出了哪些属性分配给哪些类型（即 `date` 、`number` 、`text` 等）。

## 评分系统

任何带有 `rating` 的内容都使用 1 到 7 的整数：

- 7 — 完美，必须尝试，改变人生，值得你特意去寻找
- 6 — 优秀，值得重复体验
- 5 — 良好，不需要刻意寻找，但很享受
- 4 — 及格，应急时可用
- 3 — 糟糕，如果可以就不要选择
- 2 — 糟透了，主动避开，令人厌恶
- 1 — 邪恶，以不好的方式改变人生

为什么选择这个评分标准？我更喜欢用 7 分制而不是 4 分或 5 分，因为我需要在顶部有更多的层次，用于好的体验，而 10 分制又太过细致。

## 发布到网络

这个网站是直接从Obsidian中编写、编辑和发布的。为了做到这一点，我打破了上面列出的一条规则——我为我的网站创建了一个单独的知识库。我使用一个名为 [Jekyll][21] 的 _静态网站生成器_ 来自动将我的笔记编译成网站，并将它们从 Markdown 转换为 HTML 。

我的发布流程使用起来很简单，但设置起来有点技术性。这是因为我喜欢完全控制我网站布局的各个方面。如果你不需要完全控制，你可以考虑使用 [Obsidian Publish][22] ，它更加用户友好，这也是我用于我的 [Minimal 文档网站][23]的工具。

对于这个网站，我使用 [Obsidian Git][24] 插件将笔记从 Obsidian 推送到 GitHub 仓库。然后，这些笔记会使用 [Jekyll][25] 和我的网络服务器 [Netlify][26] 自动编译。我还使用我的 [Permalink Opener][27] 插件在浏览器中快速打开笔记，这样我可以比较草稿和线上版本。

配色方案是 [Flexoki][28] ，这是我为这个网站创建的。我的 Jekyll 模板不是公开的，但你可以从 Maxime Vaillancourt 的[这个模板][29]获得类似的效果。还有许多 Jekyll 的替代品可以用来编译你的网站，比如 [Quartz][30] 、[Astro][31] 、[Eleventy][32] 和 [Hugo][33] 。

## 相关文章

- [文件优于应用][34]
- [简明解释加速进步][35]
- [常青笔记将想法转变为可操作的对象][36]
- [每年问自己的 40 个问题][37]
- [每十年问自己的 40 个问题][38]
- [我如何管理待办事项][39]

[1]: https://stephango.com/obsidian
[2]: https://stephango.com/file-over-app
[3]: https://github.com/kepano/kepano-obsidian/archive/refs/heads/main.zip
[4]: https://github.com/kepano/kepano-obsidian
[5]: https://stephango.com/minimal
[6]: https://stephango.com/obsidian-web-clipper
[7]: https://github.com/kepano/clipper-templates
[8]: https://obsidian.md/sync
[9]: https://help.obsidian.md/bases
[10]: https://github.com/javalent/obsidian-leaflet
[11]: https://stephango.com/todos
[12]: https://stephango.com/style
[13]: https://stephango.com/evergreen-notes
[14]: https://stephango.com/obsidian-web-clipper
[15]: https://stephango.com/evergreen-notes
[16]: https://stephango.com/40-questions
[17]: https://stephango.com/understand
[18]: https://github.com/kepano/kepano-obsidian/tree/main/Templates
[19]: https://help.obsidian.md/properties
[20]: https://github.com/kepano/kepano-obsidian/blob/main/.obsidian/types.json
[21]: https://jekyllrb.com/
[22]: https://obsidian.md/publish
[23]: https://minimal.guide/publish/download
[24]: https://obsidian.md/plugins?id=obsidian-git
[25]: https://jekyllrb.com/
[26]: https://www.netlify.com/
[27]: https://stephango.com/permalink-opener
[28]: https://stephango.com/flexoki
[29]: https://github.com/maximevaillancourt/digital-garden-jekyll-template
[30]: https://quartz.jzhao.xyz/
[31]: https://astro.build/
[32]: https://www.11ty.dev/
[33]: https://gohugo.io/
[34]: https://stephango.com/file-over-app
[35]: https://stephango.com/concise
[36]: https://stephango.com/evergreen-notes
[37]: https://stephango.com/40-questions
[38]: https://stephango.com/40-questions-decade
[39]: https://stephango.com/todos