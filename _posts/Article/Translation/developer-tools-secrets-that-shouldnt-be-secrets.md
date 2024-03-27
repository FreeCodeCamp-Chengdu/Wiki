---
title: 揭开浏览器开发工具的秘密
authorURL: ""
originalURL: https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/
translator: ""
reviewer: ""
---

2021 年 11 月 1 日星期一下午 3:21

> **更新**: 由于这篇文章[正在 Hackernews 上热传](https://news.ycombinator.com/item?id=29071700)，我在每个标题后的括号中为每个提示添加了支持环境的信息。当我说明 `Chromium 浏览器` 时，指的是所有使用 `Chromium` 内核并具有所有开发者工具的浏览器。这包括 `Chrome` 浏览器、`Microsoft Edge`、`Brave` 以及其他更多浏览器。在此提醒一下： `Microsoft Edge` 是 `Windows 10/11` 系统自带的浏览器，基于 `Chromium`，因此从平台角度来看与 `Chrome` 浏览器类似。它们在用户体验和核心服务方面有所不同。`Edge` 开发工具与谷歌密切合作，将我们添加到产品中的工作带回 `Chromium` 核心。但是，我在这里谈到的一些东西是微软 `Edge` 的实验和独有功能，它可在 `Windows`、`Mac` 和 `Linux` 上使用。有些功能只能通过 [Edge DevTools for VS Code 扩展](https://aka.ms/devtools-for-code)在 `Visual Studio Code` 中使用。


这是我今年 9 月在 [CityJS](https://cityjsconf.org/) 上发表的演讲。我是 `Microsoft Edge` 开发人员工具的首席产品经理，这些都是我在开发工具、记录工具和查看用户反馈时遇到的问题。

您可以在 Youtube 上观看[演讲视频](https://www.youtube.com/watch?v=q_qzHzIVxw4)。

下面是我所写的所有内容：

## 1. Console 远不止是 log()!

（所有浏览器的开发工具都遵循该标准）

毫无疑问，除了元素工具，控制台是浏览器开发工具中使用最多的部分。特别是，人们喜欢通过在代码中添加 "console.log()/"来调试，以了解发生了什么。虽然这样做存在一些问题，也有更好的调试脚本的方法，但既然人们都这么做了，那我们就来谈谈如何让这种体验变得更好。

第一个问题是产品上线后未删除的日志信息塞满了控制台。查找所需的信息变得令人望而生畏，而解决这个问题的最好办法就是了解 [控制台过滤选项](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-filters) 。使用这些选项，您可以将控制台的报告过滤为您所关心的内容，并屏蔽掉大量杂音。

[![filtering options in the console tool](https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_-1024x510.png)](https://christianheilmann.com/wp-content/uploads/2021/10/console-filter-dropdown.msft_.png)

### 你想输出什么 logging?

使用 `console.log()` 的下一个问题是，我们似乎只记录了值，却忘了添加它们的来源。例如，当你使用下面的代码时，你会得到一个数字列表，但你不知道什么是什么。

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span>width<span style="color: #009900;">)</span>
console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span>height<span style="color: #009900;">)</span></pre></td></tr></tbody></table>

```javascript
console.log(width) 
console.log(height)
```
要解决这个问题，最简单的方法是用大括号把要记录的内容包起来。这样，控制台就会同时记录你想知道的内容的名称和值。

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span>width<span style="color: #009900;">}</span><span style="color: #009900;">)</span>
console.<span style="color: #660066;">log</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span>height<span style="color: #009900;">}</span><span style="color: #009900;">)</span></pre></td></tr></tbody></table>

```javascript
console.log({width}) 
console.log({height})
```

