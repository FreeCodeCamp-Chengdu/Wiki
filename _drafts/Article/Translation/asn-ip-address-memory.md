---
title: 使用更少的内存来查找 Mess With DNS 中的 IP 地址
date: 2024-10-27T07:47:04.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/
translator: ""
reviewer: ""
---

在过去的 3 年左右时间里，我一直遇到一个问题:[Mess With DNS][1] 会周期性地耗尽内存并被 OOM killer 终止。

这对我来说并不是一个大问题:通常它只会在重启时停机几分钟,而且最多每天发生一次,所以我一直在忽略它。但是上周它开始真的造成问题了,所以我决定研究一下。

这是一个曲折的过程,我学到了很多东西,以下是目录:

-   [大约有 100MB 的可用内存][2]
-   [问题:OOM killer 终止备份脚本][3]
-   [尝试 1:使用 SQLite][4]
    -   [问题:如何存储 IPv6 地址][5]
    -   [问题:速度慢了 500 倍][6]
    -   [是时候用 EXPLAIN QUERY PLAN 了][7]
-   [尝试 2:使用字典树][8]
    -   [关于内存分析的一些笔记][9]
-   [尝试 3:让数组使用更少的内存][10]
    -   [想法 3.1:对 Name 和 Country 进行去重][11]
    -   [ASN 有多大?][12]
    -   [想法 3.2:使用 netip.Addr 替代 net.IP][13]
    -   [结果:节省了 70MB 内存!][14]

### 大约有 100MB 的可用内存

我在一个大约有 465MB RAM 的虚拟机上运行 Mess With DNS,根据 `ps aux` (RSS 列)显示内存分配如下:

-   PowerDNS 使用 100MB
-   Mess With DNS 使用 200MB
-   [hallpass][15] 使用 40MB

这样还剩下大约 110MB 的可用内存。

之前我设置了 [GOMEMLIMIT][16] 为 250MB,以确保当 Mess With DNS 使用超过 250MB 内存时垃圾收集器会运行,我觉得这有帮助但并没有解决所有问题。

### 问题:OOM killer 终止备份脚本

几周前我第一次开始使用 [restic][17] 备份 Mess With DNS 的数据库。

这个方案基本可行,但由于 Mess With DNS 没有太多额外的内存,我认为 `restic` 有时需要的内存超过了系统可用内存,所以备份脚本有时会被 OOM killer 终止。

这带来了两个问题:

1. 备份可能会被损坏
2. 更重要的是,restic 在运行时会加锁,如果我想让备份继续工作就必须手动解锁。手动操作是我在所有网络服务中最想避免的事情(谁有时间做这个!)所以我真的想解决这个问题。

这可能有不止一种解决方案,但我决定尝试让 Mess With DNS 使用更少的内存,这样系统就有更多可用内存,主要是因为这看起来是一个有趣的问题。

### 内存使用在哪里:IP 地址

我之前对 Mess With DNS 做过很多次内存分析,所以我很清楚什么占用了最多内存:IP 地址。

在启动时,Mess With DNS 会将这个[可以查找每个 IP 地址的 ASN 的数据库][18]加载到内存中,这样当它收到 DNS 查询时,就可以获取源 IP 地址(如 `74.125.16.248`)并告诉你这个 IP 地址属于 `GOOGLE`。

仅这个数据库就使用了大约 117MB 内存,而一个简单的 `du` 命令告诉我这太多了 - 原始文本文件只有 37MB!

```bash
$ du -sh *.tsv
26M	ip2asn-v4.tsv
11M	ip2asn-v6.tsv
```

最初的实现方式是我有一个这样的数组:

```golang
type IPRange struct {
	StartIP net.IP
	EndIP   net.IP
	Num     int
	Name    string
	Country string
}
```

然后我用二分查找来确定是否有任何范围包含我要查找的 IP。这是最简单的方法,而且超级快,我的机器每秒可以进行约 900 万次查找。

### 尝试 1:使用 SQLite

我最近一直在使用 SQLite,所以我的第一个想法是 - 也许我可以把所有数据存储在 SQLite 数据库的磁盘上,给表添加索引,这样就能使用更少的内存。

所以我:

-   写了一个使用 [sqlite-utils][19] 的简单 Python 脚本来将 TSV 文件导入到 SQLite 数据库
-   调整了代码以从数据库中查询

