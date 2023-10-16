---
title: High Growth Engineer
authorURL: ""
originalURL: https://careercutler.substack.com/p/the-top-7-software-engineering-workflow
translator: ""
reviewer: ""
---

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe2aed624-b6bc-45a9-b8f5-b28b519871a1_1920x1280.jpeg)

#### The top 7 software engineering workflow tips I wish I knew earlier 🧰

careercutler.substack.com

Copy link

Facebook

Email

Notes

Other

![](https://substackcdn.com/image/fetch/w_128,h_128,c_fill,f_auto,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fec021be5-22c2-4ee3-81f7-d331175ee9e9_1100x1251.jpeg)![](https://substackcdn.com/image/fetch/w_48,h_48,c_fill,f_auto,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0eb0a5d7-590d-4475-a945-fbb688e2ce5e_780x780.png)

#### Discover more from High Growth Engineer

Grow faster in your software engineering career with actionable tips you can start using right away

Over 18,000 subscribers

Subscribe

Continue reading

Sign in

# The top 7 software engineering workflow tips I wish I knew earlier 🧰

[![](https://substackcdn.com/image/fetch/w_80,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fec021be5-22c2-4ee3-81f7-d331175ee9e9_1100x1251.jpeg)](https://substack.com/profile/58854493-jordan-cutler)

[Jordan Cutler](https://substack.com/@jordancutler)

Oct 8, 2023

111

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe2aed624-b6bc-45a9-b8f5-b28b519871a1_1920x1280.jpeg)

#### The top 7 software engineering workflow tips I wish I knew earlier 🧰

careercutler.substack.com

Copy link

Facebook

Email

Notes

Other

[

31

](https://careercutler.substack.com/p/the-top-7-software-engineering-workflow/comments)

[

Share

](javascript:void(0))

Your time matters.

1 hour of efficiency gain per day saves you 1 month per year.

I’ll prove it to you:

_1 hour per day x 5 days per week x 52 weeks = 260 hours saved per year_

_260 hours / 8 hours per day = 32.5 work days per year._

That’s 1 month you could be closer to promotion, relaxing, or doing whatever the heck you want.

Optimizing the things I do every day has led to me averaging 1-2 pull requests per day over my last few years as a software engineer. **It’s not the best metric (and not something to strive for), but it gives some frame of reference.**

Today, I’m going to share the biggest workflow tips that made this possible.

---

## [System design challenges (Sponsored)](https://www.systemsdesignchallenges.com)

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6b3f06a3-2d91-4470-94a3-317a4ca88350_500x500.webp)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6b3f06a3-2d91-4470-94a3-317a4ca88350_500x500.webp)

Learn system design through real-world challenges from Senior Engineer, Manager, and CTO [Ricardo Morales](https://www.linkedin.com/in/ricardo-morales-1820/). A recent one I liked was [designing Google Sheets](https://www.systemsdesignchallenges.com/p/systems-design-challenge-1-of-2).

[Subscribe to System Design Challenges](https://www.systemsdesignchallenges.com/)

Thank you to Ricardo and his newsletter for sponsoring and keeping High Growth Engineer free for you all 🙏

---

## 💡 Big idea

Optimize the tasks you do **every day**, multiple times per day.

For example, taking the equation at the start of the article, for every 2 minutes you save per workday in automation or a better process, you get back 1 workday per year.

You can use the chart below to decide if something is worth optimizing, but I don’t.

I optimize repeated daily tasks and hope for the best 🤞

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9c3189f5-9da9-4f2e-87d1-00ca60e5c4d2_571x464.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9c3189f5-9da9-4f2e-87d1-00ca60e5c4d2_571x464.png)

xkcd time optimization chart. Note: It’s not entirely accurate to a “working day” since it’s based on a 24-hour day. [Source](https://xkcd.com/1205/)

In each section, I’ll cover the most frequent parts of a software engineer’s workflow.

Each section includes what I do, but you may find things you’re already doing more efficiently than me! That’s great. Feel free to tell me! I’m always looking to improve.

## (1) 💻 Git / Terminal workflow

There are 3 huge time savers here:

1.  Autocomplete past commands
    
2.  Aliases for commands
    
3.  Easily manage Git files
    

At a high level, I recommend checking out the [table of contents in this terminal makeover article](https://towardsdatascience.com/the-ultimate-guide-to-your-terminal-makeover-e11f9b87ac99). However, I’ll go through 3 major time savers.

### Autocomplete past commands

I use the following terminal setup:

-   [iTerm2](https://iterm2.com/) - Terminal
    
-   [Oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) and Zsh (replacement for bash) - [Setup guide here](https://github.com/ohmyzsh/ohmyzsh)
    
-   [Starship](https://starship.rs/) - Customize terminal prompt (not needed for this but nice to know)
    

Once you set up oh-my-zsh, you can add these “plugins” to your `.zshrc` file

```
# in ~/.zshrc
plugins=(
  git
  zsh-autosuggestions <--- we'll talk about this one now
  zsh-syntax-highlighting
)
```

With zsh-autosuggestions, every command you type will start autocompleting it.

See the demo below.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F06546601-9068-4b82-9640-cdf6918b5883_800x310.gif)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F06546601-9068-4b82-9640-cdf6918b5883_800x310.gif)

All I need to do is hit the right arrow on my keyboard and it will populate.

### Aliases

Above, we saw I added this to `.zshrc`

```
# in ~/.zshrc
plugins=(
  git <--- we'll talk about this one now
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

The git plugin adds aliases for git commands. A few that I use are:

-   ga => git add
    
-   gc => git commit
    
-   gd => git diff
    
-   gs => git status
    
-   gps => git push
    
-   gpl => git pull
    

This saves me at least a minute or two per day, which adds up over a year.

Another thing I do is add custom aliases in \`.zshrc\`

Here are two examples

```
# in ~/.zshrc
alias addalias='code ~/.zshrc' <-- open up zshrc file to edit it
alias reload='source ~/.zshrc' <-- reload zshrc file to apply changes
```

### Numbered shortcuts for git files

Adding [SCM Breeze](https://github.com/scmbreeze/scm_breeze) allows you to easily work with your files in Git.

These numbered shortcuts are what SCM Breeze gets you:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0d235595-37c2-4dcc-bdcb-4a106848f61c_764x340.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0d235595-37c2-4dcc-bdcb-4a106848f61c_764x340.png)

You can refer to your files like `git add 2-3` now or `git reset 1`

See the example below:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F94733d79-29a2-46e5-b975-5eb5d7b01620_800x500.gif)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F94733d79-29a2-46e5-b975-5eb5d7b01620_800x500.gif)

## (2) 🖥️ Coding

Below are the biggest areas for optimizing your time in your code editor

1.  **Tracing code down or up a stack** - Assume you have a method call you’re looking at. You want to see what the code is inside that method. **Don’t globally cmd+f that function name searching for the definition!**
    
    1.  **Do this:** Have a keyboard shortcut for tracing a variable, method, class, etc.
        
    2.  VSCode shortcut: `F12`.
        
    3.  Jetbrains IDE shortcut: `cmd+b`
        
2.  **Navigating between locations** - Often you’re working across 2-3 files, working in the same place in each file and clicking the nav at the top to find which of the 15 files you’re working with. Or you’re working in a large file and scrolling back and forth. There’s a better way though.
    
    1.  **Do this:** Have a keyboard shortcut for going backward and forward in your “location history.” It brings you backward and forward through all the places you edited, across files, so you can easily swap between files and locations.
        
    2.  VSCode shortcut: `ctrl + -` and `ctrl + shift + -`
        
    3.  Jetbrains IDE shortcut: `cmd + option + ⬅️` and `cmd + option + ➡️`
        
3.  **Typing** - Using [Github Copilot](https://github.com/features/copilot) has saved me days of time already since starting to use it a few months ago. Almost every single line of code you write can get autocompleted for you, including tests. It’s mindblowing 🤯 how smart it is.
    

## (3) 📓 Save learnings

Everything I learn and want to store for later I save in Notion.

[Tiago Forte](https://www.youtube.com/@TiagoForte) popularized this concept as “building a second brain.”

One key takeaway is to store your learnings where you will **use them**, not where you learned them.

Here’s a screenshot of 2 of my Notion folders for notes:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f934cb1-e3fc-4473-abb2-0eb472ed85cb_1182x960.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f934cb1-e3fc-4473-abb2-0eb472ed85cb_1182x960.png)

Each topic is a collection of my learnings from many different articles, videos, books, etc. This helps me find them in the future much easier.

In the future, I plan to more publicly share the content of these, but a lot of my articles are curated versions of what is in my notes.

## (4) ✅ Offloading ideas and tasks

Your brain should be used for creativity, problem-solving, and coming up with ideas—**not storing them.**

I use a combination of [Todoist](https://todoist.com/) and Slack “save for later” reminders to store any idea or task, so I can immediately get it out of my brain.

Before I started doing this, my brain was constantly “thrashing.” I’d swap between tasks in my head trying to remember them and take 10x longer to actually get it done.

This is how I’ve organized my Todoist, but you should do whatever works best for you.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbcac5a15-b230-4099-a888-95e8d2192083_2328x1220.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbcac5a15-b230-4099-a888-95e8d2192083_2328x1220.png)

**Note:** One nice feature I like is that you can add websites as tasks, so if you come across a video or article you want to view later, you can add it as a to-do in one click.

Easter egg🥚if you can notice my todo item to put up shelves for my cat to climb 😄

**The key takeaway here is that it doesn’t matter what tool you use. Make sure you can let your brain focus on solving problems, not remembering everything.**

## (5) 👀 Communicating through visuals

Here are a few daily use cases where you could use visuals to better communicate:

-   Documenting a pull request
    
-   Sending a Slack message explaining a change you want to make to a PM
    
-   Getting approval or sign-off on screenshots for a new flow you built
    

[CleanShot](https://cleanshot.sjv.io/oqVmZg) ([non-affiliate link option](https://cleanshot.com/)) has saved me an immense amount of time here.

It replaces the screenshot tool on your Mac and gives you…

-   A quick and easy way to apply a ton of visual edits to a screenshot you just took, and drop it anywhere
    
-   An all-in-one gif + screen recording tool too. It’s how I created the other visuals in this post.
    
-   Plenty more features (like scrolling capture, blurring parts of an image, etc.). If you’re interested, I’d recommend checking [their demo video](https://www.youtube.com/watch?v=FZbICrBKWIU).
    

Below is a slightly contrived example, but annotating my pull requests with screenshots is something this lets me do a lot, which results in faster approvals:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1239c71c-7928-49a2-9f85-33ebfa70a683_1614x968.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1239c71c-7928-49a2-9f85-33ebfa70a683_1614x968.png)

Another common case is making quick screen recordings for presentations or sending a Slack message to my designer or PM getting sign-off on a feature behavior.

## (6) 🔑 Signing into things

Password managers are a game changer for 2 reasons:

1.  Convenience - Never type emails or passwords again. It will do it for you.
    
2.  Security - It will create complex and unique passwords across every site for you.
    

Here’s how it looks when you sign in to sites using [1Password](https://1password.com/personal):

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F33e47349-d67e-42d5-a073-06a5a273ccaa_1044x828.png)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F33e47349-d67e-42d5-a073-06a5a273ccaa_1044x828.png)

## (7) 🪟 Window management

As software engineers, we’re constantly managing our browser window, terminal, editor, and messaging app.

For a while, I spent so much time manually resizing windows until I started using a window management tool.

I use [Rectangle](https://rectangleapp.com/). It’s free and the pro version is only $10 for lifetime access.

It lets me quickly put 1 window on the left side, 1 window on the right, full screen another window, etc. It saves me a ton of time.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F604f5344-87ae-41fd-86ac-0486512a1b19_800x452.gif)



](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F604f5344-87ae-41fd-86ac-0486512a1b19_800x452.gif)

## 💭 Closing thought

Incorporating all of these has been a 6-year process for me. I need to slowly incorporate new tools otherwise I’ll get overwhelmed.

**My intent with this post is not to make anyone feel like they’re doing something wrong if they’re not using these tools.**

It’s just to share what has worked well for me and something for you to try if you’re interested.

You don’t need to use the same tools I use. Use what works for you.

Feel free to share in the comments what you use that’s most helpful 🙏

## 📣 Shoutouts of the week

-   [Lessons in Emotional Intelligence: How to make someone truly feel heard and seen](https://www.thecaringtechie.com/p/lessons-in-emotional-intelligence) - by Irina Stanescu. Solid lessons and examples on how to improve your emotional intelligence.
    
-   [How I convinced we weren’t wrong](https://ravirajachar.substack.com/p/how-i-convinced-we-werent-wrong) - by Raviraj Achar. Excellent story and takeaways on how to effectively resolve tense disagreements between teams.
    
-   [How to get more interviews, offers, and compensation at top tech companies](https://www.linkedin.com/events/7116067854558355456) - Upcoming LinkedIn live event with myself and Alan Stein (Director+ level at many FAANG companies). Click the link to attend. It will be at 5 PM PST this Tuesday, October 10th.
    

---

_As always, thank you for reading and for the growth to 16k+ subscribers._

\- Jordan

_P.S. If you’re interested, I’m accepting the following:_

1.  _New coaching clients: See [Mentorcruise for rates](https://mentorcruise.com/mentor/jordancutler/)_
    
2.  _Newsletter sponsorships: Feel free to reply to this email to reach me._
    

Did you find this issue valuable? If so, there are two ways you can help:

Join 16,000+ engineers growing every Sunday with real-world, actionable, advice.

Subscribe

You can also hit the like ❤️ button at the bottom of this email to help support me. It really helps!

111

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe2aed624-b6bc-45a9-b8f5-b28b519871a1_1920x1280.jpeg)

#### The top 7 software engineering workflow tips I wish I knew earlier 🧰

careercutler.substack.com

Copy link

Facebook

Email

Notes

Other

[

31

](https://careercutler.substack.com/p/the-top-7-software-engineering-workflow/comments)

[

Share

](javascript:void(0))

Previous

Next