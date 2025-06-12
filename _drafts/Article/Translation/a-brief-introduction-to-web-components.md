---
title: Web Components 简介
date: 2025-05-08T15:32:15.694Z
author: Mark Mahoney
authorURL: https://www.freecodecamp.org/news/author/markm208/
originalURL: https://www.freecodecamp.org/news/a-brief-introduction-to-web-components/
translator: "yiwei"
reviewer: ""
---

<!-- more -->

在之前的一篇文章中，我简要介绍了 [React][1]。这篇教程介绍了一种构建基于组件的前端的替代方法。它涵盖了 [Web Components][2] 的基础知识，以构建模块化、可复用的网页应用元素。

Web Components 是一组标准化的浏览器 API，允许你创建具有封装功能的自定义可复用 HTML 元素。它们帮助开发者创建可以跨不同框架甚至无需任何框架使用的独立组件。

本教程假设你有一些基本的编程经验，并且能够熟练阅读和编写 JavaScript 代码。你应了解变量、函数、循环、对象、类，以及 JavaScript 在浏览器中的工作原理。你不需要了解任何关于 Web Components 的知识即可开始学习本教程。

这里介绍的四个课程选取自我的免费 Code Playbacks 书籍中：  

> [从前端到后端的 Web 开发入门][3]  
> 作者： Mark Mahoney

这本书可以在 [Playback Press][4] 免费获取。该书是现代网页开发的实践指南，涵盖了从 JavaScript 核心特性到使用各种工具和技术构建全栈应用的所有内容。

每个课程都以 [Code Playback][5] 的形式呈现，这是一种交互式代码练习，展示了程序随时间的变化过程，并附有我对所发生内容的解释。这种格式帮助你专注于代码变更背后的思考过程。

要查看回放，请点击左侧面板中的评论。每个评论都会更新编辑器中的代码并突出显示更改。阅读解释，研究代码，如果有问题，可以使用内置的 AI 导师。这里有一个简短的视频，展示了如何使用 Code playback：

<iframe width="560" height="315" src="https://www.youtube.com/embed/uYbHqCNjVDM" style="aspect-ratio: 16 / 9; width: 100%; height: auto;" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen="" loading="lazy"></iframe>

在这个视频介绍之后，你可能想探索官方的 Web Components 资源：[MDN Web Components 指南][6]。

## 目录

## **Web Components 第 1 部分：你的第一个自定义元素**

本课程首先介绍如何使用 Web Components 构建用户界面，Web Components 允许你将 HTML、CSS 和 JavaScript 打包成一个可复用的单一元素。

这些元素可以像 `h1`、`div` 和 `img` 等内置 HTML 元素一样使用。当你希望将页面拆分成为更小的、独立的、可复用的部分，如标题、表格或表单时，它们特别有用，同时保持每部分代码有序且隔离。

在此回放中，你将学习：

- 如何创建一个表示自定义 HTML 元素的 JavaScript 类   
- 如何访问 Web Component 的属性    
- 如何在 HTML 中创建和使用 Web Component

本课程重点介绍创建一个 `LegendHeader` 组件，该组件可以在整个 Web 应用中重复使用。你将看到自定义元素如何像普通 HTML 元素一样具有属性（例如 `img` 标签中的 `src`），并且如何从 JavaScript 代码中更改这些属性，从而触发相应的方法。

**在此查看回放：** [**Web Components 第 1 部分- LegendHeader**][7]

## **Web Components 第 2 部分：数据通信**

在上一课的基础上，这个回放演示了如何创建多个共享数据的组件。我将增强 `LegendHeader` 组件以显示正在跟踪的图例数量，并添加一个新的 `LegendTable` 组件，该组件使用 HTML 表格显示数据库中的所有 CS 图例。

本课引入的一个关键概念是让顶级元素来承载 Web 应用的数据，并将数据的变更传递给依赖这些数据的组件。这种方法使组件管理更加有序且易于维护。

在此回放中，你将学习：

- 如何创建和使用多个组件
- 如何在 Web 组件中设置和使用“可观察属性”
- 顶级元素如何管理数据并在数据变化时通知组件

**在此查看回放:** [**Web Components Part 2- LegendTable**][8]

## **Web Components 第 3 部分：自定义事件**