这确实解决了最初的内存目标(在 GC 之后几乎不使用任何内存,因为表在磁盘上!),虽然我不确定如果我们需要同时进行大量查询时这个解决方案会造成多少 GC 压力。我做了一个快速的内存分析,发现每次查找大约分配 1KB 内存。
让我们来谈谈使用 SQLite 遇到的问题。

### 问题:如何存储 IPv6 地址

SQLite 不支持大整数,而 IPv6 地址是 128 位的,所以我决定将它们存储为文本。我认为 BLOB 可能会更好,我最初以为 BLOB 不能比较,但 [sqlite 文档][20] 说可以。
我最终使用了这样的模式:

我最终使用了这样的表结构:

```sql
CREATE TABLE ipv4_ranges (
   start_ip INTEGER NOT NULL,
   end_ip INTEGER NOT NULL,
   asn INTEGER NOT NULL,
   country TEXT NOT NULL,
   name TEXT NOT NULL
);
CREATE TABLE ipv6_ranges (
   start_ip TEXT NOT NULL,
   end_ip TEXT NOT NULL,
   asn INTEGER,
   country TEXT,
   name TEXT
);
CREATE INDEX idx_ipv4_ranges_start_ip ON ipv4_ranges (start_ip);
CREATE INDEX idx_ipv6_ranges_start_ip ON ipv6_ranges (start_ip);
CREATE INDEX idx_ipv4_ranges_end_ip ON ipv4_ranges (end_ip);
CREATE INDEX idx_ipv6_ranges_end_ip ON ipv6_ranges (end_ip);
```

另外我了解到 Python 有一个 ipaddress 模块,所以我可以使用 ipaddress.ip_address(s).exploded 来确保 IPv6 地址被展开,这样字符串比较就能正确工作。

### 问题:速度慢了 500 倍

我做了一个快速的微基准测试,像这样。它打印出每秒可以查找 17,000 个 IPv6 地址,对于 IPv4 地址也是如此。

这相当令人沮丧——每秒查找 17,000 个地址的能力还算可以接受(Mess With DNS 不会收到太多流量),但与原来的二分查找代码相比,原来的代码每秒可以查找 900 万个地址。

```golang
	ips := []net.IP{}
	count := 20000
	for i := 0; i < count; i++ {
		// create a random IPv6 address
		bytes := randomBytes()
		ip := net.IP(bytes[:])
		ips = append(ips, ip)
	}
	now := time.Now()
	success := 0
	for _, ip := range ips {
		_, err := ranges.FindASN(ip)
		if err == nil {
			success++
		}
	}
	fmt.Println(success)
	elapsed := time.Since(now)
	fmt.Println("number per second", float64(count)/elapsed.Seconds())
```

### 是时候用 EXPLAIN QUERY PLAN 了

我从来没有真正在 sqlite 中做过 EXPLAIN,所以我认为这是一个很好的机会来看看查询计划在做什么。

