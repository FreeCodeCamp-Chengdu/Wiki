---
title: Julia Evans
date: 2024-10-27T07:47:04.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/
translator: ""
reviewer: ""
---

I’ve been having problems for the last 3 years or so where [Mess With DNS](https://messwithdns.net/) periodically runs out of memory and gets OOM killed.

This hasn’t been a big priority for me: usually it just goes down for a few minutes while it restarts, and it only happens once a day at most, so I’ve just been ignoring. But last week it started actually causing a problem so I decided to look into it.

This was kind of winding road where I learned a lot so here’s a table of contents:

*   [there’s about 100MB of memory available](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#there-s-about-100mb-of-memory-available)
*   [the problem: OOM killing the backup script](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#the-problem-oom-killing-the-backup-script)
*   [attempt 1: use SQLite](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-1-use-sqlite)
    *   [problem: how to store IPv6 addresses](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#problem-how-to-store-ipv6-addresses)
    *   [problem: it’s 500x slower](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#problem-it-s-500x-slower)
    *   [time for EXPLAIN QUERY PLAN](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#time-for-explain-query-plan)
*   [attempt 2: use a trie](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-2-use-a-trie)
    *   [some notes on memory profiling](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#some-notes-on-memory-profiling)
*   [attempt 3: make my array use less memory](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#attempt-3-make-my-array-use-less-memory)
    *   [idea 3.1: deduplicate the Name and Country](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#idea-3-1-deduplicate-the-name-and-country)
    *   [how big are ASNs?](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#how-big-are-asns)
    *   [idea 3.2: use netip.Addr instead of net.IP](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#idea-3-2-use-netip-addr-instead-of-net-ip)
    *   [the result: saved 70MB of memory!](https://jvns.ca/blog/2024/10/27/asn-ip-address-memory/#the-result-saved-70mb-of-memory)

### there’s about 100MB of memory available

I run Mess With DNS on a VM without about 465MB of RAM, which according to `ps aux` (the `RSS` column) is split up something like:

*   100MB for PowerDNS
*   200MB for Mess With DNS
*   40MB for [hallpass](https://fly.io/blog/ssh-and-user-mode-ip-wireguard/)

That leaves about 110MB of memory free.

A while back I set [GOMEMLIMIT](https://tip.golang.org/doc/gc-guide) to 250MB to try to make sure the garbage collector ran if Mess With DNS used more than 250MB of memory, and I think this helped but it didn’t solve everything.

### the problem: OOM killing the backup script

A few weeks ago I started backing up Mess With DNS’s database for the first time [using restic](https://jvns.ca/til/restic-for-backing-up-sqlite-dbs/).

This has been working okay, but since Mess With DNS operates without much extra memory I think `restic` sometimes needed more memory than was available on the system, and so the backup script sometimes got OOM killed.

This was a problem because

1.  backups might be corrupted sometimes
2.  more importantly, restic takes out a lock when it runs, and so I’d have to manually do an unlock if I wanted the backups to continue working. Doing manual work like this is the #1 thing I try to avoid with all my web services (who has time for that!) so I really wanted to do something about it.

There’s probably more than one solution to this, but I decided to try to make Mess With DNS use less memory so that there was more available memory on the system, mostly because it seemed like a fun problem to try to solve.

### what’s using memory: IP addresses

I’d run a memory profile of Mess With DNS a bunch of times in the past, so I knew exactly what was using most of Mess With DNS’s memory: IP addresses.

When it starts, Mess With DNS loads this [database where you can look up the ASN of every IP address](https://iptoasn.com/) into memory, so that when it receives a DNS query it can take the source IP address like `74.125.16.248` and tell you that IP address belongs to `GOOGLE`.

This database by itself used about 117MB of memory, and a simple `du` told me that was too much – the original text files were only 37MB!

```
$ du -sh *.tsv
26M	ip2asn-v4.tsv
11M	ip2asn-v6.tsv
```

The way it worked originally is that I had an array of these:

```
type IPRange struct {
	StartIP net.IP
	EndIP   net.IP
	Num     int
	Name    string
	Country string
}
```

and I searched through it with a binary search to figure out if any of the ranges contained the IP I was looking for. Basically the simplest possible thing and it’s super fast, my machine can do about 9 million lookups per second.

### attempt 1: use SQLite

I’ve been using SQLite recently, so my first thought was – maybe I can store all of this data on disk in an SQLite database, give the tables an index, and that’ll use less memory.

So I:

*   wrote a quick Python script using [sqlite-utils](https://sqlite-utils.datasette.io/en/stable/) to import the TSV files into an SQLite database
*   adjusted my code to select from the database instead

This did solve the initial memory goal (after a GC it now hardly used any memory at all because the table was on disk!), though I’m not sure how much GC churn this solution would cause if we needed to do a lot of queries at once. I did a quick memory profile and it seemed to allocate about 1KB of memory per lookup.

Let’s talk about the issues I ran into with using SQLite though.

### problem: how to store IPv6 addresses

SQLite doesn’t have support for big integers and IPv6 addresses are 128 bits, so I decided to store them as text. I think `BLOB` might have been better, I originally thought `BLOB`s couldn’t be compared but the [sqlite docs](https://www.sqlite.org/datatype3.html#sort_order) say they can.

I ended up with this schema:

```
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

Also I learned that Python has an `ipaddress` module, so I could use `ipaddress.ip_address(s).exploded` to make sure that the IPv6 addresses were expanded so that a string comparison would compare them properly.

### problem: it’s 500x slower

I ran a quick microbenchmark, something like this. It printed out that it could look up 17,000 IPv6 addresses per second, and similarly for IPv4 addresses.

This was pretty discouraging – being able to look up 17k addresses per section is kind of fine (Mess With DNS does not get a lot of traffic), but I compared it to the original binary search code and the original code could do 9 million per second.

```
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

### time for EXPLAIN QUERY PLAN

I’d never really done an EXPLAIN in sqlite, so I thought it would be a fun opportunity to see what the query plan was doing.

```
sqlite> explain query plan select * from ipv6_ranges where '2607:f8b0:4006:0824:0000:0000:0000:200e' BETWEEN start_ip and end_ip;
QUERY PLAN
`--SEARCH ipv6_ranges USING INDEX idx_ipv6_ranges_end_ip (end_ip>?)
```

It looks like it’s just using the `end_ip` index and not the `start_ip` index, so maybe it makes sense that it’s slower than the binary search.

I tried to figure out if there was a way to make SQLite use both indexes, but I couldn’t find one and maybe it knows best anyway.

At this point I gave up on the SQLite solution, I didn’t love that it was slower and also it’s a lot more complex than just doing a binary search. I felt like I’d rather keep something much more similar to the binary search.

A few things I tried with SQLite that did not cause it to use both indexes:

*   using a compound index instead of two separate indexes
*   running `ANALYZE`
*   using `INTERSECT` to intersect the results of `start_ip < ?` and `? < end_ip`. This did make it use both indexes, but it also seemed to make the query literally 1000x slower, probably because it needed to create the results of both subqueries in memory and intersect them.

### attempt 2: use a trie

My next idea was to use a [trie](https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014), because I had some vague idea that maybe a trie would use less memory, and I found this library called [ipaddress-go](https://github.com/seancfoley/ipaddress-go) that lets you look up IP addresses using a trie.

I tried using it [here’s the code](https://gist.github.com/jvns/3ce617796b22127017590ac62c57fddd), but I think I was doing something wildly wrong because, compared to my naive array + binary search:

*   it used WAY more memory (800MB to store just the IPv4 addresses)
*   it was a lot slower to do the lookups (it could do only 100K/second instead of 9 million/second)

I’m not really sure what went wrong here but I gave up on this approach and decided to just try to make my array use less memory and stick to a simple binary search.

### some notes on memory profiling

One thing I learned about memory profiling is that you can use `runtime` package to see how much memory is currently allocated in the program. That’s how I got all the memory numbers in this post. Here’s the code:

```
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

Also I learned that if you use `pprof` to analyze a heap profile there are two ways to analyze it: you can pass either `--alloc-space` or `--inuse-space` to `go tool pprof`. I don’t know how I didn’t realize this before but `alloc-space` will tell you about everything that was allocated, and `inuse-space` will just include memory that’s currently in use.

Anyway I ran `go tool pprof -pdf --inuse_space mem.prof > mem.pdf` a lot. Also every time I use pprof I find myself referring to [my own intro to pprof](https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/), it’s probably the blog post I wrote that I use the most often. I should add `--alloc-space` and `--inuse-space` to it.

### attempt 3: make my array use less memory

I was storing my ip2asn entries like this:

```
type IPRange struct {
	StartIP net.IP
	EndIP   net.IP
	Num     int
	Name    string
	Country string
}
```

I had 3 ideas for ways to improve this:

1.  There was a lot of repetition of `Name` and the `Country`, because a lot of IP ranges belong to the same ASN
2.  `net.IP` is an `[]byte` under the hood, which felt like it involved an unnecessary pointer, was there a way to inline it into the struct?
3.  Maybe I didn’t need both the start IP and the end IP, often the ranges were consecutive so maybe I could rearrange things so that I only had the start IP

### idea 3.1: deduplicate the Name and Country

I figured I could store the ASN info in an array, and then just store the index into the array in my `IPRange` struct. Here are the structs so you can see what I mean:

```
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

This worked! It brought memory usage from 117MB to 65MB – a 50MB savings. I felt good about this.

[Here’s all of the code for that part](https://github.com/jvns/mess-with-dns/blob/94f77b4bb1597b5e2a6768e33bd6c285919aa1bf/api/streamer/ip2asn/ip2asn.go#L18-L54).

### how big are ASNs?

As an aside – I’m storing the ASN in a `uint32`, is that right? I looked in the ip2asn file and the biggest one seems to be 401307, though there are a few lines that say `4294901931` which is much bigger, but also are just inside the range of a uint32. So I can definitely use a `uint32`.

```
59.101.179.0	59.101.179.255	4294901931	Unknown	AS4294901931
```

### idea 3.2: use `netip.Addr` instead of `net.IP`

It turns out that I’m not the only one who felt that `net.IP` was using an unnecessary amount of memory – in 2021 the folks at Tailscale released a new IP address library for Go which solves this and many other issues. [They wrote a great blog post about it](https://tailscale.com/blog/netaddr-new-ip-type-for-go).

I discovered (to my delight) that not only does this new IP address library exist and do exactly what I want, it’s also now in the Go standard library as [netip.Addr](https://pkg.go.dev/net/netip#Addr). Switching to `netip.Addr` was very easy and saved another 20MB of memory, bringing us to 46MB.

I didn’t try my third idea (remove the end IP from the struct) because I’d already been programming for long enough on a Saturday morning and I was happy with my progress.

It’s always such a great feeling when I think “hey, I don’t like this, there must be a better way” and then immediately discover that someone has already made the exact thing I want, thought about it a lot more than me, and implemented it much better than I would have.

### all of this was messier in real life

Even though I tried to explain this in a simple linear way “I tried X, then I tried Y, then I tried Z”, that’s kind of a lie – I always try to take my actual debugging process (total chaos) and make it seem more linear and understandable because the reality is just too annoying to write down. It’s more like:

*   try sqlite
*   try a trie
*   second guess everything that I concluded about sqlite, go back and look at the results again
*   wait what about indexes
*   very very belatedly realize that I can use `runtime` to check how much memory everything is using, start doing that
*   look at the trie again, maybe I misunderstood everything
*   give up and go back to binary search
*   look at all of the numbers for tries/sqlite again to make sure I didn’t misunderstand

### A note on using 512MB of memory

Someone asked why I don’t just give the VM more memory. I could very easily afford to pay for a VM with 1GB of memory, but I feel like 512MB really _should_ be enough (and really that 256MB should be enough!) so I’d rather stay inside that constraint. It’s kind of a fun puzzle.

Folks had a lot of good ideas I hadn’t thought of. Recording them as inspiration if I feel like having another Fun Performance Day at some point.

*   Try Go’s [unique](https://pkg.go.dev/unique) package for the `ASNPool`. Someone tried this and it uses more memory, probably because Go’s pointers are 64 bits
*   Try compiling with `GOARCH=386` to use 32-bit pointers to sace space (maybe in combination with using `unique`!)
*   It should be possible to store all of the IPv6 addresses in just 64 bits, because only the first 64 bits of the address are public
*   [Interpolation search](https://en.m.wikipedia.org/wiki/Interpolation_search) might be faster than binary search since IP addresses are numeric
*   Try the MaxMind db format with [mmdbwriter](https://github.com/maxmind/mmdbwriter) or [mmdbctl](https://github.com/ipinfo/mmdbctl)
*   Tailscale’s [art](https://github.com/tailscale/art) routing table package

### the result: saved 70MB of memory!

I deployed the new version and now Mess With DNS is using less memory! Hooray!

A few other notes:

*   lookups are a little slower – in my microbenchmark they went from 9 million lookups/second to 6 million, maybe because I added a little indirection. Using less memory and a little more CPU seemed like a good tradeoff though.
*   it’s still using more memory than the raw text files do (46MB vs 37MB), I guess pointers take up space and that’s okay.

I’m honestly not sure if this will solve all my memory problems, probably not! But I had fun, I learned a few things about SQLite, I still don’t know what to think about tries, and it made me love binary search even more than I already did.