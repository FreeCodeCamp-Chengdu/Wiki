---
title: 青铜三人行——每周一题@两数之和
date: 2020-04-07 14:52:52
authors:
  - demongodYY
categories:
  - Article
  - Algorithm
tags:
  - LeetCode
  - Bronze-3
toc: true
original: https://zhuanlan.zhihu.com/p/126195652
thumbnail: https://pic4.zhimg.com/v2-3cd8da7b0160b55dbc1c974712227d61_1200x500.jpg
---

哈喽，大家好，欢迎来到**青铜三人行**的每周一题现场。在接下来的时间里，我们三人（[Helen][1]、[书香][2]、曾大师）会在每周选择一道编程算法题来完成，和大家一起探讨一下解题的思路。所谓每周一题，代码无敌，欢迎各位小伙伴们一起进入我们的刷题之旅~

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    src="//player.bilibili.com/player.html?aid=242740480&bvid=BV1Le41147ok&cid=174716511&page=1"
></iframe>

> 因为个人水平有限，我们的解法不一定是最优的，只是希望抛转引用，分享自己的思路，带动和大家一起练习编程技能。大家有任何建议，也可以通过  bronze_3@163.com  邮箱联系我们~

<!-- more -->

话不多说，就进入我们这周的题目吧，它出自  [LeetCode][3]  的第一题：

## 题目

给定一个整数数组  `nums`  和一个目标值  `target`，请你在该数组中找出和为目标值的那**两个**整数，并返回他们的**数组下标**。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

> 给定 nums = [2, 7, 11, 15], target = 9
> 因为 nums[0] + nums\[1] = 2 + 7 = 9
> 所以返回 [0, 1]

## 解法一

拿到题目，Helen 心想，这次题目难度不大。略一思忖，要在数组中找到满足某个条件的两个数，一个双重循环搞定即可：

```JavaScript
function twoSum(num, target) {
    for (const index in num) {
        for (const _index in num) {
            if (index !== _index && num[index] + num[_index] === target) {
                return [index, _index];
            }
        }
    }
}
​
//作者：Helen
//链接：https://leetcode-cn.com/circle/discuss/5cC2dU/view/p3MA3g/
//来源：力扣（LeetCode）
//著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

顺利通过题目！但是效率似乎并不理想……

![](https://pic2.zhimg.com/80/v2-3d15c49489c35180fa05829991a44e21_1440w.jpg)

## 解法二

接下来就要优化算法，Helen 审视代码，发觉影响效率的主要原因恐怕是在于双重循环所造成的 O(n²) 复杂度。要想提高效率，恐怕就要在一重循环里搞定题目。但如何在一次迭代中找到两个数的关系呢？确实颇费考虑…… 算法领域中，空间与时间通常如同鱼和熊掌一般不可兼得。空间换时间…… Helen 灵光一现，对了，一次迭代中表现两个数的关系，可以在 map 结构中用查找 key 的方式呀。考虑至此，信手写出了第二版代码：

```JavaScript
function twoSum(nums, target) {
    const numsMap = {};
    for (const index in nums) {
        numsMap[nums[index]] = index;
    }
    for (const index in nums) {
        const complement = target - nums[index];
        if (numsMap[complement] && numsMap[complement] !== index) {
            return [index, numsMap[complement]];
        }
    }
}
​
//作者：Helen
//链接：https://leetcode-cn.com/circle/discuss/5cC2dU/view/p3MA3g/
//来源：力扣（LeetCode）
//著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

如此一来，时间复杂度减为 O(n)， 速度果然大大提高：

![](https://pic4.zhimg.com/80/v2-285d249e04e2eed104c89cf6cb417a7f_1440w.jpg)

[书香作为一个函数式编程的拥护者][4]，平日里对  `map`、 `filter`、  `reduce`  等方法都记在心里。看到这个代码，心想恐怕在循环中对数组的频繁引用是一个可以优化的点，于是利用 JavaScript 中内置的  `reduce`  方法稍作修改：

```JavaScript
const twoSum = function(nums, target) {
    const objNums = nums.reduce((acc, num,index) => {
        acc[num]=index;
        return acc},{});
​
    for (let i=0; i<nums.length;i++) {
        const num = nums[i];
        const other = objNums[target-num];
        if(other!==undefined && other!==i){
            return [i,other]
        }
    }
    return;
};
​
//作者：demongodYY
//链接：https://leetcode-cn.com/circle/discuss/5cC2dU/view/8eOrHo/
//来源：力扣（LeetCode）
//著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

时间和空间上居然都有所提高，看来 JavaScript 对内置方法的优化果然到位：

![](https://pic2.zhimg.com/80/v2-0eb9613ee93f5b960c94edc7d4b3f2f9_1440w.jpg)

## 解法三

于此同时，Helen 则进一步对代码进行了优化。题目要求只需要找到满足条件的两个数，那么有可能在没有遍历完的时候就能找到呀。如此一来，就不必提前将整个数组转换成 map 结构，而是边转换边查找，在找到满足条件的时候即可返回：

```JavaScript
function twoSum(nums, target) {
    const numsMap = {};
    for (const index in nums) {
        const complement = target - nums[index];
        if (numsMap[complement]) {
            return [ index, numsMap[complement]];
        }
        numsMap[nums[index]] = index;
    }
}
​
//作者：Helen
//链接：https://leetcode-cn.com/circle/discuss/5cC2dU/view/p3MA3g/
//来源：力扣（LeetCode）
//著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

如此一来，代码性能大大地得到了优化：

![](https://pic3.zhimg.com/80/v2-ab8ec6d96d152a6590f3866d0bb36fe2_1440w.jpg)

## extra

最后，由曾大师为我们在 Go 语言中展现了一把对内存的极致管理，也体现了对于不同编程语言特性的优化差别：

```Go
func twoSum(nums []int, target int) []int {
    for i := 0; i < len(nums); i++ {
        for j := i+1; j < len(nums); j++ {
            if nums[i]+nums[j] == target {
                return []int{i,j}
            }
        }
    }
    return []int{}
}
​
//作者：glowd
//链接：https://leetcode-cn.com/circle/discuss/5cC2dU/view/omqRef/
//来源：力扣（LeetCode）
//著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

天啊，内存消耗仅为 2.9MB，在所有 Go 提交中击败了 100% 的用户！😲😲😲

![](https://pic4.zhimg.com/80/v2-35eddfbe6adc838664fee3c70f3503db_1440w.jpg)

## 结尾

OK，这就是咱们**青铜三人行**的第一次分享的全部内容啦，虽然很多地方还不完善，但也希望凭借一点微薄的力量，提起大家对编程算法题的兴趣。

如果看到了这次分享，你有一些灵感的话，请**立即**拿起手中的**键盘**，打开 LeetCode 的网站找到题目先刷一遍，并与我们或者身边的小伙伴们分享你的思路~

如果有任何的建议和意见的话，也欢迎大家随时联系我们，我们的联系邮箱是  bronze_3@163.com。

下周见！

[1]: https://www.jianshu.com/u/ad825678a16f
[2]: https://github.com/demongodYY
[3]: https://leetcode-cn.com/
[4]: https://fcc-cd.dev/activity/salon/start-functional-programming/

