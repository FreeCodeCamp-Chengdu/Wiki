---
title: 浏览器开发者工具中不应该再有秘密
date: 2021-11-01 15:21:00
authors:
  - luojiyin1987
original: https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/
categories:
  - Article
  - Translation
toc: true
---

> **更新**: 由于这篇文章[正在 Hackernews 上热传][1]，我在每个标题后的括号中为每个提示添加了支持环境的信息。当我说明 `Chromium 浏览器` 时，指的是所有使用 `Chromium` 内核并具有所有开发者工具的浏览器。这包括 `Chrome` 浏览器、`Microsoft Edge`、`Brave` 以及其他更多浏览器。在此提醒一下： `Microsoft Edge` 是 `Windows 10/11` 系统自带的浏览器，基于 `Chromium`，因此从平台角度来看与 `Chrome` 浏览器类似。它们在用户体验和核心服务方面有所不同。`Edge` 开发者工具与谷歌密切合作，将我们添加到产品中的工作带回 `Chromium` 核心。但是，我在这里谈到的一些东西是微软 `Edge` 的实验和独有功能，它可在 `Windows`、`Mac` 和 `Linux` 上使用。有些功能只能通过 [Edge DevTools for VS Code 扩展][2]在 `Visual Studio Code` 中使用。

这是我今年 9 月在 [CityJS][3] 上发表的演讲。我是 `Microsoft Edge` 开发者人员工具的首席产品经理，这些都是我在开发工具、记录工具和查看用户反馈时遇到的问题。

你可以在 Youtube 上观看[演讲视频][4]。

<!-- more -->

下面是我所写的所有内容：

## 1. Console 远不止是打印日志!

（所有浏览器的开发者工具都遵循该标准）

毫无疑问，除了元素工具，控制台是浏览器开发者工具中使用最多的部分。特别是，人们喜欢通过在代码中添加 `console.log()` 来调试，以了解发生了什么。虽然这样做存在一些问题，也有更好的调试脚本的方法，但既然人们都这么做了，那我们就来谈谈如何让这种体验变得更好。

第一个问题是产品上线后未删除的日志信息塞满了控制台。查找所需的信息变得令人望而生畏，而解决这个问题的最好办法就是了解 [控制台过滤选项][5] 。使用这些选项，你可以将控制台的报告过滤为你所关心的内容，并屏蔽掉大量杂音。

[![控制台工具中的过滤选项](https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_-1024x510.png)][6]

### 你想输出什日志?

使用 `console.log()` 的下一个问题是，我们似乎只记录了值，却忘了添加它们的来源。例如，当你使用下面的代码时，你会得到一个数字列表，但你不知道什么是什么。

```javascript
console.log(width);
console.log(height);
```

要解决这个问题，最简单的方法是用大括号把要记录的内容包起来。这样，控制台就会同时记录你想知道的内容的名称和值。

```javascript
console.log({ width });
console.log({ height });
```

[![在日志信息中使用大括号记录变量的名称和值](https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png)][7]

### 添加到你的控制台词汇

[![警告、信息和错误信息示例及其在控制台中的显示方式](https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png)][8]

除了 `console.log()` 之外，你还可以使用 [更多方法][9] 。例如，`console.warn()` 记录警告，`console.info()` 记录信息，`console.error()` 记录错误信息。这不仅会使控制台的显示略有不同，而且还会使信息具有不同的日志级别，这意味着更容易对其进行过滤。

### 控制台中的错误和断言

[![console 的 error 方法会显示错误，而 assert 是 if 语句的快捷方式，其中包含 console.log](https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png)][10]

在控制台中显示错误不同于抛出错误，但向维护或调试产品的人员显示问题的严重性仍然是个好主意。另一个有趣的方法是 `console.assert()`，它只在满足特定条件时记录信息。你经常会发现自己在编写 `if` 语句时，里面包含了一个 `console.log()`。使用`assert()`后，这条语句就变得多余了，在清理调试代码时就少了一件要担心的事。

### 追溯源头

[![使用 console.trace() 追溯调用来源的示例](https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png)][11]

你经常会发现自己添加了一个 `console.log('called')` 或类似内容来测试某个功能是否被触发。一旦有了这个功能，下一件事通常就是找出是什么调用了该方法。这就是`console.trace()`的作用，因为它不仅会告诉你某个方法被调用了，还会告诉你调用来自哪里。

### 将控制台信息分组

如果要记录的内容很多，可以使用 `console.group('name')` 和 `console.groupEnd('name')` 在控制台中以可折叠和可展开的方式封装信息。你甚至可以定义组默认是展开还是折叠。

[![在控制台中定义组的示例](https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png)][12]

### 在控制台中以表格形式显示和过滤大量信息

