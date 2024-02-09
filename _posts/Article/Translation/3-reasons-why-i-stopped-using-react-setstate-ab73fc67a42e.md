---
title: 我不再使用 React.setState 的 3 个理由
authorURL: ""
originalURL: https://blog.cloudboost.io/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e
translator: TechQuery
reviewer: ""
---

[![Michel Weststrate](https://miro.medium.com/v2/resize:fill:88:88/1*XWCjUzWvB5KUrmXT1kxOOA.jpeg)][2] 

[![CloudBoost](https://miro.medium.com/v2/resize:fill:48:48/1*a8_IkAXKt7ff5oUv_QmQSw.png)][3]

[Michel Weststrate][4] · [Follow][5]

Published in [CloudBoost][6] · 5 min read · Jun 15, 2016

[][7]

\--

51

[][8]

自几个月前，我已在所有我新写的 React 组件弃用 React 的 _setState_ 。别误会我，我没有弃用本地组件状态，我只是不再用 React 去管理它，并且令人愉快！

使用 _setState_ 对初学者来说很棘手。即使经验丰富的 React 程序员在使用 React 自有状态机制时，也很容易引入微妙的 bug，例如：

![](https://miro.medium.com/v2/resize:fit:640/1*v2qbGqdV8wM1G4ixs7woEw.gif)

忘记 React 状态是异步的而引入了 bug，日志输出总是在后面一项。

这篇优秀的 React [文档][9]总结了错误使用 _setState_ 的各种情况：

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*OtKvlJDJPjbSVM6o-yjL_Q.png)

总的来说，使用 _setState_ 有三种问题：

## 1. setState 是异步的

很多开发者一开始没有意识到这一点，但 setState 是 _异步的_ 。如果你设置一些状态后再查看一下，你将仍看到旧状态。这是 setState 最棘手的部分。setState 调用看起来不像异步的，天真地调用 setState 会引入非常微妙的 bug。接下来的 gist 非常好地演示了这一点：

猛地一看，这可能看起来还不错。两个事件处理器和一个工具函数在 _onSelect_ 存在时触发事件。然而，这个 _Select_ 组件有个 bug，在上面的 GIF 图中被很好地演示。 _onSelect_ 总是带着 _state.selection_ 旧值被触发，因为 _fireOnSelect_ 工具函数执行时，_setState_ _还_ 没做完他的工作。我认为此处 React 至少可以把这个方法改名为 _scheduleState_，或让回调参数变成必填。

> 这个 bug 很容易修复，最棘手的部分是意识到它在这儿。

**_编辑于 2018 年 1 月 25 日：_** [**_Dan Abramov_**][10] **_已经热心地写下一个广为人知而有力的_** [**_解释_**][11] **_，从设计的角度讲为什么 setState 是异步的。_**

## 2. setState 引起没必要的渲染

使用 _setState_ 的第二个问题是，它总会触发重渲染。通常这些重渲染是不必要的。你可以使用 React 性能工具中的 [_printWasted_][12] 方法来找出它什么时间发生。但粗略地讲，有几种原因是一次重渲染为什么可能是不必要的：

-   新状态实际上和之前的一模一样。这通常可以通过实现 _shouldComponentUpdate_ 来解决。你可能已经使用了一个（纯渲染）库来解决。
-   有时改变的状态与渲染有关，但并不是所有情况都是。比如当有些数据只条件性可见。
-   第三，正如 Aria Buckles [在 React Europe 2015 上的演讲][13]所指出的那样，有时候实例状态根本与渲染无关！这通常是与管理事件监听器、计时器 ID 等相关的家务状态。

## 3\. setState is not sufficient to capture all component state

Following the last point above, not all component state should be stored and updated using _setState_. More complex components often have administration that is needed by lifecycle hooks to manage timers, network requests, events etc. Managing those with _setState_ not only causes unnecessary renders, but also causes related lifecycle hooks to be triggered again, leading to weird situations.

# Managing local component state with MobX

(Surprise, surprise) At [Mendix][14] we already rely on MobX to manage all our stores. However, we were still using React’s own state mechanism for local component state. Recently, we switched to managing local component state with MobX as well. That looks like this:

For completeness sake:

![](https://miro.medium.com/v2/resize:fit:640/1*LPl8MGfkPyWGtRERQdw_3w.gif)

No unexpected bugs when using a state mechanism that is synchronous

The above code snippet is not only more concise, MobX also addresses all of the setState related issues:

Changes to the state are immediately reflected in the local component state. This makes our logic simpler and code reuse easier. You don’t have to compensate for the fact that the state might not have been updated yet.

MobX determines at runtime which observables are relevant for rendering. So observables that are temporarily irrelevant for the rendering, won’t cause a re-rendering. Until they are relevant again. For this reason, there are also no rendering penalties (or lifecycle issues) when marking fields as _@observable_ that are not relevant for rendering at all.

So renderable and non-renderable state is treated uniformly. In addition, state stored in our components now works the same as state stored in any of our stores. This makes it trivial to refactor components, and move local component state into a separate store or vice versa. Which is demonstrated in this [egghead][15] tutorial.

> MobX effectively turns your components into small stores

Furthermore, rookie mistakes like assigning values directly to the _state_ object cannot be made anymore when using observables for state. Oh, and don’t worry about implementing _shouldComponentUpdate_ or _PureRenderMixin,_ MobX already takes care of that as well. Finally, you might be wondering, what if I want to wait until _setState_ has finished? Well, you can still use the \_compentDidUpdate l\_ifecycle hook for that.

## Sounds cool! How do I get started with MobX?

Pretty simple, follow the [10 minute interactive introduction][16] or watch the aforementioned video. You can simply take a single component from your code base, slap _@observer_ on it and introduce some _@observable_ properties. You don’t even have to replace your existing _setState_ calls, they continue to work while using MobX. Although, within a few minutes you might find them so convoluted that you will replace them anyway :). (Oh, and if you don’t like decorators, no worries, it works with [good ol’ ES5 as well][17]).

## TL;DR:

I’ve stopped using React to manage local component state. I use MobX instead. Now React is truly “just the view” :). MobX now manages both local component state and state in stores. It is concise, synchronous, efficient and uniform. From experience, I’ve learned that MobX is even easier to explain to React beginners than React’s own _setState._ It keeps our components clean and simple.

-   [JSBin][18] using _setState_ for state management
-   [JSBin][19] using _MobX observables_ for state management

[1]: https://blog.cloudboost.io/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e
[2]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[3]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[4]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[5]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Fde4496bfa1e2&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&user=Michel+Weststrate&userId=de4496bfa1e2&source=post_page-de4496bfa1e2----ab73fc67a42e---------------------post_header-----------
[6]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[7]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fcloudboost%2Fab73fc67a42e&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&user=Michel+Weststrate&userId=de4496bfa1e2&source=-----ab73fc67a42e---------------------clap_footer-----------
[8]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fab73fc67a42e&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&source=-----ab73fc67a42e---------------------bookmark_footer-----------
[9]: https://facebook.github.io/react/docs/component-api.html
[10]: https://medium.com/u/a3a8af6addc1?source=post_page-----ab73fc67a42e--------------------------------
[11]: https://github.com/facebook/react/issues/11527#issuecomment-360199710
[12]: https://facebook.github.io/react/docs/perf.html#perf.printwastedmeasurements
[13]: https://youtu.be/2Qu-Ulrsfl8?t=12m09s
[14]: http://www.mendix.com/
[15]: https://egghead.io/lessons/javascript-mobx-and-react-intro-syncing-the-ui-with-the-app-state-using-observable-and-observer
[16]: https://mobxjs.github.io/mobx/getting-started.html
[17]: https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/syntax.md#react-components
[18]: http://jsbin.com/yelazuvamo/edit?js%2Cconsole%2Coutput=
[19]: http://jsbin.com/sofezamavi/1/edit?js%2Cconsole%2Coutput=