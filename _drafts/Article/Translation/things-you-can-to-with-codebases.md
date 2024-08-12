---
title: Things you can do with codebases - by Thorsten Ball
date: 2001-08-05T00:00:00.000Z
authorURL: ""
originalURL: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases
translator: ""
reviewer: ""
---

Share this post

<!-- more -->

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F62cb77ac-d082-442c-bbc0-109623a9a2ff_2598x1920.png)

#### Things you can do with codebases

registerspill.thorstenball.com

Copy link

Facebook

Email

Note

Other

# Things you can do with codebases

[![](https://substackcdn.com/image/fetch/w_80,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F767e2aa6-bdc8-4dce-a08d-0f194b633a43_1770x1770.jpeg)][1]

[Thorsten Ball][2]

Aug 04, 2024

30

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F62cb77ac-d082-442c-bbc0-109623a9a2ff_2598x1920.png)

#### Things you can do with codebases

registerspill.thorstenball.com

Copy link

Facebook

Email

Note

Other

[

5

][3]

It's easy to forget that we're surrounded by millions and millions of lines of code that we can access and build and run and modify and tweak — whenever and however we want.

Maybe we forget because open source has become the new normal? Or maybe because we all have some public repositories and don’t think much about it? Or maybe — and this is the one I’m putting my money on — it’s because we never stop to think about how amazing that really is?

So let’s let that sink in together: there are over [200 million public repositories on GitHub][4], ready to be cloned, just there for the taking and tweaking.

And yet what most of us think of to do with all that code is reasonable, sane, grown-up, mature: fixing bugs, adding features, cleaning up documentation, helping out in the issue tracker.

But — isn’t a But a beautiful thing sometimes? — here’s a thought: there are millions, billions of lines of code available to play with and you can do whatever the hell you want with them.

Here are some suggestions:

-   Check out the first commit. See what the project was like as a baby. [Here is Redis at its first commit.][5] Nine C files in the root directory. A very clean, nicely commented Hash Table implementation in [slightly more than 500 lines][6]. It’s grown to [2000 lines now.][7]
    
-   Put log statements in, run it and see what it actually does. I mean, why not quadtruple the amount of log statements in [TigerBeetle’s state machine and see what it does][8]? That sounds like a nice morning.
    
-   Break it. Again: why not? How hard could it be to modify PostgreSQL to never write to disk again? What would you learn in the attempt?
    
-   Cut it apart. One of my favorite things to do when debugging: taking a codebase and deleting everything that I don’t think is relevant to the bug. You learn quite a lot about a project’s structure when deleting folder after foldering and trying to get it to run again. Can you, for example, take [Go][9] and delete the code for all of the architectures on which you don’t want to run it?
    
-   Copy out parts and see how they tie in with the rest of the system. Take Redis again: can you simply copy out the Hash Table implementation and run it in your own main.c file?
    
-   Change constants. Hardcode values in it. That’s how I started programming in school when I was 12 years old: we played around with JavaScript and DHTML and this code that generated some floating balls following the mouse cursor, as if they were elastic. I had a big lightbulb-moment when I changed some constants in the code and realised how they affect distance, width, height, and elasticity. (I _think_ it might actually have been [this code][10], but 25 years ago. I should try to get it to run again.)
    
-   Remove all the features you aren’t interested in. Can you delete all the graphs you aren’t interested in from [stats][11] and still have it run okay? How much faster can you make something start up if you remove all the things you never use?
    
-   Print it out.
    

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F62cb77ac-d082-442c-bbc0-109623a9a2ff_2598x1920.png)



][12]

The Meta-Circular Evaluator from Structure and Interpretation of Computer Programs, printed out, sitting on a shelf here in this room.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1bb9586d-d128-4370-93f5-8731d04bdadc.tif)



][13]

## **Joy & Curiosity**

_This is a new section of Register Spill that I want to experiment with, true to the original idea of this newsletter containing [“what I’d send you if you were to ask me what’s on my mind this week.“][14] In this section I want to share things from the previous week I find interesting._

-   Came across the [vim-sneak][15] README this week, thought “why not use / in Vim to navigate?”, and had to laugh out loud when this was addressed in the README with: “`/ab<cr>` requires 33% more keystrokes than `sab`” — That’s fair.
    
