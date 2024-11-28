---
title: Julia Evans
date: 2024-09-12T15:09:12.000Z
authorURL: ""
originalURL: https://jvns.ca/blog/2024/09/12/reasons-i--still--love-fish/
translator: ""
reviewer: ""
---

I wrote about how much I love [fish](https://fishshell.com/) in [this blog post from 2017](https://jvns.ca/blog/2017/04/23/the-fish-shell-is-awesome/) and, 7 years of using it every day later, I’ve found even more reasons to love it. So I thought I’d write a new post with both the old reasons I loved it and some reasons.

This came up today because I was trying to figure out why my terminal doesn’t break anymore when I cat a binary to my terminal, the answer was “fish fixes the terminal!”, and I just thought that was really nice.

### 1. no configuration

In 10 years of using fish I have never found a single thing I wanted to configure. It just works the way I want. My fish config file just has:

*   environment variables
*   aliases (`alias ls eza`, `alias vim nvim`, etc)
*   the occasional `direnv hook fish | source` to integrate a tool like direnv
*   a script I run to set up my [terminal colours](https://github.com/chriskempson/base16-shell/blob/588691ba71b47e75793ed9edfcfaa058326a6f41/scripts/base16-solarized-light.sh)

I’ve been told that configuring things in fish is really easy if you ever do want to configure something though.

### 2. autosuggestions from my shell history

My absolute favourite thing about fish is that I type, it’ll automatically suggest (in light grey) a matching command that I ran recently. I can press the right arrow key to accept the completion, or keep typing to ignore it.

Here’s what that looks like. In this example I just typed the “v” key and it guessed that I want to run the previous vim command again.

![Image 7](https://jvns.ca/images/fish-2024.png)

### 2.5 “smart” shell autosuggestions

One of my favourite subtle autocomplete features is how fish handles autocompleting commands that contain paths in them. For example, if I run:

```bash
$ ls blah.txt
```

that command will only be autocompleted in directories that contain `blah.txt` – it won’t show up in a different directory. (here’s [a short comment about how it works](https://github.com/fish-shell/fish-shell/issues/120#issuecomment-6376019))

As an example, if in this directory I type `bash scripts/`, it’ll only suggest history commands including files that _actually exist_ in my blog’s scripts folder, and not the dozens of other irrelevant `scripts/` commands I’ve run in other folders.

I didn’t understand exactly how this worked until last week, it just felt like fish was magically able to suggest the right commands. It still feels a little like magic and I love it.

### 3. pasting multiline commands

If I copy and paste multiple lines, bash will run them all, like this:

```bash
[bork@grapefruit linux-playground (main)]$ echo hi
hi
[bork@grapefruit linux-playground (main)]$ touch blah
[bork@grapefruit linux-playground (main)]$ echo hi
hi
```

This is a bit alarming – what if I didn’t actually _want_ to run all those commands?

Fish will paste them all at a single prompt, so that I can press Enter if I actually want to run them. Much less scary.

```bash
bork@grapefruit ~/work/> echo hi

                         touch blah
                         echo hi
```

### 4. nice tab completion

If I run `ls` and press tab, it’ll display all the filenames in a nice grid. I can use either Tab, Shift+Tab, or the arrow keys to navigate the grid.

Also, I can tab complete from the **middle** of a filename – if the filename starts with a weird character (or if it’s just not very unique), I can type some characters from the middle and press tab.

Here’s what the tab completion looks like:

```bash
bork@grapefruit ~/work/> ls 
api/  blah.py     fly.toml   README.md
blah  Dockerfile  frontend/  test_websocket.sh
```

I honestly don’t complete things other than filenames very much so I can’t speak to that, but I’ve found the experience of tab completing filenames to be very good.

### 5. nice default prompt (including git integration)

Fish’s default prompt includes everything I want:

*   username
*   hostname
*   current folder
*   git integration
*   status of last command exit (if the last command failed)

Here’s a screenshot with a few different variations on the default prompt, including if the last command was interrupted (the `SIGINT`) or failed.

![Image 8](https://jvns.ca/images/fish-prompt-2024.png)

### 6. nice history defaults

In bash, the maximum history size is 500 by default, presumably because computers used to be slow and not have a lot of disk space. Also, by default, commands don’t get added to your history until you end your session. So if your computer crashes, you lose some history.

In fish:

1.  the default history size is 256,000 commands. I don’t see any reason I’d ever need more.
2.  if you open a new tab, everything you’ve ever run (including commands in open sessions) is immediately available to you
3.  in an existing session, the history search will only include commands from the current session, plus everything that was in history at the time that you started the shell

I’m not sure how clearly I’m explaining how fish’s history system works here, but it feels really good to me in practice. My impression is that the way it’s implemented is the commands are continually added to the history file, but fish only loads the history file once, on startup.

I’ll mention here that if you want to have a fancier history system in another shell it might be worth checking out [atuin](https://github.com/atuinsh/atuin) or [fzf](https://github.com/junegunn/fzf).

### 7. press up arrow to search history

I also like fish’s interface for searching history: for example if I want to edit my fish config file, I can just type:

```bash
$ config.fish
```

and then press the up arrow to go back the last command that included `config.fish`. That’ll complete to:

```bash
$ vim ~/.config/fish/config.fish
```

and I’m done. This isn’t _so_ different from using `Ctrl+R` in bash to search your history but I think I like it a little better over all, maybe because `Ctrl+R` has some behaviours that I find confusing (for example you can end up accidentally editing your history which I don’t like).

### 8. the terminal doesn’t break

I used to run into issues with bash where I’d accidentally `cat` a binary to the terminal, and it would break the terminal.

Every time fish displays a prompt, it’ll try to fix up your terminal so that you don’t end up in weird situations like this. I think [this is some of the code in fish to prevent broken terminals](https://github.com/fish-shell/fish-shell/blob/a979b6341d7fc4c466b3992f25da3209e0808aaa/src/reader.rs#L3601-L3623).

Some things that it does are:

*   turn on `echo` so that you can see the characters you type
*   make sure that newlines work properly so that you don’t get that weird staircase effect
*   reset your terminal background colour, etc

I don’t think I’ve run into any of these “my terminal is broken” issues in a very long time, and I actually didn’t even realize that this was because of fish – I thought that things somehow magically just got better, or maybe I wasn’t making as many mistakes. But I think it was mostly fish saving me from myself, and I really appreciate that.

### 9. Ctrl+S is disabled

Also related to terminals breaking: fish disables Ctrl+S (which freezes your terminal and then you need to remember to press Ctrl+Q to unfreeze it). It’s a feature that I’ve never wanted and I’m happy to not have it.

Apparently you can disable `Ctrl+S` in other shells with `stty -ixon`.

### 10. `fish_add_path`

I have mixed feelings about this one, but in Fish you can use `fish_add_path /opt/whatever/bin` to add a path to your PATH, globally, permanently, across all open shell sessions. This can get a bit confusing if you forget where those PATH entries are configured but overall I think I appreciate it.

### 11. nice syntax highlighting

By default commands that don’t exist are highlighted in red, like this.

![Image 9](https://jvns.ca/images/fish-syntax-2024.png)

### 12\. easier loops

I find the loop syntax in fish a lot easier to type than the bash syntax. It looks like this:

```
for i in *.yaml
  echo $i
end
```

Also it’ll add indentation in your loops which is nice.

### 13. easier multiline editing

Related to loops: you can edit multiline commands much more easily than in bash (just use the arrow keys to navigate the multiline command!). Also when you use the up arrow to get a multiline command from your history, it’ll show you the whole command the exact same way you typed it instead of squishing it all onto one line like bash does:

```bash
$ bash
$ for i in *.png
> do
> echo $i
> done
$ # press up arrow
$ for i in *.png; do echo $i; done ink
```

### 14. Ctrl+left arrow

This might just be me, but I really appreciate that fish has the `Ctrl+left arrow` / `Ctrl+right arrow` keyboard shortcut for moving between words when writing a command.

I’m honestly a bit confused about where this keyboard shortcut is coming from (the only documented keyboard shortcut for this I can find in fish is `Alt+left arrow` / `Alt + right arrow` which seems to do the same thing), but I’m pretty sure this is a fish shortcut.

A couple of notes about getting this shortcut to work / where it comes from:

*   one person said they needed to switch their terminal emulator from the “Linux console” keybindings to “Default (XFree 4)” to get it to work in fish
*   on Mac OS, `Ctrl+left arrow` switches workspaces by default, so I had to turn that off.
*   Also apparently Ubuntu configures libreadline in `/etc/inputrc` to make `Ctrl+left/right arrow` go back/forward a word, so it’ll work in bash on Ubuntu and maybe other Linux distros too. Here’s a [stack overflow question talking about that](https://stackoverflow.com/questions/5029118/bash-ctrl-to-move-cursor-between-words-strings)

### a downside: not everything has a fish integration

Sometimes tools don’t have instructions for integrating them with fish. That’s annoying, but:

*   I’ve found this has gotten better over the last 10 years as fish has gotten more popular. For example Python’s virtualenv has had a fish integration for a long time now.
*   If I need to run a POSIX shell command real quick, I can always just run `bash` or `zsh`
*   I’ve gotten much better over the years at translating simple commands to fish syntax when I need to

My biggest day-to-day to annoyance is probably that for whatever reason I’m still not used to fish’s syntax for setting environment variables, I get confused about `set` vs `set -x`.

### on POSIX compatibility

When I started using fish, you couldn’t do things like `cmd1 && cmd2` – it would complain “no, you need to run `cmd1; and cmd2`” instead.

It seems like over the years fish has started accepting a little more POSIX-style syntax than it used to, like:

*   `cmd1 && cmd2`
*   `export a=b` to set an environment variable (though this seems a bit limited, you can’t do `export PATH=$PATH:/whatever` so I think it’s probably better to learn `set` instead)

### on fish as a default shell

Changing my default shell to fish is always a little annoying, I occasionally get myself into a situation where

1.  I install fish somewhere like maybe `/home/bork/.nix-stuff/bin/fish`
2.  I add the new fish location to `/etc/shells` as an allowed shell
3.  I change my shell with `chsh`
4.  at some point months/years later I reinstall fish in a different location for some reason and remove the old one
5.  oh no!!! I have no valid shell! I can’t open a new terminal tab anymore!

This has never been a major issue because I always have a terminal open somewhere where I can fix the problem and rescue myself, but it’s a bit alarming.

If you don’t want to use `chsh` to change your shell to fish (which is very reasonable, maybe I shouldn’t be doing that), the [Arch wiki page](https://wiki.archlinux.org/title/Fish) has a couple of good suggestions – either configure your terminal emulator to run fish or add an `exec fish` to your `.bashrc`.

### I’ve never really learned the scripting language

Other than occasionally writing a for loop interactively on the command line, I’ve never really learned the fish scripting language. I still do all of my shell scripting in bash.

I don’t think I’ve ever written a fish function or `if` statement.

### it seems like fish is getting pretty popular

I ran a highly unscientific poll on Mastodon asking people what shell they [use interactively](https://social.jvns.ca/@b0rk/112722850642874842). The results were (of 2600 responses):

*   46% bash
*   49% zsh
*   16% fish
*   5% other

I think 16% for fish is pretty remarkable, since (as far as I know) there isn’t any system where fish is the default shell, and my sense is that it’s very common to just stick to whatever your system’s default shell is.

It feels like a big achievement for the fish project, even if maybe my Mastodon followers are more likely than the average shell user to use fish for some reason.

### who might fish be right for?

Fish definitely isn’t for everyone. I think I like it because:

1.  I really dislike configuring my shell (and honestly my dev environment in general), I want things to “just work” with the default settings
2.  fish’s defaults feel good to me
3.  I don’t spend that much time logged into random servers using other shells so there’s not too much context switching
4.  I liked its features so much that I was willing to relearn how to do a few “basic” shell things, like using parentheses `(seq 1 10)` to run a command instead of backticks or using `set` instead of `export`

Maybe you’re also a person who would like fish! I hope a few more of the people who fish is for can find it, because I spend so much of my time in the terminal and it’s made that time much more pleasant.