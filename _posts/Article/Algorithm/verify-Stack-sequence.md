---
title: 【青铜三人行】每周一题之验证栈序列
date: 2020-06-08
authors:
  - demongodYY
  - TingYinHelen
  - glowd
categories:
  - Article
  - Algorithm
tags:
  - LeetCode
  - Bronze-3
  - 数据结构
  - 堆栈
original: https://zhuanlan.zhihu.com/p/146666166
thumbnail: https://pic1.zhimg.com/v2-55cbd65728928ca8b0775d6166d77e08_1200x500.jpg
toc: true
---

先说一个消息，为了方便互相交流学习，青铜三人行建了个微信群，感兴趣的伙伴可以扫码加下面的小助手抱你入群哦！

<figure>
    <img src="https://pic2.zhimg.com/80/v2-ce9c805020726a8a18c4c870a8282985_1440w.jpg">
    <figcaption>青铜三人行小助手（其实是 Helen）</figcaption>
</figure>

---

每周一题，代码无敌~ 这次的主题是 **「贪心算法」**：

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    loading="lazy" lazyload="1"
    src="//player.bilibili.com/player.html?aid=668415886&bvid=BV1ta4y1v7h8&cid=200192481&page=1"
></iframe>

<!-- more -->

## [验证栈序列][1]

给定 pushed 和 popped 两个序列，每个序列中的 值都不重复，只有当它们可能是在最初空栈上进行的推入 push 和弹出 pop 操作序列的结果时，返回 `true`；否则，返回 `false`。

### 示例 1

```text
输入：pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
输出：true
解释：我们可以按以下顺序执行：
push(1), push(2), push(3), push(4), pop() -> 4,
push(5), pop() -> 5, pop() -> 3, pop() -> 2, pop() -> 1
```

### 示例 2

```text
输入：pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
输出：false
解释：1 不能在 2 之前弹出。
```

### 提示

1.  `0 <= pushed.length == popped.length <= 1000`
2.  `0 <= pushed[i], popped[i] < 1000`
3.  `pushed` 是 `popped` 的排列

## 栈结构

要做这道题，首先得了解一下什么是 **「栈」**。为此书香搬来了**维基百科**上的解释：

> **堆栈**（英语：stack）又称为**栈**或**堆叠**，是[计算机科学][2]中的一种[抽象数据类型][3]，只允许在有序的线性数据集合的一端（称为堆栈顶端，英语：top）进行加入数据（英语：push）和移除数据（英语：pop）的运算。因而按照后进先出（LIFO, Last In First Out）的原理运作。

> **栈**常与另一种有序的线性数据集合[队列][4]相提并论。 **栈**常用一维[数组][5]或[链表][6]来实现。

这个定义看起来多，其实也没什么大不了的。**栈**本身就是一个**数组**或者**链表**，只是**人为定义**它获取数据的方式只能从栈的**顶端**获取，因此遵循**先进后出、后进先出**的规则罢了。

想象**栈**就是一摞盘子，你只能在最上面放盘子或者拿走盘子。对应起来，栈的操作就有两个：

- push 操作，往栈顶放入一个数据。
- pop 操作，从栈顶取走一个数据。

