---
title: 【青铜三人行】外篇之循环与递归
date: 2020-06-02 02:03:00
authors:
  - demongodYY
  - TingYinHelen
  - glowd
categories:
  - Article
  - Algorithm
original: https://zhuanlan.zhihu.com/p/145200610
thumbnail: https://pic1.zhimg.com/v2-a8daa9067d256ec8d3c7fcaaa72d5265_1200x500.jpg
toc: true
---

不知不觉青铜三人行已经做了两个月的题了，这次轻松点，看看不一样的吧。

## 机器擅长的事 —— 重复

作为专业的程序猿，经常被行业外的朋友问到，为什么要学习编程？其实，除了掌握技能提高工作效率、甚至成为职业以外。学习编程更重要的是：**思维训练**。

其实，计算机从一开始就是为了帮助人们解决复杂问题而设计出来的。而在这个过程中，计算机程序的「思考」模型是一个叫“图灵机”的计算模型，图灵机是图灵 (**Alan Mathison Turing**) 祖师爷模拟人思考而发明出来的。为什么图灵祖师爷要发明图灵机呢？是因为他想要试图以自己和自己周围的天才科学家的思维方式作为人类的具体实例，来抽象总结出一套解决问题的办法。所以说，计算机程序的运作方式其实是一种人类尝试**用简单的方式逐步去解决复杂问题**的天才的思考方式。

<!-- more -->

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    loading="lazy" lazyload="1"
    src="https://insights.thoughtworks.cn/think-as-a-machine/"
></iframe>

在如今的时代，计算机早已经充斥在我们生活的方方面面，想要更好地进行人机交互，或多或少地我们都需要一些「像机器一样」的思考方式。即使是作为专业程序员，不断培养自己**像机器一样思考**的思维模式也是必不可少的。

既然要像机器一样去思考，那么不妨从计算机最擅长的事情 —— 重复，开始说起吧。下面是来自 _TED-Ed_ 中「[Think like coder][1]」系列课程的第一节，讲的就是计算机的重复 —— 循环。

<iframe
    frameborder="0" allowfullscreen
    allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
    src="https://www.youtube.com/embed/KFVdHDMcepw">
</iframe>

## 各种编程语言的循环

来看看在实际编程中，不同编程语言的循环写法有什么不同吧！

### for 循环

#### C

```c
int jj;

for (jj = 0; jj < 10; jj++)
    printf("%d, ", jj);
// => prints "0, 1, 2, 3, 4, 5, 6, 7, 8, 9, "

printf("\n");
```

#### Java

```java
// for loop structure => for(<start_statement>; <conditional>; <step>)
for (int fooFor = 0; fooFor < 10; fooFor++)
    System.out.println(fooFor);
    // Iterated 10 times, fooFor 0->9

System.out.println("fooFor Value: " + fooFor);
```

#### JavaScript

```javascript
for (var i = 0; i < 5; i++) {
  // will run 5 times
}
```

#### Python

```python
for i in range(4):
    print(i)
```

```python
animals = ["dog", "cat", "mouse"]

for i, value in enumerate(animals):
    print(i, value)
```

#### Rust

```rust
 // Ranges
for i in 0u32..10 {
    print!("{} ", i);
}
println!("");
// prints `0 1 2 3 4 5 6 7 8 9 `
```

#### Go

```go
for x := 0; x < 3; x++ { // ++ is a statement.
    fmt.Println("iteration", x)
}
```

```go
for key, value := range map[string]int{"one": 1, "two": 2, "three": 3} {
    // for each pair in the map, print key and value
    fmt.Printf("key=%s, value=%d\n", key, value)
}
```

### while 循环

#### C

```c
int ii = 0;

while (ii < 10)  //ANY value less than ten is true.
    printf("%d, ", ii++); // ii++ increments ii AFTER using its current value.
// => prints "0, 1, 2, 3, 4, 5, 6, 7, 8, 9, "

printf("\n");
```

#### Java

```java
int fooWhile = 0;

while(fooWhile < 100) {
    System.out.println(fooWhile);
    // Increment the counter
    // Iterated 100 times, fooWhile 0,1,2...99
    fooWhile++;
}
System.out.println("fooWhile Value: " + fooWhile);
```

#### JavaScript

```javascript
// As does `while`.
while (true) {
  // An infinite loop!
}
```

#### Python

```python
x = 0

while x < 4:
    print(x)
    x += 1  # Shorthand for x = x + 1
```

#### Rust

```rust
 while 1 == 1 {
    println!("The universe is operating normally.");
    // break statement gets out of the while loop.
    //  It avoids useless iterations.
    break
}
```

#### Go

```go
// Go 语言里面只有 for 循环，但是 for 循环可以不加范围

for { // Infinite loop.
    break    // Just kidding.
    continue // Unreached.
}
```

### 循环的本质

事实上，不管循环本身的写法和描述有什么改变，它的本质都是一种**逻辑判断**。也就是说，它们从根本上都是 `until` 循环的类型：

> 当某条件满足的时候跳转到循环结束的地方，不然就跳转回循环开始的地方

而基本所有的循环最后大概都会被编译成以下的样子，这叫做**汇编语言**，它是最接近计算机的思考方式的编程语言了。

```assembly
    mov eax, val1        ; 把变量 val1 放到 EAX 里面
beginwhile:
    cmp eax, val2        ; 比较 val1 和 val2
    jnl     endwhile     ; 如果 val1 不小于 val2，就跳到 endwhile 的地方
    inc    eax           ; val1++;
    dec    val2          ; val2--;
    jmp    beginwhile    ; 跳回到 beginwhile 的地方
endwhile:
    mov    val1, eax     ;保存 val1 的新值
```

## 递归

循环有一对「孪生兄弟」叫做**递归**。它们的作用都在于解决「重复」的事情。所不同的在于它们对于「重复」的部分的抽象描述不同

- 如果把要重复执行的指令放在一个「块」里面，称为**循环体**，并通过外部变量来调整每次循环执行的数据，就叫做**循环。**
- 如果把要重复执行的指令抽象成「函数」，并通过传参数的形式来调整每次执行的数据，就称作**递归**啦！

关于递归的详情可以看看来自 Helen 的视频讲解：

<iframe
    frameborder="no" framespacing="0"
    scrolling="no" allowfullscreen="true"
    loading="lazy" lazyload="1"
    src="//player.bilibili.com/player.html?aid=753456083&bvid=BV1ok4y1B7Cj&cid=197572573&page=1"
></iframe>

## 后面

好啦，这次并没有做题，要讲的内容就这么多啦。有时候换换心情和视野也是很重要的，希望这次的内容可以当做故事看看，了解一些更多的事情。下次见啦！

[1]: https://ed.ted.com/search?qs=think+like+a+coder