```sql
sqlite> explain query plan select * from ipv6_ranges where '2607:f8b0:4006:0824:0000:0000:0000:200e' BETWEEN start_ip and end_ip;
QUERY PLAN
`--SEARCH ipv6_ranges USING INDEX idx_ipv6_ranges_end_ip (end_ip>?)
```

看起来它只使用了 `end_ip` 索引,而不是 `start_ip` 索引,所以也许它比二分查找慢是有道理的。

我试图找出是否有办法让 SQLite 使用两个索引,但我找不到,也许它无论如何都知道得更好。

这时我放弃了 SQLite 解决方案,我不喜欢它更慢而且比简单的二分查找复杂得多。我觉得我更愿意保持一个更接近二分查找的方案。
我尝试了一些方法来让 SQLite 使用两个索引,但都没有成功:

-   使用复合索引而不是两个单独的索引
-   运行 `ANALYZE`
-   使用 `INTERSECT` 来交叉 `start_ip < ?` 和 `? < end_ip` 的结果。这确实让它使用了两个索引,但查询也明显慢了 1000 倍,可能是因为它需要在内存中创建两个子查询的结果并交叉它们。

### 尝试 2:使用字典树

我的下一个想法是使用[字典树][21],因为我有一个模糊的想法,也许字典树会使用更少的内存。我找到了一个叫 [ipaddress-go][22] 的库,它可以使用字典树来查找 IP 地址。

我尝试使用它([这里是代码][23]),但我觉得我可能做错了什么,因为与我简单的数组+二分查找相比:

-   它使用了更多的内存(仅存储 IPv4 地址就用了 800MB)
-   查找速度慢得多(每秒只能进行 10 万次查找,而不是 900 万次)

我不太确定这里出了什么问题,但我放弃了这个方法,决定尝试让我的数组使用更少的内存,继续使用简单的二分查找。

### 关于内存分析的一些笔记

关于内存分析,我学到的一件事是可以使用 `runtime` 包来查看程序当前分配的内存量。这就是我如何得到这篇文章中所有内存数据的方法。这是代码:

```golang
func memusage() {
	runtime.GC()
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("Alloc = %v MiB\n", m.Alloc/1024/1024)
	// write mem.prof
	f, err := os.Create("mem.prof")
	if err != nil {
		log.Fatal(err)
	}
	pprof.WriteHeapProfile(f)
	f.Close()
}
```

另外我了解到,如果使用 `pprof` 分析堆配置文件,有两种方式可以分析:你可以传递 `--alloc-space` 或 `--inuse-space` 给 `go tool pprof`。我不知道我之前怎么没意识到这一点,但 `alloc-space` 会告诉你所有被分配的内存,而 `inuse-space` 只会包含当前正在使用的内存。

总之我运行了很多次 `go tool pprof -pdf --inuse_space mem.prof > mem.pdf`。每次使用 pprof 时,我都会参考[我自己写的 pprof 入门指南][24],这可能是我写过的最常用的博客文章。我应该在里面加入 `--alloc-space` 和 `--inuse-space` 的说明。

### 尝试 3:让数组使用更少的内存

我之前是这样存储 ip2asn 条目的:

```golang
type IPRange struct {
	StartIP net.IP
	EndIP   net.IP
	Num     int
	Name    string
	Country string
}
```

我有 3 个改进的想法:

1. `Name` 和 `Country` 有很多重复,因为很多 IP 范围属于同一个 ASN
2. `net.IP` 底层是一个 `[]byte`,这感觉涉及了一个不必要的指针,是否有办法将它内联到结构体中?
3. 也许我不需要同时存储起始 IP 和结束 IP,因为范围通常是连续的,所以也许我可以重新安排只存储起始 IP

### 想法 3.1:对 Name 和 Country 进行去重

我想到可以存储 ASN 信息到一个数组中,然后在 `IPRange` 结构体中只存储数组中的索引。这是结构体代码:

```golang
type IPRange struct {
	StartIP netip.Addr
	EndIP   netip.Addr
	ASN     uint32
	Idx     uint32
}

type ASNInfo struct {
	Country string
	Name    string
}