[![Using curly braces around variables in log messages logs their name and their value](https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide7.png)

### 添加到您的控制台词汇

[![Examples of warn, info and error messages and how they are displayed in the console](https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide10.png)

除了 `console.log()` 之外，您还可以使用 [更多方法](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/console-log) 。例如，`console.warn()` 记录警告，`console.info()` 记录信息，`console.error()` 记录错误信息。这不仅会使控制台的显示略有不同，而且还会使信息具有不同的日志级别，这意味着更容易对其进行过滤。

### 控制台中的错误和断言

[![The error method of console shows an error, and assert is a shortcut for an if statement with a console.log inside](https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide11.png)

在控制台中显示错误不同于抛出错误，但向维护或调试产品的人员显示问题的严重性仍然是个好主意。另一个有趣的方法是 `console.assert()`，它只在满足特定条件时记录信息。你经常会发现自己在编写 `if` 语句时，里面包含了一个 `console.log()`。使用`assert()`后，这条语句就变得多余了，在清理调试代码时就少了一件要担心的事。

### 追溯源头

[![Example of using console.trace() to track back where a call came from](https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide12.png)

你经常会发现自己添加了一个 `console.log('called')` 或类似内容来测试某个功能是否被触发。一旦有了这个功能，下一件事通常就是找出是什么调用了该方法。这就是`console.trace()`的作用，因为它不仅会告诉你某个方法被调用了，还会告诉你调用来自哪里。

### 将控制台信息分组

如果要记录的内容很多，可以使用 `console.group('name')` 和 `console.groupEnd('name')` 在控制台中以可折叠和可展开的方式封装信息。您甚至可以定义组默认是展开还是折叠。

[![An example of defining groups in the console](https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide13.png)

### 在控制台中以表格形式显示和过滤大量信息

如果您想以日志形式显示大量信息，那么阅读这些信息可能会令人望而生畏。`console.table()` 方法会在控制台中以表格形式显示类似数组的数据，你可以通过给它一个数组来过滤你想显示的属性。

例如，您可以使用 `let elms = document.querySelectorAll(':is(h1,p,script')` 从文档中获取所有 H1、段落和脚本元素，并使用 `console.table(elms)` 以表格形式显示这些信息。由于不同的元素有大量的属性和属性，因此生成的表格非常难读。如果使用`console.table(elms,['nodeName', 'innerText', 'offsetHeight'])` 过滤出你感兴趣的内容，你就会得到一个只有这些属性及其值的表格。

[![Code example using console.table() and its filtering options](https://christianheilmann.com/wp-content/uploads/2021/11/Slide14.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide14.png)

复制和粘贴这些信息时，表格结构会保持不变，因此它是将数据导入 `Excel` 或 `Word` 等的绝佳工具。

### 像jQuery一样 : $() and $$()

控制台自带了许多方便使用的方法，称为 [Console Utilities](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/utilities) 。其中两个非常有用的方法是 `$()` 和 `$$()`，它们分别是 `document.querySelector()` 和 `document.querySelectorAll()` 的替代方法。它们不仅会返回您所期望的 `nodeList`，还会将结果转换为数组，这意味着您可以直接在结果上使用 `map()` 和 `filter()`。以下代码将抓取当前文档中的所有链接，并返回一个数组，数组中的对象仅包含每个链接的 `href` 和 `innerText` 属性，即 `url` 和 `text` 属性。

<table><tbody><tr><td class="code"><pre class="javascript" style="font-family:monospace;">$$<span style="color: #009900;">(</span><span style="color: #3366CC;">'a'</span><span style="color: #009900;">)</span>.<span style="color: #660066;">map</span><span style="color: #009900;">(</span>a <span style="color: #339933;">=&gt;</span> <span style="color: #009900;">{</span>
  <span style="color: #000066; font-weight: bold;">return</span> <span style="color: #009900;">{</span>url<span style="color: #339933;">:</span> a.<span style="color: #660066;">href</span><span style="color: #339933;">,</span> text<span style="color: #339933;">:</span> a.<span style="color: #660066;">innerText</span><span style="color: #009900;">}</span>
<span style="color: #009900;">}</span><span style="color: #009900;">)</span></pre></td></tr></tbody></table>

```javascript
$$('a').map(a => { return {url: a.href, text: a.innerText} })
```

[![An example how the $$ function returns a collection of HTML elements that you can filter like any other array](https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide15.png)

## 2. 您可以在没有源访问权限的情况下进行日志记录,实时表达式和日志点
(Chromium browsers)

添加 "console.log() "的常规方法是将其放在代码中想要获取信息的地方。但你也可以深入了解你无法访问和更改的代码。[Live 表达式](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/live-expressions) 是在不更改代码的情况下记录信息的好方法。它们还能记录不断变化的值，而不会让控制台满屏日志，从而降低产品的运行速度。你可以从下面的截屏中看到它们的不同之处：
<video width="320" height="240" controls>
  <source src="https://christianheilmann.com/wp-content/uploads/2021/10/log-vs-live-expression-smaller.mp4" type="video/mp4">
</video>



日志点是一种特殊的断点。你可以右键单击开发工具源代码工具中 JavaScript 的任意一行，然后设置一个日志点。你会被要求提供一个要记录的表达式，并在执行该行代码时在控制台中获得其值。这意味着从技术上讲，你可以在网络上的任何地方注入一个 `console.log()`。我早在八月份就[写过关于日志点的文章](https://christianheilmann.com/2021/08/24/using-console-log-on-any-website-logpoints-let-you-do-that/)，你可以在下面的截屏中看到演示：

<video width="320" height="240" controls>
  <source src="https://youtu.be/DRRezUZvZ6I" type="video/mp4">
</video>

## 3. 你可以在浏览器外登录 - VS code 调试器
(Chromium Browsers and VS Code)

在 Visual Studio Code 中启动调试会话时，可以生成一个浏览器实例，调试控制台就会变成浏览器开发工具中的控制台。我曾在七月份的博客中详细介绍过这一点，所以你可以[阅读如何做到这一点](https://christianheilmann.com/2021/07/30/using-console-log-debugging-in-visual-studio-code/)。[官方文档](https://docs.microsoft.com/microsoft-edge/visual-studio-code/microsoft-edge-devtools-extension#browser-debugging-with-microsoft-edge-devtools-integration-in-visual-studio-code) 中也有更多内容。

[![](https://christianheilmann.com/wp-content/uploads/2021/10/Slide24-842x1024.png)](https://christianheilmann.com/wp-content/uploads/2021/10/Slide24.png)

你还可以观看我展示功能的一分钟视频：
<video width="320" height="240" controls>
  <source src="https://youtu.be/00MNtSzasSQ" type="video/mp4">
</video>

## 4. 您可以向任何网站注入代码。片段（Snippets）和重写（Overrides）。
(Chromium Browsers)

[Snippets](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/snippets) are a way in Developer Tools to run a script against the current web site. You can use the [Console Utilities](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/console/utilities) in these scripts and it is a great way to write and store complex DOM manipulation scripts you normally execute in the Console. You can run your scripts in the window context of the current document either from the snippets editor or from the command menu. In the latter case, start your command with an ! and type the name of the snippet you want to run.

[Overrides](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/javascript/overrides) allow you to store local copies of remote scripts and override them when the page loads. This is great if you have, for example, a slow build process for your whole application and you want to try something out. It is also a great tool to replace annoying scripts from third party web sites without having to use a browser extension.

## 5. You can inspect and debug much more than you know!
(Chromium Browsers)

You may know the Chromium developer tools from browsers like Google Chrome, Brave or Microsoft Edge, but they are available in a lot more environments. Any app that’s based on Electron can have them enabled and you can use the Tools to peek under the hood and see how the product was done. This works, for example, in GitHub Desktop, Visual Studio Code, or you can even debug the Developer Tools of the browser using Developer Tools!

If you inspect the Developer Tools, you will see that they are written in HTML, CSS and TypeScript. It is an exciting environment to use these technologies, as you you know the rendering engine your code will run in – something you never know on the web.

[![Inspecting the Chromium Developer tools with another instance of the developer tools](https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide36.png)

### Edge Developer Tools in Visual Studio Code
(Microsoft Edge via a VS Code extension)

The embeddable nature of the tools also allowed us to offer you a way to use them outside the browser. The [Microsoft Edge Tools for Visual Studio Code](https://aka.ms/devtools-for-code) extension brings the tools to Visual Studio Code. That way you can use the visual debugging tools right next to your code editor and you don’t need to jump between the two all the time.This also ties in with the “Console in Visual Studio Code” trick mentioned earlier. When you start a debugging session and you click the Developer Tools icon, the tools will open or – the first time – you will be prompted to install the extension.

[![Inspect button in the debug bar of Visual Studio Code](https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide39.png)

[![Microsoft Edge Developer tools open in an instance of Visual Studio Code](https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide40.png)

## 6\. Some dirty secrets…

Working intimately with developer tools and getting feedback and usage information taught me a few dirty secrets. The first one is that whilst we are all super excited about all the amazing features of developer tools, users only use a very small percentage of them. Many things heralded as the best thing since sliced bread in presentations and video tutorials are hardly every opened, let alone used. I thought this was about a lack of documentation and we spent a massive amount of time to update the [DevTools documentation](https://docs.microsoft.com/microsoft-edge/devtools-guide-chromium/) to ensure everything in them is described and explained, but that wasn’t it. Documentation is something people seem to go to as a last resort when they are stuck and Google/Stack Overflow/Social channels didn’t yield any results.

### Developer tools have become complex and are overwhelming – a few ideas how to fix that
(Microsoft Edge)

It might be that the plain fact is that the Developer Tools of browsers grew organically over the years and can be incredibly overwhelming to look at. And that bothers me and I think we should do better. Here’s my mantra when it comes to tools for developers:

> Developer tools should not expect people to be experts but turn them into experts over time.

We’re working on a few ideas to make that easier, and you will soon see those in Microsoft Edge. One idea we had is a “Focus Mode”. Instead of showing you all the tools and tabs we sorted the tools into different use cases, like “Elements/CSS debugging”, “Sources/JavaScript Debugging” or “Network inspection”. We then show only the relevant tools and hide all the ones that may be confusing or in the way.

[![Developer tools in focus mode, showing only what's needed in the current context](https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide45.png)

Another feature we are working on are “informational overlays”. You get a help button that allows you to turn on overlays for the developer tools, explaining what each of the tools is, how to use it and providing links to the documentation. We hope that this would make it easier for people to learn about more features.

[![Developer tools covered by overlays explaining what each of them are,](https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide46.png)

### There is still a disconnect between authoring code and debugging the outcome
(Microsoft Edge)

Whilst it is amazing what tools provide us these days there is still a disconnect between authoring and debugging. Most of the time we write our code, create the app and then go to the browser to see what doesn’t work. We then use the browser developer tools to tweak and fix these issues. And then comes the big issue we still need to fix: how do you get the changes you created using the browser developer tools back into your code? Most of the time, the answer is “copy and paste or try to remember what needs changing”.

We’re currently working on two ways to make this easier. One is to replace the in-devtools editor with Visual Studio Code when it is available and to change files on the hard drive as you use the browser developer tools. The other is part of the VS Code extension and changes the source code in the editor as you use the developer tools but still gives you the final say in changing the file on disk. I [described the problem and the possible solutions on the Edge blog](https://blogs.windows.com/msedgedev/2021/10/21/improved-authoring-debugging-devtools-visual-studio-code/) or you can watch the following two screencasts to see them in action.

CSS Mirroring in Visual Studio Code:

What if… Visual Studio Code became the editor of in-browser Developer Tools?

## 7\. You’re the audience and the clients of Developer Tools!
(Applies to all browsers, but channels shown here are Microsoft Edge only)

As a developer, you are the main audience for Developer Tools. We are open to your feedback and many of the recent changes to the tools are direct results from demands from outside developers. We try to make this as easy as possible by providing in-context ways to contact us directly. For example, the Visual Studio Code extension has prominent links and buttons for you to report issues and request features.

[![Screenshot of the in-context links provided in the VS Code extension to demand new features, file bugs and learn abour experiments](https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide56.png)

The [source code of the extension](https://github.com/microsoft/vscode-edge-devtools) is also on GitHub and you can [file issues](https://github.com/microsoft/vscode-edge-devtools/issues) there.

The in-browser developer tools also have a direct button to give us feedback. To make it easier for you to provide actionable feedback, the button includes a lot of automatic information.

[![The feedback tool built into the browser developer tools of Microsoft Edge](https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png)](https://christianheilmann.com/wp-content/uploads/2021/11/Slide58.png)

It records automatically what URL the issue happened on, takes a screenshot to include and offers to send diagnostic data. We also ask for you to provide an email in case we need more information and you can add attachments and info how to recreate the issue. We check this feedback daily, and a lot of great inventions and bug fixes came from that source.

[Share on Mastodon (needs instance)](https://mastodon.social/share?text=Developer Tools secrets that shouldn’t be secrets https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/)

[Share on Twitter](http://twitter.com/share?url=https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/&text=Developer Tools secrets that shouldn’t be secrets&via=codepo8)

## My other work:

-   [The Developer Advocacy Handbook](https://developer-advocacy.com)
    -   [Buy it on Amazon](https://www.amazon.com/dp/B0BKNTPDFJ/)
    -   [Buy it on Leanpub](https://leanpub.com/developer-advocacy-handbook)
-   Skillshare Classes:
    -   [Tools and Tips to Optimize Your Workflow as a Developer](https://skl.sh/3uKu5G1)
    -   [Tools for Improving Product Accessibility](https://skl.sh/3eCFWRR)
    -   [The JavaScript Toolkit: Write Cleaner, Faster & Better Code](https://skl.sh/2CpiTGZ)
    -   [Demystifying Artificial Intelligence: Understanding Machine Learning](https://skl.sh/2MHkYl1)

[Skip to search](#s)

[Christian Heilmann](http://christianheilmann.com) is the blog of Christian Heilmann [chris@christianheilmann.com](mailto:chris@christianheilmann.com?subject=NOT%20A%20GUEST%20POST%20REQUEST&body=I%20understand%20this%20blog%20does%20not%20do%20guest%20posts) (Please do not contact me about guest posts, I don't do those!) a Principal Program Manager living and working in Berlin, Germany.

Theme by Chris Heilmann. SVG Icons by [Dan Klammer](https://github.com/danklammer/bytesize-icons) . Hosted by MediaTemple. Powered by Coffee and Spotify Radio.

[Get the feed, all the cool kids use RSS!](https://christianheilmann.com/feed/)
$$
