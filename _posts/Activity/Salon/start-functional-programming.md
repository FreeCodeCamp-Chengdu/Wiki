---
title: 走近函数式编程
date: 2019-03-26 23:30:37
categories:
  - Activity
  - Salon
tags:
  - offline
  - functional-programming
toc: true
authors:
  - TechQuery
  - jiangyuzhen

# Activity meta
description: "FCC 成都社区 Coffee & Code #34"
start: 2019-03-30 14:00
end: 2019-03-30 17:00
address: 成都市高新区世纪城路1029号天华社区乡愁故事馆
mentors:
  - demongodYY
  - TechQuery
workers:
  - jiangyuzhen
---

## 活动简介

活动主要是从[书香墨剑][1]学习**函数式编程**的心得体会出发，来谈谈他所理解的函数式编程，并以一个**罗马数字转阿拉伯数**的例子和大家一起探讨函数式编程的使用以及对我们日常编码的影响，最后大家互相分享对函数式的理解。

## 活动信息

- 时间：2019.03.30 14:00 - 17:00
- 地点：成都市高新区世纪城路 1029 号天华社区乡愁故事馆

## 活动流程

| 时刻  |        内容        |
| :---: | :----------------: |
| 14:00 |    Who are you?    |
| 14:10 |  函数式编程之我知  |
| 14:40 |  函数式编程之我用  |
| 16:00 | 你谈？我谈？共交流 |

## 我可以参与？

欢迎有一定 JavaScript 基础、对函数式编程有所理解 或 有不同看法的小伙伴~

<!-- more -->

---

## 活动总结

一来到活动现场，**乡愁故事馆**的文艺气息似乎可以冲淡些技术宅的刻板，为沙龙参会者带来一丝别样的感受。本次沙龙的主讲**书香墨剑**不但网名文艺，平时也是个话剧爱好者，活动场地也是他亲自选的，果然符合本人气质。

### What ?

业内人士深知源自**数学思想**的“函数式编程”抽象晦涩，书香便以**川菜经典“回锅肉”的做法**来讲解 ——

```javascript
function 煮(肉) {
  return 水 + 火 + 肉;
}

function 炒(...食材) {
  return 油 + 火 + 食材 + 盐;
}

let 回锅肉 = 炒(炒(切(煮(肉))), 切(蒜苗), 豆豉);
```

数学函数 **层层传递输入输出**、**不引用外部变量**、**不改变外部变量** 等主要原则一目了然。

但往往概念、原则讲多了，初学者要么云里雾里、要么颠覆三观，不能**对新学的思想方法正确认识、合理运用**。于是书香便一针见血地来了个“敲黑板”三连 ——

> 函数式编程只是一个**编程范式**！
>
> 区别在于**对程序的抽象看法**；
>
> 一段程序里可以存在多个编程范式。

那…… 书香你怎么看？一图胜千言 ——

![](https://p.ssl.qhimg.com/t01bce2290d50ad4f6b.png)

### Why & How ?

既然函数式编程有这么多好处 ——

1. 方便**单元测试**
2. 减少**外部状态**干扰
3. 通过**高阶抽象**方便阅读、灵活组合

那该怎么用呢？很多人连 JavaScript 数组自带的 `map()`、`filter()`、`reduce()` 都还用不好呢。无妨，我们动手演练 ——

```javascript
//  罗马数字与阿拉伯数字的对应
const roman_arab = {
  I: 1,
  V: 5,
  X: 10,
  L: 50,
  C: 100,
  D: 500,
  M: 1000
};
```

以上罗马数字从大到小排列，只有下列情况除外：

- I 可在 V、X 前，表减 1（如 IV 表示 4）
- X 可在 L、C 前，表减 10
- C 可在 D、M 前，表减 100

### 再举个栗子

本次活动主题较为抽象，主讲者虽尽力借鉴生活中的例子来讲解“函数式编程”的理念，但大家还是比较困惑；加上参会者自带电脑的又比较少，原定的**现场动手实践**较早结束。于是[水歌][2]便上台即兴分享了自己做过的 TDD 习题 —— [基于函数式编程的**保龄球算法**][3]，让大家更清晰地认识函数式编程的应用范式。

1. 每局比赛每人有十轮投球
2. 每轮共有两次机会来打倒全部十个瓶子
3. 一次打完为“全中”，本轮得分为 10 + 后两球分数
4. 两次打完为“补中”，本轮得分为 10 + 后一球分数
5. 第十轮全中、补中加 2、1 次击球

[规则][4]看完，想必*全中、补中的事后加分*最让人头疼…… 但函数式编程范式却像我们上学解数理化题时“套公式”一样，让计算程序一目了然 ——

> 每局分数 = 本局第一次击球分数 + 本局第二次击球分数 +
> 　　下局第一次击球分数 x 补中系数 +
> 　　(下局第一次击球分数 + 下局第二次击球分数) x 全中系数

用代码描述如下：

```javascript
//  两数相加
function sum(first, second) {
  return first + second;
}

//  补中系数
function isSpare(first, second) {
  return first !== 10 && first + second === 10 ? 1 : 0;
}

//  全中系数
function isStrike(first) {
  return first === 10 ? 1 : 0;
}

//  每轮分数
function round(this_first, this_second, next_one, next_two) {
  return (
    this_first +
    this_second +
    isSpare(this_first, this_second) * next_one +
    isStrike(this_first, this_second) * (next_one + next_two)
  );
}
```

上述代码虽清晰明了，但还有个关键没有解决 —— 未来的分数怎么办？这就需要学习函数式编程的第一难关**柯里化**来解决：

```javascript
function curry(origin) {
  //  原函数声明了几个参数
  const count = origin.length;
  //  包装函数
  return function wrapper() {
    //  当前是否已有足够的参数
    return count > arguments.length
      ? //  还差几个，返回一个记住已传入参数的新函数
        wrapper.bind(this, ...arguments)
      : //  够了，执行原函数
        origin.apply(this, arguments);
  };
}
```

这样我们就可以把击球数一个个填进去，最终合计即可算出总分 ——

```javascript
const curry_round = curry(round),
  score = [];

score[0] = curry_round(3)(7)(3)(4); // 13

score[1] = curry_round(3)(4)(0)(0); // 7

score[2] = curry_round(10)(0)(3)(7); // 20

//  以此类推……
```

讲到这儿，台下部分同学频频点头，露出了“原来如此”的笑容~

---

## 活动反馈

以下是会后部分同学的建议 ——

1. 编辑器居然没有设置自动保存
2. 不写注释一时爽，一直不写一直爽
3. 其实讲一讲 Redux，对大家理解函数式编程帮助比较大
4. 其实讲一些实用的例子会更好，也更容易理解！
5. 听了对柯里化理解还不够
6. 罗马数字转阿拉伯数字的例子不错。阿拉伯数字转罗马数字的例子可以不用讲了，比较冗余了。可以加点例子: 如果其中某个函数抛出异常了怎么处理。

---

## 参考资料

- [主讲 PPT](https://ppt.baomitu.com/d/f77aa0b4)

- [罗马、阿拉伯 数字互转](https://github.com/demongodYY/Integer-convert-Roman)

- [保龄球计分](https://github.com/TechQuery/Functional-Bowling)

[1]: https://github.com/demongodYY
[2]: https://tech-query.me/
[3]: https://github.com/TechQuery/Functional-Bowling
[4]: http://www.csa.edu.hk/~pe/bowling/means.htm
