---
title: 【青铜三人行】删除链表的倒数第 N 个节点
date: 2020-05-05 19:13:52
authors:
  - demongodYY
  - TingYinHelen
  - glowd
categories:
  - Article
  - Algorithm
tags:
  - 链表
  - 数据结构
  - LeetCode
  - Bronze-3
toc: true
original: https://zhuanlan.zhihu.com/p/133873072
thumbnail: https://pic1.zhimg.com/v2-84568441e08f60e9900a87e5d53c131a_1200x500.jpg
---

每周一题，代码无敌。这周，「青铜三人行」为你带来了一道关于“链表的题目”。

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    loading="lazy" lazyload="1"
    src="//player.bilibili.com/player.html?aid=882851982&bvid=BV1nK4y1k75D&cid=180931816&page=1"
></iframe>

## 删除链表的倒数第 N 个节点

给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

### 示例

> 给定一个链表: 1 -> 2 -> 3 -> 4 -> 5, 和 n = 2；
> 当删除了倒数第二个节点后，链表变为 1 -> 2 -> 3 -> 5。

### 说明

给定的 n 保证是有效的。

### 进阶

你能尝试使用一趟扫描实现吗？

[力扣 ​ leetcode-cn.com][2]

<!-- more -->

## 啥是链表

要完成这道题，首先就得了解一下啥是链表？简单来说，**链表**是一种数据结构，它由一系列离散的**节点**组成。其特点是，每个节点上除了自己的数据以外，还会有一个或两个指针指向下一个或者上一个节点，使得这些节点可以**链**起来。

其中，只有指向下一个节点的链表称为**单向链表**，它只能从前一个节点到下一个节点一个方向来查找其中的节点数据：

![](https://pic4.zhimg.com/80/v2-1cf240bd549d6f37c5cc3526d97fefbb_1440w.png)

而**双向链表**则拥有两个指针，分别指向之前和之后的节点：

![](https://pic2.zhimg.com/80/v2-ba4663d586b2181725d053226212c125_1440w.png)

而在 JS 中，这道题目里给我们设定了链表的结构，很明显，是一个单向列表：

```javascript
function ListNode(val) {
  this.val = val;
  this.next = null;
}
```

## 解法一：两次遍历找到对应的节点

了解了链表的数据结构以后，这道题就不难解决了。不过题目里有个小小的花招，即要求寻找「倒数第 n 个节点」。因为是单向链表，我们没法倒着寻找节点，因此我们很容易想到先找到整个链表的长度，计算出要找的元素的正向位置，然后再从头遍历，进行删除：

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
function removeNthFromEnd(head, n) {
  let node = head,
    length = 0;

  while (true) {
    if (node === null) break;

    node = node.next;
    length++;
  }
  node = head;

  if (n === length) return head.next;

  for (let i = 0; i < length - n - 1; i++) node = node.next;

  node.next = node.next.next;

  return head;
}
```

## 解法二：转离散节点为连续节点

这道题数据量较小，因此运行的速度都比较快。于是向着题目中「只扫描一次」这个进阶目标前进。书香提出了一种方法，既然题目的难点在于链表不容易反向查找，那么把它映射成一个连续的数据结构不就可以解决了吗？于是很自然想到了应用**数组**完成了题目：

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
function removeNthFromEnd(head, n) {
  let node = head;
  const arrNodes = [];

  while (true) {
    if (node === null) {
      const length = arrNodes.length;
      if (length === n) head = head.next;
      else arrNodes[length - n - 1].next = arrNodes[length - n + 1];
      break;
    }
    arrNodes.push(node);
    node = node.next;
  }
  return head;
}
```

值得注意的是，当需要删除的元素是第一个元素的时候，容易造成数组的越界，需要特殊处理：

```javascript
if (length === n) head = head.next;
```

## 解法三：双指针法

上面书香的解法虽然在一次扫描中完成了任务，却额外引入了一个数组的外部结构。有没有更好的办法呢？Helen 和 曾大师对于这个问题，采用了新的办法：题目要求删除倒数第 n 个节点，那么我只需要在我当前扫描到的节点指针之后相隔 n 的节点再设置一个指针，到后一个指针越界的时候，当前节点就是需要删除的节点了：

![](https://pic4.zhimg.com/80/v2-54faa5b2a27d7d1ece3ea52a2dcaaa1b_1440w.jpg)

代码如下：

```javascript
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0);
  dummy.next = head;

  let l = dummy,
    r = dummy,
    offset = n;

  while (offset--) r = r.next;

  while (r.next) (l = l.next), (r = r.next);

  l.next = l.next.next;

  return dummy.next;
}
```

## Ertra

最后，曾大师 go 语言的福利时间又到啦，同样是双指针，你能看出有什么不同吗？

```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    store := head
    i := 0
    p := head

    for p != nil {
        if (i > n) {
            store = store.Next
        }
		p = p.Next
        i++
	}
    // 删除头节点
    if (i == n) {
        return store.Next
    }
    store.Next = store.Next.Next

    return head
}
```

而在时间和空间上，相当惊人，嗯嗯……

![](https://pic3.zhimg.com/80/v2-0a19434826b19ed6fa7a451b3671f18e_1440w.jpg)

## 结尾

这周的题目相对来说比较简单，主要是说明了链表的数据结构。链表相对于**数组**来说，更容易**插入、删除**其中的节点，而数组比起来则更容易**查找**到某个节点（想想为什么？）。两个数据结构相辅相成，在不同的应用场景选择合适的数据结构，可以让你的程序运行起来事半功倍哦！

这次的题目就这样了，欢迎通过 bronze_3@163.com 邮箱联系我们，下周见！

[1]: https://www.bilibili.com/video/BV1nK4y1k75D
[2]: https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/