本课通过添加一个 `NewLegendForm` 组件来扩展应用，允许用户向数据库添加新的图例。回放介绍了“冒泡”通过 DOM 的自定义事件概念，使顶层元素能够控制数据请求。

你将了解为什么通常最好不要让组件自己管理应用的数据。由顶级元素管理整个 Web 应用程序的数据使组件更简单、更可复用，因为它们不需要相互了解或直接通信。

在此回放中，你将学习：

- 组件如何生成自定义事件以请求数据而不是自己管理数据
- 顶级元素如何监听事件并处理它们
- 顶级元素如何访问数据并将其传递给需要的组件

**在此查看回放：** [**Web Components 第 3 部分 - NewLegendForm**][9]

## **Web Components 第 4 部分：构建完整应用**

在本节最后的课程中，我将把所有内容整合起来，创建一个带有身份验证的完整应用。我将添加一个新的 `AuthBox` 组件，并实现一个轻量级身份验证系统，以便只有注册并登录的用户才能向数据库添加新的图例。

Playback 在服务器上使用会话来控制用户访问，并加强了组件化架构中集中数据管理的重要性。

在此回放中，你将学习：
- 如何使用 Web Component 实现用户身份验证
- 如何将所有组件集成到一个完整的、功能齐全的应用中
- 基于组件的 Web 应用的数据管理最佳实践

**在此查看回放：** [**Web Components 第 4 部分 - AuthBox**][10]

## **React 与 Web Components 比较**

React 和 Web Components 的主要区别可以总结如下：

| 关键属性 | React | Web Components |
|:---:|:---:|:---:|
| 组件定义 | 通常使用函数定义组件 | 使用扩展 HTMLElement 的 JavaScript 类 |
| 工具要求 | 需要安装、转译、打包工具（npm、webpack、babel 等） | 原生浏览器支持，无需构建工具或安装 |
| 模板语法 | 使用 JSX，一种 JavaScript 中的类 HTML 语法 | 在字符串或 HTML 模板中使用标准 HTML |
| DOM 更新 | 使用虚拟 DOM 高效批处理并最小化实际 DOM 操作 | 直接操作 DOM，通常对频繁更新的优化较少 |
| 属性类型 | 接受各种数据类型作为 props（字符串、数组、对象、函数） | 在 HTML 中仅接受字符串作为属性 |
| 渲染模型 | 声明式：描述 UI 应该是什么样子，React 处理更新 | 更具命令性：直接操作 DOM 以响应更改 |
| 样式封装 | 无内置样式封装（需要 CSS-in-JS 或 CSS 模块） | 使用 Shadow DOM 内置样式封装 |
| 浏览器支持 | 通过 polyfills 在所有浏览器中工作 | 仅现代浏览器（可能需要对旧浏览器的 polyfills） |
| 生态系统 | 拥有大量库和工具的大生态系统 | 较小但不断增长的生态系统 |

## 总结

这四课涵盖了 Web Components 的基础知识，但还有更多内容可供探索。Web Components 提供了一种基于标准的方法来创建可复用元素，而无需外部库或框架，使其在构建可维护和可移植的代码时特别有价值。

如果你发现这种格式有帮助，请探索本书的其余部分，看看如何使用现代工具和方法从头开始构建完整的 Web 应用程序。

Web Components 只是基于组件的 Web 开发的一种方法。继续构建，继续阅读，并在准备好进一步学习时尝试其他 Playback 。

如果你对这些 Playback 有任何反馈，我很乐意听取你的意见。你可以通过 mark@playbackpress.com 联系我。

---

[1]: https://www.freecodecamp.org/news/a-brief-introduction-to-react/
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Web_components
[3]: https://playbackpress.com/books/webdevbook/
[4]: https://playbackpress.com/books/
[5]: https://markm208.github.io/
[6]: https://developer.mozilla.org/en-US/docs/Web/API/Web_components
[7]: https://playbackpress.com/books/webdevbook/chapter/3/8
[8]: https://playbackpress.com/books/webdevbook/chapter/3/9
[9]: https://playbackpress.com/books/webdevbook/chapter/3/10
[10]: https://playbackpress.com/books/webdevbook/chapter/3/11
[11]: mailto:mark@playbackpress.com