如果你想以日志形式显示大量信息，那么阅读这些信息可能会令人望而生畏。`console.table()` 方法会在控制台中以表格形式显示类似数组的数据，你可以通过给它一个数组来过滤你想显示的属性。

例如，你可以使用 `let elms = document.querySelectorAll(':is(h1,p,script')` 从文档中获取所有 H1、段落和脚本元素，并使用 `console.table(elms)` 以表格形式显示这些信息。由于不同的元素有大量的属性和属性，因此生成的表格非常难读。如果使用`console.table(elms,['nodeName', 'innerText', 'offsetHeight'])` 过滤出你感兴趣的内容，你就会得到一个只有这些属性及其值的表格。

![使用 console.table() 及其过滤选项的代码示例][13]

复制和粘贴这些信息时，表格结构会保持不变，可以轻松地将数据导入到 Excel 或 Word 中

### 像 jQuery 一样锋利 : $() and $$()

控制台自带了许多方便使用的方法，称为 [Console Utilities][14] 。其中两个非常有用的方法是 `$()` 和 `$$()`，它们分别是 `document.querySelector()` 和 `document.querySelectorAll()` 的替代方法。它们不仅会返回你所期望的 `nodeList`，还会将结果转换为数组，这意味着你可以直接在结果上使用 `map()` 和 `filter()`。以下代码将抓取当前文档中的所有链接，并返回一个数组，数组中的对象仅包含每个链接的 `href` 和 `innerText` 属性，即 `url` 和 `text` 属性。

```javascript
$$("a").map((a) => {
    return { url: a.href, text: a.innerText };
});
```

[![下面举例说明 $$ 函数如何返回一个 HTML 元素集合，您可以像过滤其他数组一样过滤这些元素](https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png)][15]

## 2. 你可以在没有源访问权限的情况下进行日志记录，实时表达式和日志点

(Chromium 浏览器)

添加 `console.log()` 的常规方法是将其放在代码中想要获取信息的地方。但你也可以深入了解你无法访问和更改的代码。[Live 表达式][16] 是在不更改代码的情况下记录信息的好方法。它们还能记录不断变化的值，而不会让控制台满屏日志，从而降低产品的运行速度。你可以从下面的截屏中看到它们的不同之处：

<video width="320" height="240" controls>
  <source src="https://christianheilmann.com/wp-content/uploads/2021/10/log-vs-live-expression-smaller.mp4" type="video/mp4">
</video>

日志点是一种特殊的断点。你可以右键单击开发者工具源代码工具中 JavaScript 的任意一行，然后设置一个日志点。你会被要求提供一个要记录的表达式，并在执行该行代码时在控制台中获得其值。这意味着从技术上讲，你可以在网络上的任何地方注入一个 `console.log()`。我早在八月份就[写过关于日志点的文章][17]，你可以在下面的截屏中看到演示：

<video width="320" height="240" controls>
  <source src="https://youtu.be/DRRezUZvZ6I" type="video/mp4">
</video>

## 3. 你可以在浏览器之外使用 VS code 调试器进行日志记录

(Chromium 浏览器 和 VS Code)

在 Visual Studio Code 中启动调试会话时，可以生成一个浏览器实例，调试控制台就会变成浏览器开发者工具中的控制台。我曾在七月份的博客中详细介绍过这一点，所以你可以[阅读如何做到这一点][18]。[官方文档][19] 中也有更多内容。

