---
title: 我不再使用 React.setState 的 3 个理由
authorURL: ""
originalURL: https://blog.cloudboost.io/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e
translator: TechQuery
reviewer: ""
---

[![Michel Weststrate](https://miro.medium.com/v2/resize:fill:88:88/1*XWCjUzWvB5KUrmXT1kxOOA.jpeg)][1]

[![CloudBoost](https://miro.medium.com/v2/resize:fill:48:48/1*a8_IkAXKt7ff5oUv_QmQSw.png)][2]

[Michel Weststrate][3]

·

[Follow][4]

Published in

[CloudBoost][5]

·

5 min read

·

Jun 15, 2016

[][6]

\--

51

[][7]

Listen

Share

Since a few months I’ve stopped using React’s _setState_ on all my new React components. Don’t get me wrong, I didn’t stop having local component state, I just stopped using React to manage it. And it’s been delightful!

Using _setState_ is tricky for beginners. Even experienced React programmers easily introduce subtle bugs when using React’s own state mechanism, like this:

![](https://miro.medium.com/v2/resize:fit:640/1*v2qbGqdV8wM1G4ixs7woEw.gif)

Bug introduced by forgetting that React state is asynchronous; the log is always one item behind

The excellent Reacts [docs][8] sum up everything that could go wrong when using _setState_:

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*OtKvlJDJPjbSVM6o-yjL_Q.png)

To summarize, there are 3 issues when using _setState:_

## 1\. setState is asynchronous

Many devs don’t realize this initially, but setState is _asynchronous._ If you set some state, than take a look at it, you will still see the old state.This is the most tricky part of setState. setState calls don’t look asynchronous, and naively calling setState introduces very subtle bugs. The next gist demonstrates that nicely:

At first glance this might look fine. Two event handlers and a utility function that fire the _onSelect_ event if provided. However, this _Select_ component has a bug that is nicely demonstrated in the GIF above. _onSelect_ is always fired with the previous value of _state.selection_, because _setState_ hasn’t done it’s job _yet_ when the _fireOnSelect_ helper is invoked. I think the least React could do here is rename the method to _scheduleState_ or make the callback required.

> This bug is easily fixed, the tricky part is realizing it’s there.

**_Edit 25–jan–2018:_** [**_Dan Abramov_**][9] **_has been so kind to write down an extensive an sound_** [**_explanation_**][10] **_why setState is asynchronous by design._**

## 2\. setState causes unnecessary renders

The second issue with _setState_ is that it always triggers a re-render. Often those re-renders are unnecessary. You can use the [_printWasted_][11] method from the React performance tools to find out when this happens. But roughly speaking there are several reasons why a re-render may be unnecessary:

-   The new state is actually the same as the previous one. This can often be addressed by implementing _shouldComponentUpdate_. You may already be using a (pure render) library to solve this for you.
-   Sometimes the changed state is relevant for the rendering, but not under all circumstances. For example when some data is only conditionally visible.
-   Thirdly, as pointed out in Aria Buckles’ [talk at React Europe 2015][12], sometimes instance state is not relevant for the rendering at all! This is often householding state related to managing event listeners, timer ID’s etc.

## 3\. setState is not sufficient to capture all component state

Following the last point above, not all component state should be stored and updated using _setState_. More complex components often have administration that is needed by lifecycle hooks to manage timers, network requests, events etc. Managing those with _setState_ not only causes unnecessary renders, but also causes related lifecycle hooks to be triggered again, leading to weird situations.

# Managing local component state with MobX

(Surprise, surprise) At [Mendix][13] we already rely on MobX to manage all our stores. However, we were still using React’s own state mechanism for local component state. Recently, we switched to managing local component state with MobX as well. That looks like this:

For completeness sake:

![](https://miro.medium.com/v2/resize:fit:640/1*LPl8MGfkPyWGtRERQdw_3w.gif)

No unexpected bugs when using a state mechanism that is synchronous

The above code snippet is not only more concise, MobX also addresses all of the setState related issues:

Changes to the state are immediately reflected in the local component state. This makes our logic simpler and code reuse easier. You don’t have to compensate for the fact that the state might not have been updated yet.

MobX determines at runtime which observables are relevant for rendering. So observables that are temporarily irrelevant for the rendering, won’t cause a re-rendering. Until they are relevant again. For this reason, there are also no rendering penalties (or lifecycle issues) when marking fields as _@observable_ that are not relevant for rendering at all.

So renderable and non-renderable state is treated uniformly. In addition, state stored in our components now works the same as state stored in any of our stores. This makes it trivial to refactor components, and move local component state into a separate store or vice versa. Which is demonstrated in this [egghead][14] tutorial.

> MobX effectively turns your components into small stores

Furthermore, rookie mistakes like assigning values directly to the _state_ object cannot be made anymore when using observables for state. Oh, and don’t worry about implementing _shouldComponentUpdate_ or _PureRenderMixin,_ MobX already takes care of that as well. Finally, you might be wondering, what if I want to wait until _setState_ has finished? Well, you can still use the \_compentDidUpdate l\_ifecycle hook for that.

## Sounds cool! How do I get started with MobX?

Pretty simple, follow the [10 minute interactive introduction][15] or watch the aforementioned video. You can simply take a single component from your code base, slap _@observer_ on it and introduce some _@observable_ properties. You don’t even have to replace your existing _setState_ calls, they continue to work while using MobX. Although, within a few minutes you might find them so convoluted that you will replace them anyway :). (Oh, and if you don’t like decorators, no worries, it works with [good ol’ ES5 as well][16]).

## TL;DR:

I’ve stopped using React to manage local component state. I use MobX instead. Now React is truly “just the view” :). MobX now manages both local component state and state in stores. It is concise, synchronous, efficient and uniform. From experience, I’ve learned that MobX is even easier to explain to React beginners than React’s own _setState._ It keeps our components clean and simple.

-   [JSBin][17] using _setState_ for state management
-   [JSBin][18] using _MobX observables_ for state management

[1]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[2]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[3]: https://medium.com/@mweststrate?source=post_page-----ab73fc67a42e--------------------------------
[4]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Fde4496bfa1e2&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&user=Michel+Weststrate&userId=de4496bfa1e2&source=post_page-de4496bfa1e2----ab73fc67a42e---------------------post_header-----------
[5]: https://blog.cloudboost.io/?source=post_page-----ab73fc67a42e--------------------------------
[6]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fcloudboost%2Fab73fc67a42e&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&user=Michel+Weststrate&userId=de4496bfa1e2&source=-----ab73fc67a42e---------------------clap_footer-----------
[7]: https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fab73fc67a42e&operation=register&redirect=https%3A%2F%2Fblog.cloudboost.io%2F3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e&source=-----ab73fc67a42e---------------------bookmark_footer-----------
[8]: https://facebook.github.io/react/docs/component-api.html
[9]: https://medium.com/u/a3a8af6addc1?source=post_page-----ab73fc67a42e--------------------------------
[10]: https://github.com/facebook/react/issues/11527#issuecomment-360199710
[11]: https://facebook.github.io/react/docs/perf.html#perf.printwastedmeasurements
[12]: https://youtu.be/2Qu-Ulrsfl8?t=12m09s
[13]: http://www.mendix.com/
[14]: https://egghead.io/lessons/javascript-mobx-and-react-intro-syncing-the-ui-with-the-app-state-using-observable-and-observer
[15]: https://mobxjs.github.io/mobx/getting-started.html
[16]: https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/syntax.md#react-components
[17]: http://jsbin.com/yelazuvamo/edit?js%2Cconsole%2Coutput=
[18]: http://jsbin.com/sofezamavi/1/edit?js%2Cconsole%2Coutput=