type ASNPool struct {
	asns   []ASNInfo
	lookup map[ASNInfo]uint32
}
```

这确实有效!它将内存使用量从 117MB 减少到 65MB – 节省了 50MB。我对此感到满意。

[这里是这部分的全部代码][25]。

### ASN 有多大?

顺便说一句 – 我存储 ASN 的 `uint32` 是对的吗?我看了 ip2asn 文件,最大的一个似乎是 401307,尽管也有几行说 `4294901931` 这个数字更大,但也只是在 `uint32` 的范围之内。所以我绝对可以使用 `uint32`。

```
59.101.179.0	59.101.179.255	4294901931	Unknown	AS4294901931
```

### 想法 3.2:使用 `netip.Addr` 替代 `net.IP`

事实证明,我不只是一个人认为 `net.IP` 使用了不必要的内存 – 在 2021 年,Tailscale 发布了一个新的 IP 地址库,解决了这个问题和许多其他问题。[他们写了一篇很棒的博客文章][26]。

我惊喜地发现这个新的 IP 地址库不仅存在,而且确实是我想要的,而且现在也在 Go 标准库中作为 [netip.Addr][27]。切换到 `netip.Addr` 非常简单,又节省了 20MB 内存,将内存使用量减少到 46MB。

我没有尝试我的第三个想法(从结构体中删除结束 IP),因为我已经在一个周六早上编程了足够长的时间,我对我的进展感到满意。

每次我想到“嘿,我不喜欢这个,一定有更好的方法”,然后立即发现有人已经做了我想要的东西,比我思考得更多,实现得更好。这总是如此美妙的感觉。

### 现实生活中的调试过程更混乱

尽管我试图以一种简单线性的方式解释这一点“我尝试了 X,然后我尝试了 Y,然后我尝试了 Z”,但这有点不真实 – 我总是试图将我的实际调试过程(完全混乱)变得更有条理和更容易理解,因为现实情况写下来太烦人了。更像是:

-   尝试 SQLite
-   尝试字典树
-   再次猜测我关于 SQLite 的结论,再次查看结果
-   wait what about indexes
-   非常非常晚地意识到我可以使用 `runtime` 来检查每个东西使用了多少内存,开始这样做
-   再次查看字典树,也许我误解了一切
-   放弃并回到二分查找

### 使用 512MB 内存的注意事项

有人问我为什么不直接给虚拟机更多的内存。我完全可以负担得起一台有 1GB 内存的虚拟机,但我感觉 512MB 真的应该足够了(而且真的 256MB 应该也足够了!)所以我宁愿待在这个限制内。这有点像一个有趣的谜题。

人们有很多我没想到的好主意。如果我有一天想再有一个 Fun Performance Day,我会把它们记录为灵感。

-   尝试 Go 的 [unique][28] 包来存储 `ASNPool`。有人尝试了这个,但它使用了更多的内存,可能是因为 Go 的指针是 64 位的
-   尝试使用 `GOARCH=386` 来使用 32 位指针来节省空间(可能与使用 `unique` 一起!)
-   应该可以将所有 IPv6 地址存储在 64 位中,因为只有地址的前 64 位是公共的
-   [插值搜索][29] 可能比二分查找更快,因为 IP 地址是数字的
-   尝试使用 MaxMind db 格式与 [mmdbwriter][30] 或 [mmdbctl][31]
-   Tailscale 的 [art][32] 路由表包

### 结果:节省了 70MB 的内存!

我部署了新版本,现在 Mess With DNS 使用的内存更少了! 耶!

其他一些注意事项:

-   查找速度稍微慢了一点 – 在我的微基准测试中,它们从每秒 900 万次查找下降到每秒 600 万次查找,可能是因为我添加了一些间接寻址。使用更少的内存和一点更多的 CPU 似乎是一个很好的权衡。
-   它仍然比原始文本文件使用更多的内存(46MB vs 37MB),我猜指针也占用空间,这是可以接受的。

我确实不确定这会解决我所有的内存问题,可能不会! 但我很开心,我学到了一些关于 SQLite 的知识,我还是不知道该怎么看字典树,但它让我更爱二分查找了。

[1]: https://messwithdns.net/
[2]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#there-s-about-100mb-of-memory-available
[3]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#the-problem-oom-killing-the-backup-script
[4]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-1-use-sqlite
[5]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#problem-how-to-store-ipv6-addresses
[6]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#problem-it-s-500x-slower
[7]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#time-for-explain-query-plan
[8]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-2-use-a-trie
[9]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#some-notes-on-memory-profiling
[10]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-3-make-my-array-use-less-memory
[11]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#idea-3-1-deduplicate-the-name-and-country
[12]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#how-big-are-asns
[13]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#idea-3-2-use-netip-addr-instead-of-net-ip
[14]: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#the-result-saved-70mb-of-memory
[15]: https://fly.io/blog/ssh-and-user-mode-ip-wireguard/
[16]: https://tip.golang.org/doc/gc-guide
[17]: https://jvns.ca/til/restic-for-backing-up-sqlite-dbs/
[18]: https://iptoasn.com/
[19]: https://sqlite-utils.datasette.io/en/stable/
[20]: https://www.sqlite.org/datatype3.html#sort_order
[21]: https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014
[22]: https://github.com/seancfoley/ipaddress-go
[23]: https://gist.github.com/jvns/3ce617796b22127017590ac62c57fddd
[24]: https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/
[25]: https://github.com/jvns/mess-with-dns/blob/94f77b4bb1597b5e2a6768e33bd6c285919aa1bf/api/streamer/ip2asn/ip2asn.go#L18-L54
[26]: https://tailscale.com/blog/netaddr-new-ip-type-for-go
[27]: https://pkg.go.dev/net/netip#Addr
[28]: https://pkg.go.dev/unique
[29]: https://en.m.wikipedia.org/wiki/Interpolation_search
[30]: https://github.com/maxmind/mmdbwriter
[31]: https://github.com/ipinfo/mmdbctl
[32]: https://github.com/tailscale/art
