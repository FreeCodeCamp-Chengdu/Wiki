---
title: Chrome 性能调试工具
date: 2019-07-08 12:14:00
updated: 2019-07-08 12:21:00
authors:
  - TingYinHelen
categories:
  - Article
  - Summary
tags:
  - Chrome
  - performance
  - debug

original: https://www.jianshu.com/p/ed219471b393
toc: true
---

> 2019 年 06 月 30 日 FCC 分享复盘
>
> [分享的示例][1]

上周在 FCC 分享了一次[浏览器的渲染性能调优][2]，在分享的过程中大家问到一些问题，并且在分享之后，我自己也发现了一点问题，所以想写一篇文章来总结和复盘。

<!-- more -->

## 问：z-index 和合成层有关系吗？

**答**：z-index 跟合成层没有关系，跟渲染层有关系。

参考：[层叠上下文][3]

> 层叠上下文是 HTML 元素的三维概念，这些 HTML 元素在一条假想的相对于面向（电脑屏幕的）视窗或者网页的用户的 z 轴上延伸，HTML 元素依据其自身属性按照优先级顺序占用层叠上下文的空间

渲染层保证页面元素以正确的顺序合成，这样才能正确的展示元素的重叠和半透明等等。满足形成层叠上下文条件的 LayoutObject 就会为其创建新的渲染层。

文档中的层叠上下文由满足以下任意一个条件的元素形成：

- 根元素 `<html />`
- z-index 值不为 `auto` 的绝对/相对定位
- 一个 z-index 值不为 `auto` 的 flex 项目 (flex item)，即：父元素 `display: flex|inline-flex`
- [`opacity`][4] 属性值小于 1 的元素（参考 [the specification for opacity][5]）
- [`transform`][6] 属性值不为 `none` 的元素
- [`mix-blend-mode`][7] 属性值不为 `normal` 的元素
- [`filter`][8] 值不为 `none` 的元素，
- [`perspective`][9] 值不为 `none` 的元素，
- [`isolation`][10] 属性被设置为 `isolate` 的元素，
- `position: fixed`
- 在 [`will-change`][11] 中指定了任意 CSS 属性，即便你没有直接指定这些属性的值（参考[这篇文章][12]）
- [`-webkit-overflow-scrolling`][13] 属性被设置 `touch` 的元素

某些特殊的渲染层会被认为是合成层。合成层有单独的 GraphicsLayer。

渲染层提升为合成层的情况：

- 3D transform
- `<iframe />`
- `<video />`
- 3D 或者 硬件加速的 2D Canvas 元素
- `will-change`

## 问：什么工具可以查看 CSS 的属性改变触发渲染的方法？

**答**：可以查看：[https://csstriggers.com/][14]，这个网站注明了在不同的浏览器内核下，每种 CSS 属性在默认和改变时会触发哪一种渲染操作，后面会有详细解释。

## 问：这次分享的是针对的什么渲染引擎

**答**：Blink

这里说一点点浏览器的历史吧，最开始 Chrome 是用的苹果公司的 WebKit，后来谷歌和苹果产生了分歧，谷歌从 WebKit 中复制了一个 Blink 项目，所以我们分析渲染性能是针对的 Blink。

对浏览器渲染引擎感兴趣的可以看：

- [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/][15]
- [WebKit 技术内幕][16]

虽然 Blink 是 WebKit 的一个分支，但是有很多细节不同。首先，谷歌把 WebKit 的代码梳理得可读性很高，比如 `RenderObject` 变成了 `LayoutObject`，`RenderLayer` 变成了 `PaintLayer`。所以这里我需要修正一下分享时的渲染流程图，现在 Blink 内核的渲染流程图是下面这个。

{% asset_img 247sqm6e3bl.webp  %}

当然除了代码可读性高了，渲染的细节也有改变。

分享的时候有人问到为什么我用了 `transform` 但是在 Layer tab 没有看到合成层。下来我查证了一下，先看一下下面这张 CSS trigger 的图

{% asset_img 1boy8hyxdin.webp  %}

最上边的 `B`、`G`、`W`、`E` 分别代表的是：Blink(Chrome)、Gecko(火狐)、WebKit(Safari)、EdgeHTML（Edge 的内核，不过微软已经宣布 Edge 浏览器将切换至 Chromium 内核，也就是 Blink）。

左侧显示的 CSS 属性。

左边的 BGWE 是设置属性（从默认值修改，相当于一开始没设置 CSS）。右边的 BGWE 是修改属性（对现有的属性值进行修改）。

下面每个小方块的颜色就是对应的渲染流程中触发的方法。

{% asset_img 9w7mjjnuw4g.webp  %}

从 `transform` 会触发的方法可以看到，在 Blink 下只触发了 Composite，而在 WebKit 下触发了 Layout、Paint、Composite。从上面列举的会触发合成层产生情况，可以看出 `transform` 只有在 3D 的情况下才会形成合成层，而在 2D 的情况下是没有合成层的。

{% asset_img 1w1sf5mg4ad.webp  %}

上面这个是我分享的优化成了 `transform` 的例子。如果我把每个小方块都改成 3D 渲染，layer 显示应该是这样的

{% asset_img 2mhwb4ftyt8.webp  %}

所以这里需要修正我所讲的“`transform` 都会形成合成层”，正确的解释应该是：

> 在所有引擎下 `transform` 2D 并不会形成合成层，只有 3D 才会形成合成层

在 Blink 引擎下，本身 `transform` 2D 不会触发 layout 和 paint，而在 WebKit 引擎下仍然会触发 layout 和 paint。

所以 Chrome 提升动画性能仍然会触发 layout 和 paint 的动画操作改为 `transform`。但是如果是 3d 情况下，形成合成层的元素太多，会很消耗内存，所以需要做一个平衡。

[1]: https://github.com/TingYinHelen/performance
[2]: https://fcc-cd.dev/activity/salon/deno-chrome/#%E3%80%8AChrome-DevTools-%E4%B9%8B-performance%E3%80%8B
[3]: https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context
[4]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/opacity
[5]: https://www.w3.org/TR/css-color-3/#transparency
[6]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform
[7]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/mix-blend-mode
[8]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter
[9]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/perspective
[10]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/isolation
[11]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change
[12]: http://dev.opera.com/articles/css-will-change-property/
[13]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-overflow-scrolling
[14]: https://csstriggers.com/
[15]: https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/
[16]: https://read.douban.com/ebook/35765513/?dct=Web&type=paid&dcc=35765513&dcm=douban&dcs=updates
