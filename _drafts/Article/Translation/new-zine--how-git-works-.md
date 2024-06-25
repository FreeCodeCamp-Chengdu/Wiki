---
title: Julia Evans
date: 2024-06-03T09:45:11.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/04/25/new-zine--how-git-works-/
translator: ""
reviewer: ""
---

# New zine: How Git Works!

<!-- more -->

• [git][1] •

Hello! I’ve been writing about git on here nonstop for months, and the git zine is FINALLY done! It came out on Friday!

You can get it for $12 here: [https://wizardzines.com/zines/git][2], or get an [14-pack of all my zines here][3].

Here’s the cover:

[![](https://wizardzines.com/zines/git/cover-small.jpg)][4]

### the table of contents

Here’s the table of contents:

[![](https://wizardzines.com/zines/git/toc.png)][5]

### who is this zine for?

I wrote this zine for people who have been using git for years and are still afraid of it. As always – I think it sucks to be afraid of the tools that you use in your work every day! I want folks to feel confident using git.

My goals are:

-   To explain how some parts of git that initially seem scary (like “detached HEAD state”) are pretty straightforward to deal with once you understand what’s going on
-   To show some parts of git you probably _should_ be careful around. For example, the stash is one of the places in git where it’s easiest to lose your work in a way that’s incredibly annoying to recover form, and I avoid using it heavily because of that.
-   To clear up a few common misconceptions about how the core parts of git (like commits, branches, and merging) work

### what’s the difference between this and Oh Shit, Git!

You might be wondering – Julia! You already have a zine about git! What’s going on? [Oh Shit, Git!][6] is a set of tricks for fixing git messes. [“How Git Works”][7] explains how Git **actually** works.

Also, Oh Shit, Git! is the amazing [Katie Sylor Miller][8]’s [concept][9]: we made it into a zine because I was such a huge fan of her work on it.

I think they go really well together.

### what’s so confusing about git, anyway?

This zine was really hard for me to write because when I started writing it, I’d been using git pretty confidently for 10 years. I had no real memory of what it was _like_ to struggle with git.

But thanks to a huge amount of help from [Marie][10] as well as everyone who talked to me about git on Mastodon, eventually I was able to see that there are a lot of things about git that are counterintuitive, misleading, or just plain confusing. These include:

-   [confusing terminology][11] (for example “fast-forward”, “reference”, or “remote-tracking branch”)
-   misleading messages (for example how `Your branch is up to date with 'origin/main'` doesn’t necessary mean that your branch is up to date with the `main` branch on the origin)
-   uninformative output (for example how I _STILL_ can’t reliably figure out which code comes from which branch when I’m looking at a merge conflict)
-   a lack of guidance around handling diverged branches (for example how when you run `git pull` and your branch has diverged from the origin, it doesn’t give you great guidance how to handle the situation)
-   inconsistent behaviour (for example how git’s reflogs are almost always append-only, EXCEPT for the stash, where git will delete entries when you run `git stash drop`)

The more I heard from people how about how confusing they find git, the more it became clear that git really does not make it easy to figure out what its internal logic is just by using it.

### handling git’s weirdnesses becomes pretty routine

The previous section made git sound really bad, like “how can anyone possibly use this thing?“.

But my experience is that after I learned what git actually means by all of its weird error messages, dealing with it became pretty routine! I’ll see an `error: failed to push some refs to 'github.com:jvns/wizard-zines-site'`, realize “oh right, probably a coworker made some changes to `main` since I last ran `git pull`”, run `git pull --rebase` to incorporate their changes, and move on with my day. The whole thing takes about 10 seconds.

Or if I see a `You are in 'detached HEAD' state` warning, I’ll just make sure to run `git checkout mybranch` before continuing to write code. No big deal.

For me (and for a lot of folks I talk to about git!), dealing with git’s weird language can become so normal that you totally forget why anybody would even find it weird.

### a little bit of internals

One of my biggest questions when writing this zine was how much to focus on what’s in the `.git` directory. We ended up deciding to include a couple of pages about internals (“inside .git”, pages 14-15), but otherwise focus more on git’s _behaviour_ when you use it and why sometimes git behaves in unexpected ways.

This is partly because there are lots of great guides to git’s internals out there already ([1][12], [2][13]), and partly because I think even if you _have_ read one of these guides to git’s internals, it isn’t totally obvious how to connect that information to what you actually see in git’s user interface.

For example: it’s easy to find documentation about remotes in git – for example [this page][14] says:

> Remote-tracking branches \[…\] remind you where the branches in your remote repositories were the last time you connected to them.

But even if you’ve read that, you might not realize that the statement `Your branch is up to date with 'origin/main'"` in `git status` doesn’t necessarily mean that you’re actually up to date with the remote `main` branch.

So in general in the zine we focus on the behaviour you see in Git’s UI, and then explain how that relates to what’s happening internally in Git.

### the cheat sheet

The zine also comes with a free printable cheat sheet: (click to get a PDF version)

[![](https://wizardzines.com/images/cheat-sheet-smaller.png)][15]

### it comes with an HTML transcript!

The zine also comes with an HTML transcript, to (hopefully) make it easier to read on a screen reader! Our Operations Manager, Lee, transcribed all of the pages and wrote image descriptions. I’d love feedback about the experience of reading the zine on a screen reader if you try it.

### I really do love git

I’ve been pretty critical about git in this post, but I only write zines about technologies I love, and git is no exception.

Some reasons I love git:

-   it’s fast!
-   it’s backwards compatible! I learned how to use it 10 years ago and everything I learned then is still true
-   there’s tons of great free Git hosting available out there (GitHub! Gitlab! a million more!), so I can easily back up all my code
-   simple workflows are REALLY simple (if I’m working on a project on my own, I can just run `git commit -am 'whatever'` and `git push` over and over again and it works perfectly)
-   Almost every internal file in git is a pretty simple text file (or has a version which is a text file), which makes me feel like I can always understand exactly what’s going on under the hood if I want to.

I hope this zine helps some of you love it too.

### people who helped with this zine

I don’t make these zines by myself!

I worked with [Marie Claire LeBlanc Flanagan][16] every morning for 8 months to write clear explanations of git.

The cover is by Vladimir Kašiković, Gersande La Flèche did copy editing, James Coglan (of the great [Building Git][17]) did technical review, our Operations Manager Lee did the transcription as well as a million other things, my partner Kamal read the zine and told me which parts were off (as he always does), and I had a million great conversations with Marco Rogers about git.

And finally, I want to thank all the beta readers! There were 66 this time which is a record! They left hundreds of comments about what was confusing, what they learned, and which of my jokes were funny. It’s always hard to hear from beta readers that a page I thought made sense is actually extremely confusing, and fixing those problems before the final version makes the zine so much better.

### get the zine

Here are some links to get the zine again:

-   get [How Git Works][18]
-   get an [14-pack of all my zines here][19].

As always, you can get either a PDF version to print at home or a print version shipped to your house. The only caveat is print orders will ship in **July** – I need to wait for orders to come in to get an idea of how many I should print before sending it to the printer.

### thank you

As always: if you’ve bought zines in the past, thank you for all your support over the years. And thanks to all of you (1000+ people!!!) who have already bought the zine in the first 3 days. It’s already set a record for most zines sold in a single day and I’ve been really blown away.

[Notes on git's error messages][20]

[1]: /categories/git
[2]: https://wizardzines.com/zines/git
[3]: https://wizardzines.com/zines/all-the-zines/
[4]: https://wizardzines.com/zines/git
[5]: https://wizardzines.com/zines/git/toc.png
[6]: https://wizardzines.com/zines/oh-shit-git
[7]: https://wizardzines.com/zines/git/
[8]: https://sylormiller.com/
[9]: https://ohshitgit.com/
[10]: https://marieflanagan.com/
[11]: https://jvns.ca/blog/2023/11/01/confusing-git-terminology/
[12]: https://maryrosecook.com/blog/post/git-from-the-inside-out
[13]: https://shop.jcoglan.com/building-git/
[14]: https://git-scm.com/book/en/v2/Git-Branching-Remote-Branches
[15]: https://wizardzines.com/git-cheat-sheet.pdf
[16]: https://marieflanagan.com/
[17]: https://shop.jcoglan.com/building-git/
[18]: https://wizardzines.com/zines/git
[19]: https://wizardzines.com/zines/all-the-zines/
[20]: https://jvns.ca/blog/2024/04/10/notes-on-git-error-messages/ "Previous Post: Notes on git's error messages"