---
title: "Dear Rubyists: Shopify Isn't Your Enemy"
date: 2025-10-09T05:03:51.000Z
authorURL: ""
originalURL: https://byroot.github.io/opensource/ruby/2025/10/09/dear-rubyists.html
translator: ""
reviewer: ""
---

# Dear Rubyists: Shopify Isn't Your Enemy

<!-- more -->

I’ve been meaning to write a post about my perspective on Open Source and corporate entities. I already got the rough outline of it; however, I’m suffering from writer’s block, but more importantly, the whole post is a praise of how Shopify engages with Open Source communities. Hence, given the current climate, I don’t think I could publish it without addressing the elephant in the room first anyway.

So here it is, I am deeply convinced that contrary to what has been alleged recently, Shopify has nothing but good intentions toward Ruby and its community.

It is healthy to be skeptical toward corporations, I certainly am, but I believe Shopify is currently receiving undue distrust considering their track record of massive investment in the Ruby ecosystem. And some of that may be due to a lack of understanding of how they engage with Open Source communities.

So I’ll try to explain what they do, how they do it, and why we need more companies like Shopify, not less.

## Proper Disclaimer

As is customary in this sort of situation, I first need to disclose the nature of my relationship with Shopify.

I could try to brush it off by just saying that I was employed by them from November 2013 to August 2025, but in my opinion, that would be a cop-out. Knowing that someone has been previously employed by someone else doesn’t tell you anything about where they’re speaking from. Worse, instead of enlightening you on which biases the author might have, it might let you think they have insider knowledge, hence are even more reliable.

What is important to disclose is how the relationship ended.

In my case, I left Shopify for several reasons, but mainly because of my constant friction with the CEO. Ever since my first interaction with him twelve years ago, I knew he was someone I couldn’t see eye to eye with on almost every subject. Even when I’d occasionally happen to agree on a specific topic, his overly maximalist position and lack of nuance would drive me away. The only reason I managed to stick this long at the company is that I made sure to pick projects and teams so as to minimize my interactions with him.

And the reason why I ended up quitting is that it was no longer possible to avoid him. Since I consider him directly responsible for my burnout last year, I couldn’t possibly stay any longer.

I could go on for hours about all the hard feelings, but this is not really the place, I only mean to share enough to explain where I’m speaking from. What is important to know is that I have absolutely zero reasons to give a pass to Shopify over anything.

## People Are Multidimensional

But despites my personal feelings and history, it has to be said that Shopify’s CEO is a Rubyist at heart, almost to a fault.

Contrary to what you might think, Ruby isn’t all that popular at Shopify. Even when I started back in 2013, only a small fraction of new hires had any prior experience with Ruby, and a decade later, there aren’t so many proud Rubyists in the Shopify ranks. Most developers, and even many executives, would rather use something else.

Yet, Ruby and Rails remain the default stack at Shopify, and the only reason for that is the CEO. Every Shopify employee knows that suggesting straying away from Ruby wouldn’t fly there. And I’m convinced that if it were anyone else at the helm, Shopify would have joined the long list of companies that attempted to migrate to something else and are now stuck with both a Ruby monolith and a ton of half-migrated micro-services in Java or Go.

Hence, it’s important to recognize that people are multidimensional. Just because you can’t see eye to eye on some topic doesn’t mean you can’t be allies (even if only by circumstances) on another.

But Shopify isn’t only its CEO.

## The Ruby & Rails Infrastructure Team (R&RI)

As Rubyists, the side of Shopify you are the most likely to interact with, or at least be familiar with, is the Ruby and Rails Infrastructure team (R&RI).

It’s a team of 40ish people. They’re the ones you see on countless GitHub issues and pull requests, maintaining countless projects, and speaking at conferences. I know all of them very well, and I can attest that, barring a couple of rare exceptions, they’re all long-time proud Rubyists, not mercenaries nor zealous “company men”.

I believe, without the shadow of a doubt, that if Shopify ever started to have ill intentions toward the community, many people in the R&RI team would either resign or call it out or both. At the very least, they would confide in other members of the community, and that would inevitably be public rather quickly.

You may think I’m exaggerating, and surely with their cushy salaries, many of them would have second thoughts. But I honestly don’t think so. Shopify isn’t even paying that well (depending on the market). Based on my discussions with the people who left the team over the years, the most common cause of voluntary departures, by far, was compensation. And most of the team could find another job rather quickly anyway, even in this market.

