---
title: Julia Evans
date: 2025-02-05T16:57:00.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2025/02/05/some-terminal-frustrations/
translator: ""
reviewer: ""
---

A few weeks ago I ran a terminal survey (you can [read the results here][1]) and at the end I asked:

> What’s the most frustrating thing about using the terminal for you?

1600 people answered, and I decided to spend a few days categorizing all the responses. Along the way I learned that classifying qualitative data is not easy but I gave it my best shot. I ended up building a custom [tool][2] to make it faster to categorize everything.

As with all of my surveys the methodology isn’t particularly scientific. I just posted the survey to Mastodon and Twitter, ran it for a couple of days, and got answers from whoever happened to see it and felt like responding.

Here are the top categories of frustrations!

I think it’s worth keeping in mind while reading these comments that

*   40% of people answering this survey have been using the terminal for **21+ years**
*   95% of people answering the survey have been using the terminal for at least 4 years

These comments aren’t coming from total beginners.

Here are the categories of frustrations! The number in brackets is the number of people with that frustration. I’m mostly writing this up for myself because I’m trying to write a zine about the terminal and I wanted to get a sense for what people are having trouble with.

### [remembering syntax (115)][3]

People talked about struggles remembering:

*   the syntax for CLI tools like awk, jq, sed, etc
*   the syntax for redirects
*   keyboard shortcuts for tmux, text editing, etc

One example comment:

> There are just so many little “trivia” details to remember for full functionality. Even after all these years I’ll sometimes forget where it’s 2 or 1 for stderr, or forget which is which for `>` and `>>`.

### [switching terminals is hard (91)][4]

People talked about struggling with switching systems (for example home/work computer or when SSHing) and running into:

*   OS differences in keyboard shortcuts (like Linux vs Mac)
*   systems which don’t have their preferred text editor (“no vim” or “only vim”)
*   different versions of the same command (like Mac OS grep vs GNU grep)
*   no tab completion
*   a shell they aren’t used to (“the subtle differences between zsh and bash”)

as well as differences inside the same system like pagers being not consistent with each other (git diff pagers, other pagers).

One example comment:

> I got used to fish and vi mode which are not available when I ssh into servers, containers.

### [color (85)][5]

Lots of problems with color, like:

*   programs setting colors that are unreadable with a light background color
*   finding a colorscheme they like (and getting it to work consistently across different apps)
*   color not working inside several layers of SSH/tmux/etc
*   not liking the defaults
*   not wanting color at all and struggling to turn it off

This comment felt relatable to me:

> Getting my terminal theme configured in a reasonable way between the terminal emulator and fish (I did this years ago and remember it being tedious and fiddly and now feel like I’m locked into my current theme because it works and I dread touching any of that configuration ever again).

### [keyboard shortcuts (84)][6]

Half of the comments on keyboard shortcuts were about how on Linux/Windows, the keyboard shortcut to copy/paste in the terminal is different from in the rest of the OS.

Some other issues with keyboard shortcuts other than copy/paste:

*   using `Ctrl-W` in a browser-based terminal and closing the window
*   the terminal only supports a limited set of keyboard shortcuts (no `Ctrl-Shift-`, no `Super`, no `Hyper`, lots of `ctrl-` shortcuts aren’t possible like `Ctrl-,`)
*   the OS stopping you from using a terminal keyboard shortcut (like by default Mac OS uses `Ctrl+left arrow` for something else)
*   issues using emacs in the terminal
*   backspace not working (2)

### [other copy and paste issues (75)][7]

Aside from “the keyboard shortcut for copy and paste is different”, there were a lot of OTHER issues with copy and paste, like:

*   copying over SSH
*   how tmux and the terminal emulator both do copy/paste in different ways
*   dealing with many different clipboards (system clipboard, vim clipboard, the “middle click” clipboard on Linux, tmux’s clipboard, etc) and potentially synchronizing them
*   random spaces added when copying from the terminal
*   pasting multiline commands which automatically get run in a terrifying way
*   wanting a way to copy text without using the mouse

### [discoverability (55)][8]

There were lots of comments about this, which all came down to the same basic complaint – it’s hard to discover useful tools or features! This comment kind of summed it all up:

> How difficult it is to learn independently. Most of what I know is an assorted collection of stuff I’ve been told by random people over the years.

### [steep learning curve (44)][9]

A lot of comments about it generally having a steep learning curve. A couple of example comments:

> After 15 years of using it, I’m not much faster than using it than I was 5 or maybe even 10 years ago.