-   I keep thinking of [this HackerNews comment][16] in a thread about advice and why advice doesn’t work. I’ve also thought a lot about the general idea of “advice” in the past few years and how hard it is to give advice and how many lessons are tied to the context from which they come. I thought this comment about storytelling being a replacement for advice and how “stories are essentially beneficial viruses” was very interesting.
    
-   [Patrick][17] forward this to me: [The Five Year Rule of Software Transitions][18]. The part about VS Code and text editors stuck out. “Atom was known as being slow, and VS Code has a reputation of being snappy in comparison.”
    
-   After seeing GitHub announcing [Github Models][19] I’m now idly wondering whether we're entering the phase of AI in which it's becoming clear that models are a commodity.
    
-   I’m a big fan of [Dan John][20] and his thoughtfulness. I tried to find a quote of his about the best workout being the one you can repeat every day, not the all-out workout that leaves you hurt for days. Couldn’t find it. Instead found [this interview][21] with him that contains this precious line: “There was always someone else who was far better than me, but they didn’t show up".”
    
-   I love using [HandBrake][22] and especially [this little detail][23].
    
-   Everbody had something to say about [the Friend.com announcement this week][24] and, as often, I don’t think I have a fully-formed opinion yet. It does look dystopian. And yet I started to wonder: what could you do with an “assistant” that can interrupt your thoughts? I’ve read [Don't feed the monkey mind][25] earlier this year and one big idea in it is to try to break bad patterns your mind falls into. When you catch yourself going back to the same thoughts, you need to stop the cycle. Now I’m wondering: could something like the Friend.com thing help you stop your thoughts? Could it message you after listening in to a conversation with your boss, saying, “hey, don’t overthink this, I know you assume that what he said about X, Y, Z and is about you, but it’s not.” Again: not sure what my opinion will be (if I will have any), but I sure wish I could re-program my thought patterns sometimes.
    
-   This horror story in the form of a [HackerNews comment][26]. Pow: “The solution was obvious: installing Visual Studio for each customer on site and teaching the users to run the app in debug mode from Visual Studio” And bang: “What happened next was even worse.”
    

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe567c301-07bf-48d1-8c45-ea4ae008b7eb.tif)



][27]

You can subscribe here if you want to receive this newsletter, including the artisanal, hand-crafted divider-images, in your inbox:

30

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F62cb77ac-d082-442c-bbc0-109623a9a2ff_2598x1920.png)

#### Things you can do with codebases

registerspill.thorstenball.com

Copy link

Facebook

Email

Note

Other

[

5

][28]

5 Comments