What makes them stay at Shopify, and why it took me so long to finally decide to quit, is that right now, it is hands down the best place in the world to contribute to the Ruby ecosystem. Nowhere else comes close, and that’s all due to Shopify’s philosophy toward Open Source.

## Your Dependencies Are Your Code Too

Whether you realized it already or not, all the code you depend on, all the code that runs on your servers, is your code. It doesn’t matter if it was written by someone you never met in Nebraska, or by a multi-billion-dollar corporation.

You run it, you own it.

If it has a bug, if it is missing a feature, or if it has any other needs, that’s on you to figure out the solution for yourself. There’s no relying on the original author to get that responsibility off your plate.

To illustrate this, I remember back in 2014 or 2015, when MySQL servers started segfaulting in production at a regular interval. IIRC, Shopify had a support contract with a MySQL consultancy, and they probably were notified of it. But we didn’t sit there waiting for the “owners” or experts to figure it out.

It’s a colleague who went knee deep in core dumps to figure out this was caused by an `alloca` call in a non-leaf function, causing a stack overflow, produced a patch, patched our MySQL servers, and then sent the patch upstream[1][1].

This philosophy is at the heart of Shopify’s Ruby & Rails Infrastructure team. It is determined not just to be a user of the open source ecosystem, but to proactively engage with it, contribute, and make it better through engineering time and contributions. Not by delegating the responsibility to a third party nor exploiting maintainers goodwill.

I also sometimes hear people saying that Shopify is snatching all the super senior Ruby developers, but I’d argue that’s mostly untrue. The reality is that in most cases, Shopify is growing these developers internally.

Take Kevin Newton, for instance. He started as a product developer at Shopify, but after a few years, he pitched his vision of a universal parser for Ruby, managed to get transferred to the R&RI team, worked on the project that became Prism, became a Ruby core committer, won the Ruby Prize award, etc. Since then, he left Shopify to work on a Python JIT at Meta, yet he is still maintaining Prism, because he is a Rubyist at heart. And Kevin is far from the only example of that; I am one as well, and so are dozens of my former teammates. Some, like Peter Zhu, even started as interns.

The reason I’m explaining this is that I feel there is a part of the community that is naturally distrustful of Shopify or corporations in general. I don’t blame them, there have been countless examples of nefarious behaviours from companies, so it’s logical and healthy to at least be skeptical. But it’s also important to recognize and salute positive behavior when it happens.

In this specific case, I believe that recently, Shopify has been giving the community something that is priceless: a large number of proficient and deeply committed contributors to Ruby itself and the whole ecosystem. And I’d argue that is way more valuable for the future and sustainability of Ruby than any amount of money.

## Sustainability Isn’t Just About Money

Usually, when the topic of Open Source sustainability comes up, it ends up revolving around how to make companies pay for developers’ time. There is this idealized image of Open Source being an amalgamation of lone developers tirelessly maintaining projects for free, eating ramen while big bad companies make huge profits out of their work. There is definitely some truth to it, it is far from uncommon, but it’s also a bit of a tired cliché.

The Open Source ecosystem is also a lot of projects that are contributed to by people on various companies’ payrolls. Linux is the poster child of healthy corporate involvement, with the overwhelming majority of contributions coming from employees of companies with a vested interest in the kernel. That’s just one example, but when you look at big and complex open source projects, most of the time you’ll see big companies involved in one way or another. That’s how most of the sustainable open source happens today, way more than through donations.

Hence, I’d argue that if an open source community wants to be sustainable, it needs to be welcoming of corporate contributions. I don’t mean trust them blindly, it’s important to keep them in check just in case, but you have to let them play ball.

Ruby has successfully done that. Back in 2019, Rafael França and Matz met in Bristol. Rafael asked Matz what he needed, and Matz answered: “I need people”. That’s how the Ruby and Rails Infrastructure team started getting involved in Ruby development, that’s what ultimately led to YJIT, now ZJIT, numerous GC improvements like Variable Width Allocation, modular GC, Prism, tons of Ractors improvements, etc. But more importantly, almost a dozen new Ruby core committers.

I would wager that if that day Matz had asked for money, we’d have much worse results to show for.

## Money Can Create Perverse Incentives

And aside from worse results, I’d argue it would have created perverse incentives.

I have nothing but respect for people who try to find ways to fund open source development in alternative ways. However, it’s important to look at it through the lens of structures and incentives.

Whenever you design a system that involves people, you need to consider how a person who tries to maximize their personal benefits is incentivized to behave.