![](https://pic1.zhimg.com/80/v2-413de4df18cc333944027e29514aa99c_1440w.jpg)

## 解题思路

回到这道题目，一开始看起来题目有点绕，让人不知道要做什么。后来 Helen 提议，既然题目要求是考虑在*最初空栈上进行的推入 push 和弹出 pop 操作*，那么我们不妨就建立一个**空栈**尝试用程序的方式来模拟一遍操作的流程，看看会不会明朗点：

```javascript
function validateStackSequences(pushed, popped) {
  const stack = [];

  pushed.forEach(ele => {
    stack.push(ele);
    stack.pop();
  });
  return !stack.length;
}
```

这样我们就建立了一个**栈**，并且在按题目中 `pushed` 数组的顺序将元素 `push` 进栈， 然后再按照**同样的顺序** `pop` 出去。

不过这样子就跟数学题里面一边放水，一边加水的疯狂管理员一般，返回的结果肯定为 `true`。

回头再看看题目，发现其实就是在这个**一边增加一边移出**的过程上，添加了一个条件：只能按照 `poped` 的顺序来 `pop` 数据，看看能不能将 `stack` 清空 。

再拆解一下目标，就更明确了：

- 按照 `pushed` 的顺序将元素 `push` 入栈。
- 在 `push` 的过程中尝试 `pop` 元素。
- `pop` 元素的顺序要和 `poped` 的顺序一样。

要满足这三个条件，一个方法就是，**尝试在 `push` 的每一步时，尽可能按照指定顺序 `pop` 出所有的元素**。根据这个思路，Helen 给出了题解：

```javascript
function validateStackSequences(pushed, popped) {
  const stack = [];
  let popIndex = 0;

  for (const val of pushed) {
    stack.push(val);

    while (stack.length !== 0 && stack[stack.length - 1] === popped[popIndex]) {
      stack.pop();
      popIndex++;
    }
  }
  return stack.length === 0;
}
```

书香的思路一模一样，只是把代码写的更短了点 ：

```javascript
function validateStackSequences(pushed, popped) {
  const stack = [];

  pushed.forEach(ele => {
    stack.push(ele);
    while (stack.length && stack[stack.length - 1] === popped[0]) {
      stack.pop();
      popped.shift();
    }
  });
  return !stack.length;
}
```

### Extra Go

对于书香和 Helen 这样的初级选手，通过实现一个**栈**结构来模拟题目中要求的操作，解出题目，就已经开心地到一边去玩耍了 ~

但对于追求完美的曾大师来说，`push` 和 `pop` 这两个操作，都非常消耗计算资源。而这种把数据一边 `push` 一边 `pop` 的疯狂操作显然是不能容忍的。

为此，他写出了 **2 米长**的 Go 语言代码：

```go
func validateStackSequences(pushed []int, popped []int) bool {

    if len(popped) == 0 {
        return true
    }

    pushedValues := make(map[int]int) //存储所有的已经入栈的值和数组索引

    left := 0
    right := 0

    for i := 0; i < len(pushed); i++ {
        if pushed[i] == popped[0] {
            pushed[i] = -1  // 出栈
            left = i-1
            right = i+1
            break
        } else {
            pushedValues[pushed[i]] = i
        }
    }

    for j := 1; j < len(popped); j++ {

        if _, ok := pushedValues[popped[j]]; ok { // 值已经加入stack了
            for left >= 0 {
                if pushed[left] == -1 {
                    left--
                } else {
                    break
                }
            }

            if left < 0 {
                left = 0
            }

            if popped[j] != pushed[left] {
                return false
            } else { // 值相等，出栈
                 pushed[left] = -1
                 left--
            }
        } else { // 值没有加入stack，继续往前找
             for right < len(popped) {
                if popped[j] == pushed[right] { // 找到了
                    pushed[right] = -1 //出栈
                    left = right - 1 // 重新赋值left
                    right ++  // 重新赋值right
                    break
                } else { // 没有找到，继续往前
                    pushedValues[pushed[right]] = right
                    right++
                }
            }
        }
    }
    return true
}
```

嗯…… 8 ms 的运行时间……

![](https://pic4.zhimg.com/80/v2-14d286e072f598b6b8ea2728f07a1d8b_1440w.jpg)

## 贪心算法

不知道你有没有发现，这道题目，在开始的时候看起来比较绕，但是真正实现起来并没有那么困难？

其实关键点在于 **「分而治之」**，将任务中的每一步拆分开来，并且在每一步时，都**尽可能去寻找最优解**，再将每一步的最优解达到合起来，看是否能达成目标。

这种思路的算法就称为 **「贪心算法」**，它在遇到**寻找最优解**问题的情况下，能够提供很大的帮助。

下次见~

[1]: https://leetcode-cn.com/problems/validate-stack-sequences/
[2]: https://zh.wikipedia.org/wiki/%25E8%25A8%2588%25E7%25AE%2597%25E6%25A9%259F%25E7%25A7%2591%25E5%25AD%25B8
[3]: https://zh.wikipedia.org/wiki/%25E6%258A%25BD%25E8%25B1%25A1%25E8%25B3%2587%25E6%2596%2599%25E5%259E%258B%25E5%2588%25A5
[4]: https://zh.wikipedia.org/wiki/%25E4%25BD%2587%25E5%2588%2597
[5]: https://zh.wikipedia.org/wiki/%25E9%2599%25A3%25E5%2588%2597
[6]: https://zh.wikipedia.org/wiki/%25E9%2580%25A3%25E7%25B5%2590%25E4%25B8%25B2%25E5%2588%2597
