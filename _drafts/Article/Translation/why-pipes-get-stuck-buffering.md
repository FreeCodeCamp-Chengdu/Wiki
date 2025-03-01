---
title: 为什么管道有时会卡住：缓冲机制
date: 2024-11-29T08:23:31.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/
translator: "luojiyin"
reviewer: ""
---

这是一个困扰我多年的终端小问题，直到几周前我才真正理解它。假设你运行以下命令来监控日志文件中的某些特定输出：

```bash
tail -f /some/log/file | grep thing1 | grep thing2
```

如果日志文件中的新行添加得比较慢，结果就是……什么也没有！无论日志文件中是否有匹配项，都不会有任何输出。

我过去一直认为`管道有时会卡住，不显示输出，这很奇怪`，然后我会通过直接运行`grep thing1 /some/log/file | grep thing2` 来解决这个问题，这样确实有效。

所以，最近几个月我在深入研究终端时，终于搞清楚了这背后的原因。

### [原因：缓冲机制](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#why-this-happens-buffering)

`管道卡住`的原因在于，程序在将数据写入管道或文件之前通常会先进行缓冲。所以管道本身工作正常，问题在于程序根本没有将数据写入管道！

这是出于性能考虑：立即输出所有数据会使用更多的系统调用，因此更高效的做法是积累数据，直到有大约 8KB 的数据（或者程序退出时）才一次性写入管道。

在这个例子中：

```bash
tail -f /some/log/file | grep thing1 | grep thing2
```

问题在于 `grep thing1` 会保存所有匹配项，直到有 8KB 的数据才写入，而这可能永远不会发生。

### [程序在写入终端时不会缓冲](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#programs-don-t-buffer-when-writing-to-a-terminal)

让我感到困惑的部分是，`tail -f file | grep thing` 会完全正常工作，但当你添加第二个 `grep` 时，它就不工作了！原因是 `grep` 的缓冲行为取决于它是否在写入终端。

以下是 `grep`（以及许多其他程序）决定如何缓冲输出的方式：

*   使用 `isatty` 函数检查标准输出是否是终端
    *   如果是终端，则使用行缓冲（立即输出每一行）
    *   否则，使用`块缓冲`——只有在有至少 8KB 的数据时才输出

所以，如果 `grep` 直接写入终端，你会立即看到输出，但如果它写入管道，你就不会看到。