A typical example is ticket inspectors on trains and buses. You may be tempted to give them a cut on the fines they give to people, as to incentivise them to work harder, but by doing so, you create a problem that they are incentivized to be inflexible with commuters, causing a lot of conflicts instead of resolving situations peacefully. Some of them might even be incentivized to give bullshit fines to earn a little extra money[2][2].

If a system requires all the people involved to be perfect and act selflessly, then I’d argue it’s a flawed system.

Now, if Shopify had instead poured millions in cash into the Ruby Association or Matz himself, how would you, I, or anyone be able to trust that the project direction and decision are free of influence? How to trust that a given feature was accepted solely on its own merit and not just because it came from a big sponsor? Inversely, when a feature is declined, how do you trust it wasn’t because it didn’t come from a sponsor?

That’s the thing with money, once you have it, it’s very difficult to do without. When a big sponsor pulls out, you have to lay off staff, stop some initiatives, etc. So even if you publicly declare that there’s no strings attached, even if you never explicitly say anything about it, entities and people who receive funding are naturally incentivized to keep the donor happy so that the funding keeps coming.

Whereas with corporate contributors, sure, their employer may decide to assign them to another project, but there are no hard consequences, and most of them will stick around regardless. Most will even remain contributors if they quit or are laid off.

You can actually witness that dynamic between Shopify and Ruby publicly, for instance, in how Prism is now the default parser, but isn’t yet the only official parser. I can tell you that this has ruffled quite a few feathers at Shopify, but that’s the thing, Matz and Ruby don’t feel indebted to Shopify, they feel entirely free to say no. And I think that’s how it should be.

To be clear, I’m not saying open source should be free of any monetary exchanges, just that it’s crucial to do it in a way that doesn’t let these sorts of suspicions arise.

## Not Every Project Is Equally Forkable

I know some people will object to the above, arguing that this is all open source, so if you are not happy with the direction of the project, you can always fork, ergo: shut up! And while this is true in most cases, in practice, there are some projects that aren’t as easily forked because of their position.

For instance, if you look at Sidekiq, it’s making loads of money with its Pro and Enterprise offerings, and quite openly declines some features in the open source project so as not to cannibalize sales. As far as I am aware, pretty much everyone is fine with it. Sure, you’ll find a few people complaining about it, but that’s just background noise.

This is because Sidekiq isn’t on any critical path, there are plenty of alternatives you can go for if you aren’t satisfied with it, and if you wish to fork it and add such a feature for yourself, it’s pretty trivial, you don’t need to convince anyone. Hence, everyone sees it as fair.

However, some projects have a moat. A dominant position granted by another project. Imagine if, instead of allowing you to use any job processor you want through Active Job, Rails had instead decided to make Sidekiq the only option. In such a world, then I believe a whole lot more people would be upset or suspicious, because the bar to clear to use an alternative would be way higher. A lot of Rails users would feel captive.

Well, I would argue that rubygems is in such a situation. It is distributed with Ruby, required early during the Ruby boot process, is coupled with all distributed gems via the `gemspec` format, etc. Because of this, it has a massive moat. Forking it to build and use your own alternative to it is hardly viable, even for a big team like Shopify’s Ruby and Rails Infrastructure team.

As such, while it’s still nothing but commendable to try to fund its maintenance work, you have to be careful to avoid any perverse incentives and conflicts of interest. Otherwise, even if you are exceptionally selfless and well-intentioned, you will inevitably spur suspicion whenever you refuse contributions or ask for sponsorship on a GitHub issue.

Unfortunately, it did happen.

Over the past decade, people in the community, not just Shopify employees, started to conclude that rubygems and bundler were being monetized by some key maintainers. To be clear, I’m not trying to convince anyone that this was actually the case. Some of that dirty laundry that has been an open secret among the Ruby maintainers’ community for a long time has recently been aired out, and I suspect there’s more to come. You are free to form your own opinion on the topic if you so wish.

But my point is that it doesn’t actually matter whether rubygems was actually being unduly taken advantage of or not. Ultimately, it’s down to who and what you consider legitimate.

My point is that the economic model chosen to fund rubygems’ maintenance, combined with its critical position in the ecosystem, has allowed for these suspicions to exist and persist, creating tensions and driving potential sources of funding away.

Again, I believe the problem is with structures and incentives, as well as optics, not specific people being imperfect or ill-intentioned.

## Shopify and Rubygems Rocky Relationship