and

> That I know I could make my life easier by learning more about the shortcuts and commands and configuring the terminal but I don’t spend the time because it feels overwhelming.

### [history (42)][10]

Some issues with shell history:

*   history not being shared between terminal tabs (16)
*   limits that are too short (4)
*   history not being restored when terminal tabs are restored
*   losing history because the terminal crashed
*   not knowing how to search history

One example comment:

> It wasted a lot of time until I figured it out and still annoys me that “history” on zsh has such a small buffer; I have to type “history 0” to get any useful length of history.

### [bad documentation (37)][11]

People talked about:

*   documentation being generally opaque
*   lack of examples in man pages
*   programs which don’t have man pages

Here’s a representative comment:

> Finding good examples and docs. Man pages often not enough, have to wade through stack overflow

### [scrollback (36)][12]

A few issues with scrollback:

*   programs printing out too much data making you lose scrollback history
*   resizing the terminal messes up the scrollback
*   lack of timestamps
*   GUI programs that you start in the background printing stuff out that gets in the way of other programs’ outputs

One example comment:

> When resizing the terminal (in particular: making it narrower) leads to broken rewrapping of the scrollback content because the commands formatted their output based on the terminal window width.

### [“it feels outdated” (33)][13]

Lots of comments about how the terminal feels hampered by legacy decisions and how users often end up needing to learn implementation details that feel very esoteric. One example comment:

> Most of the legacy cruft, it would be great to have a green field implementation of the CLI interface.

### [shell scripting (32)][14]

Lots of complaints about POSIX shell scripting. There’s a general feeling that shell scripting is difficult but also that switching to a different less standard scripting language (fish, nushell, etc) brings its own problems.

> Shell scripting. My tolerance to ditch a shell script and go to a scripting language is pretty low. It’s just too messy and powerful. Screwing up can be costly so I don’t even bother.

### [more issues][15]

Some more issues that were mentioned at least 10 times:

*   (31) inconsistent command line arguments: is it -h or help or –help?
*   (24) keeping dotfiles in sync across different systems
*   (23) performance (e.g. “my shell takes too long to start”)
*   (20) window management (potentially with some combination of tmux tabs, terminal tabs, and multiple terminal windows. Where did that shell session go?)
*   (17) generally feeling scared/uneasy (“The debilitating fear that I’m going to do some mysterious Bad Thing with a command and I will have absolutely no idea how to fix or undo it or even really figure out what happened”)
*   (16) terminfo issues (“Having to learn about terminfo if/when I try a new terminal emulator and ssh elsewhere.”)
*   (16) lack of image support (sixel etc)
*   (15) SSH issues (like having to start over when you lose the SSH connection)
*   (15) various tmux/screen issues (for example lack of integration between tmux and the terminal emulator)
*   (15) typos & slow typing
*   (13) the terminal getting messed up for various reasons (pressing `Ctrl-S`, `cat`ing a binary, etc)
*   (12) quoting/escaping in the shell
*   (11) various Windows/PowerShell issues

### [n/a (122)][16]

There were also 122 answers to the effect of “nothing really” or “only that I can’t do EVERYTHING in the terminal”

One example comment:

> Think I’ve found work arounds for most/all frustrations

### [that’s all!][17]

I’m not going to make a lot of commentary on these results, but here are a couple of categories that feel related to me:

*   remembering syntax & history (often the thing you need to remember is something you’ve run before!)
*   discoverability & the learning curve (the lack of discoverability is definitely a big part of what makes it hard to learn)
*   “switching systems is hard” & “it feels outdated” (tools that haven’t really changed in 30 or 40 years have many problems but they do tend to be always _there_ no matter what system you’re on, which is very useful and makes them hard to stop using)

Trying to categorize all these results in a reasonable way really gave me an appreciation for social science researchers’ skills.

[1]: https://jvns.ca/terminal-survey/results-bsky
[2]: https://github.com/jvns/classificator
[3]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#remembering-syntax-115
[4]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#switching-terminals-is-hard-91
[5]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#color-85
[6]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#keyboard-shortcuts-84
[7]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#other-copy-and-paste-issues-75
[8]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#discoverability-55
[9]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#steep-learning-curve-44
[10]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#history-42
[11]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#bad-documentation-37
[12]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#scrollback-36
[13]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#it-feels-outdated-33
[14]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#shell-scripting-32
[15]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#more-issues
[16]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#n-a-122
[17]: http://jvns.ca/blog/2025/02/05/some-terminal-frustrations/#that-s-all