<table class="comment-content"><tbody><tr><td class="comment-head"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><div class="user-head"><a href="https://substack.com/profile/15930373-vi_mi"><div class="profile-img-wrap"><picture><source type="image/webp" srcset="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F867bb01c-b011-464f-a1da-e01462a5fb75_144x144.png"><img src="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F867bb01c-b011-464f-a1da-e01462a5fb75_144x144.png" sizes="100vw" alt="" width="66" height="66" class="_img_16u6n_1 pencraft pc-reset"></picture></div></a></div></div></td><td class="comment-rest"><div class="comment-meta"><span class="commenter-name"><div class="pencraft pc-display-flex pc-gap-4 pc-alignItems-center pc-reset pc-display-inline-flex"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><a href="https://substack.com/profile/15930373-vi_mi"><div class="pencraft pc-reset _color-pub-primary-text_3axfk_204 _line-height-20_3axfk_95 _font-text_3axfk_121 _size-13_3axfk_45 _weight-semibold_3axfk_165 _reset_3axfk_1">vi_mi</div></a></div></div></span><span class="highlight"><svg role="img" width="24" height="24" viewBox="0 0 24 24" fill="#000000" stroke-width="1.8" stroke="#000" xmlns="http://www.w3.org/2000/svg"><g><title></title><path d="M20.4134 4.58753C19.9121 4.08434 19.3164 3.68508 18.6606 3.41266C18.0047 3.14023 17.3015 3 16.5914 3C15.8812 3 15.178 3.14023 14.5222 3.41266C13.8663 3.68508 13.2707 4.08434 12.7694 4.58753L12 5.36717L11.2306 4.58753C10.7293 4.08434 10.1337 3.68508 9.47781 3.41266C8.82195 3.14023 8.11878 3 7.40863 3C6.69847 3 5.9953 3.14023 5.33944 3.41266C4.68358 3.68508 4.08793 4.08434 3.58665 4.58753C1.46832 6.70656 1.33842 10.2849 4.00631 13.0037L12 21L19.9937 13.0037C22.6616 10.2849 22.5317 6.70656 20.4134 4.58753Z"></path></g></svg><span class="highlight-text">Liked by Thorsten Ball</span></span></div><div class="comment-body"><p><span>Hey Thorsten!</span></p><p><span>Thanks for writing a spill every week, I really enjoy reading it. Also, would love to see new section of `Joy &amp; Curiosity` continue, absolutely loved it!</span></p><div role="button" class="show-all-toggle"><div class="show-all-toggle-label">Expand full comment</div></div></div><div class="pencraft pc-display-flex pc-gap-16 pc-paddingTop-8 pc-justifyContent-flex-start pc-alignItems-center pc-reset comment-actions _withShareButton_mhaex_5"><span class="pencraft pc-reset _decoration-hover-underline_3axfk_298 _reset_3axfk_1"><a class="pencraft pc-reset _link_mhaex_1 _link_1ixw5_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle"><path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"></path></svg><div class="pencraft pc-reset _color-pub-secondary-text_3axfk_207 _line-height-20_3axfk_95 _font-meta_3axfk_131 _size-11_3axfk_35 _weight-medium_3axfk_162 _transform-uppercase_3axfk_242 _reset_3axfk_1 _meta_3axfk_442">Reply</div></div></a></span><span class="pencraft pc-reset _decoration-hover-underline_3axfk_298 _reset_3axfk_1"><a class="pencraft pc-reset _link_mhaex_1 _link_1ixw5_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-share"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path><polyline points="16 6 12 2 8 6"></polyline><line x1="12" x2="12" y1="2" y2="15"></line></svg><div class="pencraft pc-reset _color-pub-secondary-text_3axfk_207 _line-height-20_3axfk_95 _font-meta_3axfk_131 _size-11_3axfk_35 _weight-medium_3axfk_162 _transform-uppercase_3axfk_242 _reset_3axfk_1 _meta_3axfk_442">Share</div></div></a></span><button tabindex="0" type="button" id="trigger183845" aria-expanded="false" aria-haspopup="dialog" aria-controls="dialog183846" arialabel="View more" class="pencraft pc-reset _iconButton_1oht6_145 _iconButtonBase_1oht6_145 _buttonBase_1oht6_1 _buttonOldColors_1oht6_56 _priority_secondary-theme_1oht6_256 _fill_empty_1oht6_317"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-more-horizontal"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg></button></div></td></tr></tbody></table>

[1 reply by Thorsten Ball][31]

<table class="comment-content"><tbody><tr><td class="comment-head"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><div class="user-head"><a href="https://substack.com/profile/257003607-wise-guy"><div class="profile-img-wrap"><picture><source type="image/webp" srcset="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack.com%2Fimg%2Favatars%2Forange.png"><img src="https://substackcdn.com/image/fetch/w_66,h_66,c_fill,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack.com%2Fimg%2Favatars%2Forange.png" sizes="100vw" alt="" width="66" height="66" class="_img_16u6n_1 pencraft pc-reset"></picture></div></a></div></div></td><td class="comment-rest"><div class="comment-meta"><span class="commenter-name"><div class="pencraft pc-display-flex pc-gap-4 pc-alignItems-center pc-reset pc-display-inline-flex"><div class="profile-hover-card-target _profileHoverCardTarget_c9bh7_50"><a href="https://substack.com/profile/257003607-wise-guy"><div class="pencraft pc-reset _color-pub-primary-text_3axfk_204 _line-height-20_3axfk_95 _font-text_3axfk_121 _size-13_3axfk_45 _weight-semibold_3axfk_165 _reset_3axfk_1">wise guy</div></a></div></div></span><a href="https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comment/64336318" rel="nofollow" native="" class="comment-timestamp">Aug 4</a><span class="comment-publication-name-separator">·</span><span class="edited-indicator"><em>edited Aug 4</em></span></div><div class="comment-body"><p><span>"did you know you can just do things ?"</span></p><div role="button" class="show-all-toggle"><div class="show-all-toggle-label">Expand full comment</div></div></div><div class="pencraft pc-display-flex pc-gap-16 pc-paddingTop-8 pc-justifyContent-flex-start pc-alignItems-center pc-reset comment-actions _withShareButton_mhaex_5"><span class="pencraft pc-reset _decoration-hover-underline_3axfk_298 _reset_3axfk_1"><a class="pencraft pc-reset _link_mhaex_1 _link_1ixw5_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle"><path d="M7.9 20A9 9 0 1 0 4 16.1L2 22Z"></path></svg><div class="pencraft pc-reset _color-pub-secondary-text_3axfk_207 _line-height-20_3axfk_95 _font-meta_3axfk_131 _size-11_3axfk_35 _weight-medium_3axfk_162 _transform-uppercase_3axfk_242 _reset_3axfk_1 _meta_3axfk_442">Reply</div></div></a></span><span class="pencraft pc-reset _decoration-hover-underline_3axfk_298 _reset_3axfk_1"><a class="pencraft pc-reset _link_mhaex_1 _link_1ixw5_1"><div class="pencraft pc-display-flex pc-gap-6 pc-alignItems-center pc-reset"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--color-secondary-themed)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-share"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path><polyline points="16 6 12 2 8 6"></polyline><line x1="12" x2="12" y1="2" y2="15"></line></svg><div class="pencraft pc-reset _color-pub-secondary-text_3axfk_207 _line-height-20_3axfk_95 _font-meta_3axfk_131 _size-11_3axfk_35 _weight-medium_3axfk_162 _transform-uppercase_3axfk_242 _reset_3axfk_1 _meta_3axfk_442">Share</div></div></a></span><button tabindex="0" type="button" id="trigger183847" aria-expanded="false" aria-haspopup="dialog" aria-controls="dialog183848" arialabel="View more" class="pencraft pc-reset _iconButton_1oht6_145 _iconButtonBase_1oht6_145 _buttonBase_1oht6_1 _buttonOldColors_1oht6_56 _priority_secondary-theme_1oht6_256 _fill_empty_1oht6_317"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-more-horizontal"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg></button></div></td></tr></tbody></table>