当然，缓冲区大小并不总是 8KB，这取决于具体实现。对于 `grep`，缓冲由 libc 处理，libc 的缓冲区大小由 `BUFSIZ` 变量定义。[这是 glibc 中的定义](https://github.com/bminor/glibc/blob/c69e8cccaff8f2d89cee43202623b33e6ef5d24a/libio/stdio.h#L100)。

（顺便说一句：`程序在写入终端时不使用 8KB 输出缓冲区`,并不是终端物理学的定律，程序完全可以在写入终端时使用 8KB 缓冲区，只是这样做会非常奇怪，我想不出有任何程序会这样做）

### [会缓冲的命令和不会缓冲的命令](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#commands-that-buffer-commands-that-don-t)

这种缓冲行为的一个烦人之处在于，你需要记住哪些命令在写入管道时会缓冲输出。

一些**不会**缓冲输出的命令：

*   tail
*   cat
*   tee

我认为几乎所有其他命令都会缓冲输出，尤其是那些你可能用于批处理的命令。以下是一些在写入管道时会缓冲输出的常见命令，以及禁用块缓冲的标志。

*   grep (`--line-buffered`)
*   sed (`-u`)
*   awk（有 `fflush()` 函数）
*   tcpdump (`-l`)
*   jq (`-u`)
*   tr (`-u`)
*   cut（无法禁用缓冲）

这些是我能想到的所有命令，许多 Unix 命令（如 `sort`）可能会或可能不会缓冲输出，但这并不重要，因为 `sort` 在完成接收输入之前无法做任何事情。

此外，我尽力测试了 Mac OS 和 GNU 版本的这些命令，但有很多变体，我可能犯了一些错误。

### [默认`print`语句会缓冲的编程语言](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#programming-languages-where-the-default-print-statement-buffers)

此外，以下是一些默认 `print` 语句在写入管道时会缓冲输出的编程语言，以及一些禁用缓冲的方法：

*   C（使用 `setvbuf` 禁用）
*   Python（使用 `python -u`，或 `PYTHONUNBUFFERED=1`，或 `sys.stdout.reconfigure(line_buffering=False)`，或 `print(x, flush=True)` 禁用）
*   Ruby（使用 `STDOUT.sync = true` 禁用）
*   Perl（使用 `$| = 1` 禁用）

我认为这些语言这样设计是为了在批处理时默认的 `print` 函数会更快。

此外，输出是否缓冲可能取决于你如何打印，例如在 C++ 中，`cout << "hello\n"` 在写入管道时会缓冲，但 `cout << "hello" << endl` 会刷新输出。

### [当你按下 `Ctrl-C` 时，缓冲区的内容会丢失](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#when-you-press-ctrl-c-on-a-pipe-the-contents-of-the-buffer-are-lost)

假设你运行以下命令作为一种监控 `example.com` 的 DNS 请求的临时方法，但你忘记给 tcpdump 传递 `-l` 参数：

```bash
sudo tcpdump -ni any port 53 | grep example.com
```

当你按下 `Ctrl-C` 时，会发生什么？在一个完美的世界里，我希望 `tcpdump` 刷新它的缓冲区，`grep` 会搜索 `example.com`，然后我会看到所有错过的输出。

但在现实世界中，所有程序都会被杀死，`tcpdump` 缓冲区中的输出会丢失。

我认为这个问题可能无法避免——我用 `strace` 研究了一下，发现 `grep` 会在 `tcpdump` 之前收到 `SIGINT`，所以即使 `tcpdump` 尝试刷新缓冲区，`grep` 也已经死了。

经过进一步研究，有一个变通方法：如果你找到 `tcpdump` 的 PID 并执行 `kill -TERM $PID`，那么 tcpdump 会刷新缓冲区，这样你就能看到输出。这有点麻烦，但我测试过，它似乎有效。

### [重定向到文件也会缓冲](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#redirecting-to-a-file-also-buffers)

不仅仅是管道，以下命令也会缓冲：

```bash
sudo tcpdump -ni any port 53 > output.txt
```

重定向到文件不会有 `Ctrl-C` 会完全破坏缓冲区内容的问题——根据我的经验，它通常会表现得像你期望的那样，缓冲区的内容会在程序退出前写入文件。我不完全确定这是否总是可靠的。

### [一些避免缓冲的方法](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#a-bunch-of-potential-ways-to-avoid-buffering)

好了，让我们来谈谈解决方案。假设你运行了以下命令：

```bash
tail -f /some/log/file | grep thing1 | grep thing2
```

我在 Mastodon 上询问了人们在实际中如何解决这个问题，有五种基本方法。以下是它们：

#### [方法 1：运行一个快速结束的程序](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#solution-1-run-a-program-that-finishes-quickly)

历史上，我的解决方案是完全避免`命令缓慢写入管道`的情况，而是运行一个会快速结束的程序，比如：

```
cat /some/log/file | grep thing1 | grep thing2 | tail
```

这与原始命令不同，但它确实意味着你可以避免思考这些奇怪的缓冲问题。

（你也可以直接运行 `grep thing1 /some/log/file`，但我通常更喜欢使用不必要的 `cat`）

#### [方法 2：记住 grep 的`行缓冲`标志](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#solution-2-remember-the-line-buffer-flag-to-grep)

你可以记住 grep 有一个避免缓冲的标志，并像这样传递它：

```bash
tail -f /some/log/file | grep --line-buffered thing1 | grep thing2
```

#### [方法 3：使用 awk](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering

有些人说，如果他们遇到多个 `grep` 的情况，他们会改用单个 `awk` 来重写命令，比如：

```
tail -f /some/log/file |  awk '/thing1/ && /thing2/'
```

或者你可以写一个更复杂的 `grep`，比如：

```
tail -f /some/log/file |  grep -E 'thing1.*thing2'
```

（`awk` 也会缓冲，所以为了让这个方法有效，你需要让 `awk` 成为管道中的最后一个命令）

#### [方法 4：使用 `stdbuf`](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#solution-4-use-stdbuf)

`stdbuf` 使用 `LD_PRELOAD` 来关闭 libc 的缓冲，你可以像这样使用它来关闭输出缓冲：

```bash
tail -f /some/log/file | stdbuf -o0 grep thing1 | grep thing2
```

像任何 `LD_PRELOAD` 解决方案一样，它有点不可靠——它不适用于静态二进制文件，我认为如果程序没有使用 libc 的缓冲，它也不会工作，而且在 Mac OS 上也不总是有效。Harry Marr 有一篇很好的 [How stdbuf works](https://hmarr.com/blog/how-stdbuf-works/) 文章。

#### [方法 5：使用 `unbuffer`](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#solution-5-use-unbuffer)

`unbuffer program` 会强制程序的输出为 TTY，这意味着它会像在 TTY 上一样行为（减少缓冲，彩色输出等）。你可以在这个例子中这样使用它：

```bash
tail -f /some/log/file | unbuffer grep thing1 | grep thing2
```

与 `stdbuf` 不同，它总是有效，尽管可能会有一些不想要的副作用，例如 `grep thing1` 也会对匹配项进行着色。

如果你想安装 `unbuffer`，它在 `expect` 包中。

### [这就是我知道的所有解决方案！](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#that-s-all-the-solutions-i-know-about)

很难说哪个是`最好的`，我个人最有可能使用 `unbuffer`，因为我知道它总是有效。

如果我了解到更多的解决方案，我会尝试将它们添加到这篇文章中。

### [我不太确定这种情况有多常见](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#i-m-not-really-sure-how-often-this-comes-up)

我认为这种情况对我来说并不常见，通常如果我使用管道，大量数据会非常快速地写入，由管道中的所有内容处理，然后所有内容退出。我现在能想到的唯一例子是：

*   tcpdump
*   `tail -f`
*   以其他方式监控日志文件，比如使用 `kubectl logs`
*   缓慢计算的输出

### [如果有一个环境变量来禁用缓冲会怎样？](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#what-if-there-were-an-environment-variable-to-disable-buffering)

我认为如果有一个标准的环境变量来关闭缓冲，比如 Python 中的 `PYTHONUNBUFFERED`，那会很酷。这个想法来自 Mark Dominus 在 2018 年的[几篇](https://blog.plover.com/Unix/stdio-buffering.html) [博客文章](https://blog.plover.com/Unix/stdio-buffering-2.html)。也许可以像 [NO_COLOR](https://no-color.org/) 一样叫 `NO_BUFFER`？

设计似乎很难做到完美；Mark 指出 NETBSD 有[名为 `STDBUF`、`STDBUF1` 等的环境变量](https://man.netbsd.org/setbuf.3)，它们让你对缓冲有很多控制，但我想大多数开发者不想实现许多不同的环境变量来处理一个相对较小的边缘情况。

我也很好奇是否有任何程序会自动在一段时间后（比如 1 秒）刷新它们的输出缓冲区。理论上这似乎不错，但我想不出有任何程序这样做，所以我想可能有一些缺点。

### [我遗漏的内容](http://jvns.ca/blog/2024/11/29/why-pipes-get-stuck-buffering/#stuff-i-left-out)

由于这些文章最近变得很长，而且真的有人想读 3000 字关于缓冲的内容吗？所以我在这篇文章中没有谈到以下内容：

*   行缓冲和完全无缓冲输出的区别
*   缓冲到 stderr 与缓冲到 stdout 的不同
*   这篇文章只讨论了**程序内部**的缓冲，你的操作系统的 TTY 驱动程序有时也会做一些缓冲
*   除了`你正在写入管道`之外，你可能需要刷新输出的其他原因