[![](https://christianheilmann.com/wp-content/uploads/2021/10/Slide24-842x1024.png)][20]

你还可以观看我展示功能的一分钟视频：

<video width="320" height="240" controls>

  <source src="https://youtu.be/00MNtSzasSQ" type="video/mp4">
</video>

## 4. 你可以向任何网站注入代码，片段（Snippets）和重写（Overrides）

(Chromium 浏览器)

[片段(Snippets)][21] 是开发者工具中针对当前网站运行脚本的一种方法。你可以在这些脚本中使用[控制台实用工具][14]，它是编写和存储通常在控制台中执行的复杂 DOM 操作脚本的绝佳方法。你可以通过片段编辑器或命令菜单在当前文档的窗口上下文中运行脚本。在后一种情况下，以 ! 开头，然后键入要运行的代码段名称即可。

[重写(Overrides)][22] 允许你存储远程脚本的本地副本，并在页面加载时覆盖它们。举例来说，如果你的整个应用程序的构建过程很慢，而你又想试一试，这就非常好用。此外，它还是替代第三方网站恼人脚本的绝佳工具，而无需使用浏览器扩展。

## 5. 你可以检查(inspect)和调试(debug)的东西比你知道的要多得多！

(Chromium 浏览器)

你可能从 Google Chrome、Brave 或 Microsoft Edge 等浏览器中了解到 Chromium 开发者工具，但它们在更多环境中可用。任何基于 Electron 的应用程序都可以启用这些工具，你可以使用这些工具窥探引擎盖下的内容，看看产品是如何完成的。例如，这可以在 GitHub Desktop、Visual Studio Code 中使用，你甚至可以使用开发者工具调试浏览器的开发者工具！

如果你检查一下开发者工具，就会发现它们是用 HTML、CSS 和 TypeScript 编写的。使用这些技术是一个令人兴奋的环境，因为你知道你的代码将在哪个渲染引擎中运行,这在网络上是永远无法知道的。

[![使用另一个开发者工具实例检查 Chromium 开发工具][23]](https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png)

### Edge 在 Visual Studio Code 的开发者工具

(Microsoft Edge 使用 VS Code 扩展)

这些工具的可嵌入性也使我们能够为你提供一种在浏览器之外使用它们的方法。[Microsoft Edge Tools for Visual Studio Code][2] 扩展将这些工具带到了 Visual Studio Code 中。这样，你就可以在代码编辑器旁边使用可视化调试工具，而不必总是在两者之间跳来跳去。当你开始调试会话并单击 "开发者工具"（Developer Tools）图标时，工具就会打开，或者首次打开时，系统会提示你安装扩展。

[![Visual Studio 代码调试栏中的检查按钮](https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png)][24]

[![在 Visual Studio Code 实例中打开 Microsoft Edge 开发者工具](https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png)][25]

## 6. 肮脏的秘密

与开发者工具进行亲密合作，并获得反馈和使用信息，这让我了解了一些 `肮脏的秘密`。第一个是，尽管我们所有人都对开发者工具的所有惊人功能感到非常兴奋，但用户只使用了其中的一小部分。许多在演示和视频教程中被誉为自切片面包以来最好的东西实际上很少被打开，更不用说被使用了。我以为这是因为缺乏文档记录，所以我们花费了大量的时间来更新[DevTools 文档][26]，以确保其中的所有内容都被描述和解释，但这不是问题所在。文档似乎是人们在谷歌、Stack Overflow 或社交媒体渠道没有找到任何结果后的最后手段。

### 开发人员的工具变得复杂不堪，如何解决这个问题？

(Microsoft Edge)

可能是因为浏览器的开发者工具在过去几年中有机地发展壮大，让人目不暇接。这让我很烦恼，我认为我们应该做得更好。说到开发者工具，我有一句口头禅：

> 开发者工具不应期望人们成为专家，而应随着时间的推移把他们变成专家。

我们正在想办法让这一切变得更容易，你很快就会在 Microsoft Edge 中看到这些想法。我们的一个想法是 “聚焦模式(Focus Mode)”。我们不再向你显示所有工具和选项卡，而是将工具按不同的使用情况进行分类，如 “元素/CSS 调试(Elements/CSS debugging)”、“源代码/JavaScript 调试(Sources/JavaScript Debugging)” 或 “网络检查(Network inspection)”。然后，我们只显示相关工具，隐藏所有可能会造成混淆或碍事的工具。

[![聚焦模式下的开发者工具，只显示当前环境下所需的工具](https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png)][27]

我们正在开发的另一项功能是 “信息覆盖(informational overlays)”。你可以获得一个帮助按钮，打开开发者工具的叠加信息，解释每个工具是什么、如何使用以及提供文档链接。我们希望这将使人们更容易了解更多的功能。

[![开发者工具，通过覆盖图解释每种工具的含义](https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png)][28]

### 编写代码与调试结果之间仍然存在脱节现象

(Microsoft Edge)

尽管现在的工具令人惊叹，但编写和调试之间仍然存在脱节。大多数情况下，我们在编写代码、创建应用程序后，会在浏览器中查看哪些地方无法正常工作。然后，我们使用浏览器开发者工具来调整和修复这些问题。然后，我们还需要解决一个大问题：如何将使用浏览器开发者工具创建的更改返回到代码中？大多数情况下，答案是 "复制并粘贴，或者试着记住需要更改的内容"。

我们目前正在研究两种方法来简化这一过程。一种是在可用时用 Visual Studio Code 代替开发者工具内的编辑器，并在使用浏览器开发者工具时更改硬盘上的文件。另一种是 VS Code 扩展的一部分，可以在使用开发者工具时更改编辑器中的源代码，但在更改磁盘上的文件时仍有最终决定权。我[在 Edge 博客上描述了问题和可能的解决方案][29]，你也可以观看以下两个截屏视频，了解它们的实际效果。

Visual Studio 代码中的 CSS 镜像:

如果…… Visual Studio Code 成为浏览器内开发者工具的编辑器，会发生什么？

## 7. 你们是开发人员工具的受众和客户！

（适用于所有浏览器，但此处显示的渠道仅限于 Microsoft Edge）

作为开发人员，你是开发人员工具的主要受众。我们愿意听取你的反馈意见，最近对工具的许多更改都是外部开发人员要求的直接结果。我们通过提供直接与我们联系的上下文方式，尽可能地简化这一过程。例如，Visual Studio Code 扩展有显著的链接和按钮供你报告问题(report issues)和功能需求(request features)。

[![VS 代码扩展中提供的上下文链接截图，用于要求新功能、提交错误和了解实验情况](https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png)][30]

[扩展插件的源代码][31] 也在 GitHub 上，你可以在那里[提交 issues][32]。

浏览器内的开发者工具也有一个直接向我们提供反馈的按钮。为了便于你提供可操作的反馈，该按钮包含了大量自动信息。

[![Microsoft Edge 浏览器开发工具中内置的反馈工具](https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png)][33]

它会自动记录发生问题的 URL、截图并发送诊断数据。我们还要求你提供电子邮件，以备我们需要更多信息，你还可以添加附件和如何重现问题的信息。我们每天都会检查这些反馈，很多伟大的发明和错误修复都来自于此。

[分享到 Mastodom][34]

[分享到 Twitter][35]

## 我的其它作品

-   [开发人员倡导手册][36]
    -   [在 Amazon 购买][37]
    -   [在 Leanpub 购买][38]
-   技能分享课程:
    -   [优化开发人员工作流程的工具和技巧][39]
    -   [改善产品无障碍环境的工具][40]
    -   [JavaScript 工具包： 编写更简洁、更快速、更优秀的代码][41]
    -   [揭开人工智能的神秘面纱： 了解机器学习][42]

[Christian Heilmann 博客][43]  
[chris@christianheilmann.com 邮箱][44] (请不要就客座文章与我联系，我不做这种事！)
我是一个首席项目经理，在德国柏林生活和工作。

主题由 Chris Heilmann 制作。SVG 图标由 [Dan Klammer][45] 制作。由 MediaTemple 主持。由 Coffee 和 Spotify Radio 提供技术支持。

[获取订阅，所有酷孩子都在使用 RSS！][46]

[1]: https://news.ycombinator.com/item?id=29071700
[2]: https://aka.ms/devtools-for-code
[3]: https://cityjsconf.org/
[4]: https://www.youtube.com/watch?v=q_qzHzIVxw4
[5]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-filters
[6]: https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_-1024x510.png
[7]: https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png
[8]: https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png
[9]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-log
[10]: https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png
[11]: https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png
[12]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png
[13]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide14.png
[14]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/utilities
[15]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png
[16]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/live-expressions
[17]: https://christianheilmann.com/2021/08/24/using-console-log-on-any-website-logpoints-let-you-do-that/
[18]: https://christianheilmann.com/2021/07/30/using-console-log-debugging-in-visual-studio-code/
[19]: https://docs.microsoft.com/microsoft-edge/visual-studio-code/microsoft-edge-devtools-extension#browser-debugging-with-microsoft-edge-devtools-integration-in-visual-studio-code
[20]: https://christianheilmann.com/wp-content/uploads/2021/10/Slide24-842x1024.png
[21]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/snippets
[22]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/overrides
[23]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png
[24]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png
[25]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png
[26]: https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/
[27]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png
[28]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png
[29]: https://blogs.windows.com/msedgedev/2021/10/21/improved-authoring-debugging-devtools-visual-studio-code/
[30]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png
[31]: https://github.com/microsoft/vscode-edge-devtools
[32]: https://github.com/microsoft/vscode-edge-devtools/issues
[33]: https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png
[34]: https://mastodon.social/share?text=Developer%20Tools%20secrets%20that%20shouldn%E2%80%99t%20be%20secrets%20https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/
[35]: http://twitter.com/share?url=https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/&text=Developer
[36]: https://developer-advocacy.com
[37]: https://www.amazon.com/dp/B0BKNTPDFJ/
[38]: https://leanpub.com/developer-advocacy-handbook
[39]: https://skl.sh/3uKu5G1
[40]: https://skl.sh/3eCFWRR
[41]: https://skl.sh/2CpiTGZ
[42]: https://skl.sh/2MHkYl1
[43]: http://christianheilmann.com
[44]: mailto:chris@christianheilmann.com?subject=NOT%20A%20GUEST%20POST%20REQUEST&body=I%20understand%20this%20blog%20does%20not%20do%20guest%20posts
[45]: https://github.com/danklammer/bytesize-icons
[46]: https://christianheilmann.com/feed/