[1 reply by Thorsten Ball][35]

[3 more comments...][36]

Top

Latest

Discussions

No posts

Ready for more?

[1]: https://substack.com/profile/1234646-thorsten-ball
[2]: https://substack.com/@thorstenball
[3]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comments
[4]: https://github.com/search?q=is%3Apublic&type=repositories
[5]: https://github.com/redis/redis/tree/ed9b544e10b84cd43348ddfab7068b610a5df1f7
[6]: https://github.com/redis/redis/blob/ed9b544e10b84cd43348ddfab7068b610a5df1f7/dict.c
[7]: https://github.com/redis/redis/blob/8038eb3147ad1c528a58142464584ada376936ee/src/dict.c
[8]: https://github.com/tigerbeetle/tigerbeetle/blob/fc11bec55efa40e9882613d008a91d0f93a43957/src/state_machine.zig
[9]: https://github.com/golang/go
[10]: http://jsmadeeasy.com/javascripts/cursor%20effects/elastic%20bullets/template.htm
[11]: https://github.com/exelban/stats
[12]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F62cb77ac-d082-442c-bbc0-109623a9a2ff_2598x1920.png
[13]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1bb9586d-d128-4370-93f5-8731d04bdadc.tif
[14]: https://registerspill.thorstenball.com/about
[15]: https://github.com/justinmk/vim-sneak
[16]: https://news.ycombinator.com/item?id=41111324
[17]: https://dubroy.com/blog/
[18]: https://blog.robenkleene.com/2023/06/19/software-transitions-the-five-year-rule/
[19]: https://x.com/ashtom/status/1819041110200906202
[20]: https://www.youtube.com/channel/UCrf_X-KnNGBy75IGsPuI7AQ
[21]: https://www.trainheroic.com/blog/simplify-the-weightroom/
[22]: https://handbrake.fr/
[23]: https://x.com/thorstenball/status/1817889819344748936?s=46
[24]: https://x.com/AviSchiffmann/status/1818284595902922884
[25]: https://www.goodreads.com/book/show/33977140-don-t-feed-the-monkey-mind
[26]: https://news.ycombinator.com/item?id=41147787
[27]: https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe567c301-07bf-48d1-8c45-ea4ae008b7eb.tif
[28]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comments
[29]: https://substack.com/profile/15930373-vi_mi
[30]: https://substack.com/profile/15930373-vi_mi
[31]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comment/64419190
[32]: https://substack.com/profile/257003607-wise-guy
[33]: https://substack.com/profile/257003607-wise-guy
[34]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comment/64336318
[35]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comment/64336318
[36]: https://registerspill.thorstenball.com/p/things-you-can-to-with-codebases/comments