---
title: 【青铜三人行】二叉树中的最大路径和
date: 2020-05-06
authors:
  - demongodYY
  - TingYinHelen
  - glowd
categories:
  - Article
  - Algorithm
tags:
  - 二叉树
toc: true
original: https://zhuanlan.zhihu.com/p/138447429
---

每周一题，代码无敌~ 这一次，青铜三人行决定在五一假期期间挑战一道难度为「困难」的题目：

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    loading="lazy" lazyload="1"
    src="//player.bilibili.com/player.html?aid=285596520&bvid=BV1yf4y1m7Le&cid=187460008&page=1"
></iframe>

<!-- more -->

## 二叉树中的最大路径和

[力扣 ​leetcode-cn.com][1]

给定一个**非空**二叉树，返回其最大路径和。

本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径**至少包含一个**节点，且不一定经过根节点。

### 示例 1

```text
输入：[1, 2, 3]

       1
      / \
     2   3

输出：6
```

### 示例 2

```text
输入：[-10, 9, 20, null, null, 15, 7]

   -10
   / \
  9  20
    /  \
   15   7

输出: 42
```

## 解题思路

因为这次题目相对来说比较困难，因此就以一种思路来说明。

这道题的难点在于，题目中要求取的“任意节点出发”的路径，且不一定经过根节点。导致在如何迭代求取上，陷入了一个比较复杂的境地。

为了求解这个问题，我们需要将题目先简化一下，分步骤完成：

### 求取某一节点为起始的最大路径和

```JavaScript
function maxChildrenPathValue(node) {
  if (node == null) return 0;

  const leftPathVal = maxChildrenPathValue(node.left),
    rightPathVal = maxChildrenPathValue(node.right);

  const maxPathValue = Math.max(leftPathVal, rightPathVal) + node.val;

  return Math.max(maxPathValue, 0);
}
```

在这一步中，我们递归求取了某一个节点为开始的**单边最大路径和**，值得注意的是，如果取出来的值是负值，则设为 0，意为「舍弃」掉这条路径。

### 求取经过某一根节点的最大路径和

完成了上一步，我们就可以求取经过某一特定根节点的最大路径和了，即把「某个节点的值」与「左边最大路径和」和「右边最大路径和」相加：

```JavaScript
function getRootMaxPathVal(root) {
  const leftMaxPathVal = maxChildrenPathValue(root.left),
    rightMaxPathVal = maxChildrenPathValue(root.right);

  return leftMaxPathVal + rightMaxPathVal + root.val;
}
```

### 遍历求取整颗二叉树的最大路径值

有了上面的基础，我们就可以遍历整个二叉树，来求取所有节点的最大路径和，并取出其中的最大值来作为整颗二叉树的最大路径和了，在这里我们用了二叉树前序遍历，并使用了一个全局变量 `result` 来记录最大值：

```JavaScript
function preorderTraversal(root) {
  if (!root) return;

  const value = getRootMaxPathVal(root);

  if (value > result) result = value;

  preorderTraversal(root.left),
  preorderTraversal(root.right);
}
```

到此我们就可以解出这道题目了，完整代码如下：

```JavaScript
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
function maxPathSum(root) {
  let result = -Infinity;

  function maxChildrenPathValue(node) {
    if (node == null) return 0;

    const leftPathVal = maxChildrenPathValue(node.left),
      rightPathVal = maxChildrenPathValue(node.right);

    const maxPathValue = Math.max(leftPathVal, rightPathVal) + node.val;

    return Math.max(maxPathValue, 0);
  }

  function getRootMaxPathVal(root) {
    const leftMaxPathVal = maxChildrenPathValue(root.left),
      rightMaxPathVal = maxChildrenPathValue(root.right);

    return leftMaxPathVal + rightMaxPathVal + root.val;
  }

  function preorderTraversal(root) {
    if (!root) return;

    const value = getRootMaxPathVal(root);

    if (value > result) result = value;

    preorderTraversal(root.left),
    preorderTraversal(root.right);
  }

  preorderTraversal(root);

  return result;
}
```

## 优化

同样的解题思路下，Helen 发现到在**求取某一节点为起始的最大路径和**这一步的时候，已经在对二叉树进行遍历了，那能不能直接在一次递归遍历中解出题目呢？Helen 对代码进行了优化：

```JavaScript
function maxPathSum(root) {
  let max_sum = -Infinity;

  function max_gain(root) {
    if (root === null) return 0;

    const left_gain = Math.max(max_gain(root.left), 0),
      right_gain = Math.max(max_gain(root.right), 0);

    const newPath = root.val + left_gain + right_gain;

    max_sum = Math.max(newPath, max_sum);

    return root.val + Math.max(left_gain, right_gain);
  }

  max_gain(root);

  return max_sum;
}
```

代码简洁多了，运行也更快了！你有没有发现两个解法的共同之处和不同之处呢？

## Extra

最后依然是曾大师的 Go 语言 show time~

```Go
func maxPathSum(root *TreeNode) int {
    var val = INT_MIN();

    subMaxPathSum(root, &val);

    return val;
}

func subMaxPathSum(root *TreeNode,val *int) int{
    if (root == nil) return 0;

    left := subMaxPathSum(root.Left, val);
    right := subMaxPathSum(root.Right, val);
    threeSub := root.Val + max(0, left) + max(0, right);
    twoSub := root.Val + max(0, max(left, right));
    *val = max(*val, max(threeSub, twoSub));

    return twoSub;
}

func INT_MIN() int{
    const intMax = int(^uint(0) >> 1);
    return ^intMax
}

func max(x, y int) int {
    if x < y
        return y

    return x
}
```

![](https://pic3.zhimg.com/v2-1cb2ef07ced0438a01b6bffe9c0ca4ce_b.jpg)

结果依然很惊人啊…… 嗯……

## 最后

这次的题目有些复杂，但通过简化题目、拆解步骤，也可以让困难的题目得到解决。而日常编程的过程中，也是在将复杂问题简单化、步骤化的一个过程。最后留个小问题，之前提到过，所有的递归都可以用循环来解决，那么在第一步的递归中，如果用循环解决该怎么做呢？下周见~

[1]: https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/