Because of this, the relationship between Shopify and the various entities overseeing rubygems development has been quite rocky for a long time.

As you are probably aware, supply chain security has been a hot topic in the corporate world, hence, around 2021, Shopify started trying to contribute more to rubygems, and an entire team of developers was assembled with the goal of helping the upstream projects.

I no longer have access to all the history, and some details are now blurry. But from what I recall, there were various goals, such as requiring multi-factor authentication to publish the most popular packages, making code signing easier, and a few other topics.

However, that initiative didn’t exactly receive a warm welcome from upstream. It’s not that these features weren’t desired, but the understanding on Shopify’s side was that maintainers preferred to be paid to do it, rather than just accept contributions.

This is what ultimately led to Shopify funding Ruby Central directly (other than being a recurring major sponsor at their conferences for years). The deal was for [one million dollars over 4 years, under the name Ruby Shield][3].

But even after that, the feeling on the Shopify side was that upstream was still uncooperative, until ultimately they decided to cut their losses and re-assigned engineers elsewhere. The 4-year funding deal remained, but not much was expected of it.

Shopify could have threatened to pull funding at that time to try to coerce Ruby Central, yet they didn’t.

## Shopify Never Threatened To Pull Funding

As I said earlier, ever since this controversy started, I’ve been unconvinced by the theory that all this would have been orchestrated by Shopify or through Shopify. That simply would have required involving too many people, and I absolutely can’t imagine that none of them would have objected in one way or another.

But anyway, since then, I did contact two former coworkers, and they both assured me that Shopify never threatened to pull Ruby Central’s funding, nor threatened not to renew it.

Now, as I tried to explain earlier, even if you loudly claim money comes with no strings attached, people and entities are naturally incentivized to do what they think is necessary to keep it coming. As such, it’s entirely possible that despite the absence of threats, Ruby Central’s moves may have been motivated by the need to secure the existing funding and/or find additional sources of funding.

My former coworkers also told me their side of the story, and it’s absolutely nothing like what has been alleged so far. I deeply trust these two people, and I can’t possibly imagine they’d be lying to me, but I’d understand if you don’t want to take my word for it.

I don’t know when their side of the story will come out, nor if it will come out at all, but I do hope it comes out soon and with receipts. Seeing so many good-natured and well-intentioned people get demonized like they have been over the last few weeks is depressing.

It is undeniable that, regardless of what Ruby Central’s intentions were, the communication and execution have been abysmal. It is also true that there is a deep disagreement about what they rightfully or legitimately owned that won’t easily be resolved. However, I can’t believe the entire organisation was ill-intentioned, here again, that would involve too many people to be conceivable.

Similarly, the claim that Aaron sending patches to rubygems is a clue that there was a conspiracy at play drives me nuts. I’ve seen these pull requests being made with my own eyes, and I can tell you that the reason is way more mundane than that. We were at Rails World, someone mentioned `rv`, the question of why you’d need to write something in Rust to speed up gem installation was raised, and Aaron and a few others started to profile Bundler to see if it could be made faster.

That’s it, that’s all there is. Aaron got nerd sniped into making Bundler faster, and now he’s being called out for supposedly being part of a hostile takeover? Give me a break.

## We Need More Shopifies, Not Less

I think it’s healthy to be wary of Shopify’s huge footprint on the ecosystem. Companies are fickle beings, and even if I’m not particularly concerned about them ever having ill intent toward the Ruby ecosystem, it’s not impossible that in the future they may decide to invest less.

But the response shouldn’t be to try to cast Shopify and its employees aside. It would be silly to punish them for helping too much. What we need is more companies doing their part. Both to reduce Shopify’s relative influence, but also to have more diverse perspectives, use cases, and priorities.

I’m not saying every company should have a team as big as Shopify’s R&RI, but there are numerous Ruby-based companies with valuations in billions and several hundred developers on their payroll, yet they contribute very little upstream. If you work at one of such companies, you should really consider how you could do more.

That’s what I intend to do at my next job, to get one more Ruby company to pull its weight.

1.  For the annecdote, MySQL refused the patch arguing that this error couldn’t realistically happen. LOL. [↩][4]
    
2.  This is not a made up example by the way. It has been a big issue in France for over a decade. [↩][5]
    

[][6]

[1]: #fn:1
[2]: #fn:2
[3]: https://rubycentral.org/news/ruby-shield/
[4]: #fnref:1
[5]: #fnref:2
[6]: /opensource/ruby/2025/10/09/dear-rubyists.html