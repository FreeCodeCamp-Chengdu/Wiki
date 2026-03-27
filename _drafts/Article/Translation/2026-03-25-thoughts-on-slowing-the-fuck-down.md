---
title: Thoughts on slowing the fuck down
date: 2026-03-27T03:53:47.792Z
authorURL: ""
originalURL: https://mariozechner.at/posts/2026-03-25-thoughts-on-slowing-the-fuck-down/
translator: ""
reviewer: ""
---

# Thoughts on slowing the fuck down

<!-- more -->

2026-03-25

![](/media/header.png)

The turtle's face is me looking at our industry

# Table of contents

-   [Everything is broken][1]
-   [How we should not work with agents and why][2]
    -   [Compounding booboos with zero learning, no bottlenecks, and delayed pain][3]
    -   [Merchants of learned complexity][4]
    -   [Agentic search has low recall][5]
-   [How we should work with agents (for now, I think)][6]

It's been about a year since coding agents appeared on the scene that could actually build you full projects. There were precursors like Aider and early Cursor, but they were more assistant than agent. The new generation is enticing, and a lot of us have spent a lot of free time building all the projects we always wanted to build but never had time to.

And I think that's fine. Spending your free time building things is super enjoyable, and most of the time you don't really have to care about code quality and maintainability. It also gives you a way to learn a new tech stack if you so want.

During the Christmas break, both Anthropic and OpenAI handed out some freebies to hook people to their addictive slot machines. For many, it was the first time they experienced the magic of agentic coding. The fold's getting bigger.

Coding agents are now also introduced to production codebases. After 12 months, we are now beginning to see the effects of all that "progress". Here's my current view.

## Everything is broken

While all of this is anecdotal, it sure feels like software has become a brittle mess, with 98% uptime becoming the norm instead of the exception, including for big services. And user interfaces have the weirdest fucking bugs that you'd think a QA team would catch. I give you that that's been the case for longer than agents exist. But we seem to be accelerating.

We don't have access to the internals of companies. But every now and then something slips through to some news reporter. Like this supposed [AI caused outage at AWS][7]. Which AWS immediately ["corrected"][8]. Only to then follow up internally with a [90-day reset][9].

Satya Nadella, the CEO of Microsoft, has been going on about [how much code is now being written by AI][10] at Microsoft. While we don't have direct evidence, there sure is a feeling that Windows is going down the shitter. Microsoft itself seems to agree, based on this fine [blog post][11].

Companies claiming 100% of their product's code is now written by AI consistently put out the worst garbage you can imagine. Not pointing fingers, but memory leaks in the gigabytes, UI glitches, broken-ass features, crashes: that is not the seal of quality they think it is. And it's definitely not good advertising for the fever dream of having your agents do all the work for you.

Through the grapevine you hear more and more people, from software companies small and large, saying they have agentically coded themselves into a corner. No code review, design decisions delegated to the agent, a gazillion features nobody asked for. That'll do it.

## How we should not work with agents and why

We have basically given up all discipline and agency for a sort of addiction, where your highest goal is to produce the largest amount of code in the shortest amount of time. Consequences be damned.

You're building an orchestration layer to command an army of autonomous agents. You installed Beads, completely oblivious to the fact that it's basically uninstallable malware. The internet told you to. That's how you should work or you're ngmi. You're ralphing the loop. Look, Anthropic built a C compiler with an agent swarm. It's kind of broken, but surely the next generation of LLMs can fix it. Oh my god, Cursor built a browser with a battalion of agents. Yes, of course, it's not really working and it needed a human to spin the wheel a little bit every now and then. But surely the next generation of LLMs will fix it. Pinky promise! Distribute, divide and conquer, autonomy, dark factories, software is solved in the next 6 months. SaaS is dead, my grandma just had her Claw build her own Shopify!

Now again, this can work for your side project barely anyone is using, including yourself. And hey, maybe there's somebody out there who can actually make this work for a software product that's not a steaming pile of garbage and is used by actual humans in anger.

If that's you, more power to you. But at least among my circle of peers I have yet to find evidence that this kind of shit works. Maybe we all have skill issues.

### Compounding booboos with zero learning, no bottlenecks, and delayed pain

The problem with agents is that they make errors. Which is fine, humans also make errors. Maybe they are just correctness errors. Easy to identify and fix. Add a regression test on top for bonus points. Or maybe it's a code smell your linter doesn't catch. A useless method here, a type that doesn't make sense, duplicated code over there. On their own, these are harmless. A human will also do such booboos.

But clankers aren't humans. A human makes the same error a few times. Eventually they learn not to make it again. Either because someone starts screaming at them or because they're on a genuine learning path.

An agent has no such learning ability. At least not out of the box. It will continue making the same errors over and over again. Depending on the training data it might also come up with glorious new interpolations of different errors.

Now you can try to teach your agent. Tell it to not make that booboo again in your AGENTS.md. Concoct the most complex memory system and have it look up previous errors and best practices. And that can be effective for a specific category of errors. But it also requires you to actually observe the agent making that error.

There's a much more important difference between clanker and human. A human is a bottleneck. A human cannot shit out 20,000 lines of code in a few hours. Even if the human creates such booboos at high frequency, there's only so many booboos the human can introduce in a codebase per day. The booboos will compound at a very slow rate. Usually, if the booboo pain gets too big, the human, who hates pain, will spend some time fixing up the booboos. Or the human gets fired and someone else fixes up the booboos. So the pain goes away.

With an orchestrated army of agents, there is no bottleneck, no human pain. These tiny little harmless booboos suddenly compound at a rate that's unsustainable. You have removed yourself from the loop, so you don't even know that all the innocent booboos have formed a monster of a codebase. You only feel the pain when it's too late.

Then one day you turn around and want to add a new feature. But the architecture, which is largely booboos at this point, doesn't allow your army of agents to make the change in a functioning way. Or your users are screaming at you because something in the latest release broke and deleted some user data.

You realize you can no longer trust the codebase. Worse, you realize that the gazillions of unit, snapshot, and e2e tests you had your clankers write are equally untrustworthy. The only thing that's still a reliable measure of "does this work" is manually testing the product. Congrats, you fucked yourself (and your company).

### Merchants of learned complexity

You have zero fucking idea what's going on because you delegated all your agency to your agents. You let them run free, and they are merchants of complexity. They have seen many bad architectural decisions in their training data and throughout their RL training. You have told them to architect your application. Guess what the result is?

An immense amount of complexity, an amalgam of terrible cargo cult "industry best practices", that you didn't rein in before it was too late. But it's worse than that.

Your agents never see each other's runs, never get to see all of your codebase, never get to see all the decisions that were made by you or other agents before they make a change. As such, an agent's decisions are always local, which leads to the exact booboos described above. Immense amounts of code duplication, abstractions for abstractions' sake.

All of this compounds into an unrecoverable mess of complexity. The exact same mess you find in human-made enterprise codebases. Those arrive at that state because the pain is distributed over a massive amount of people. The individual suffering doesn't pass the threshold of "I need to fix this". The individual might not even have the means to fix things. And organizations have super high pain tolerance. But human-made enterprise codebases take years to get there. The organization slowly evolves along with the complexity in a demented kind of synergy and learns how to deal with it.

With agents and a team of 2 humans, you can get to that complexity within weeks.

### Agentic search has low recall

So now you hope your agents can fix the mess, refactor it, make it pristine. But your agents can also no longer deal with it. Because the codebase and complexity are too big, and they only ever have a local view of the mess.

And I'm not just talking about context window size or long context attention mechanisms failing at the sight of a 1 million lines of code monster. Those are obvious technical limitations. It's more devious than that.

Before your agent can try and help fix the mess, it needs to find all the code that needs changing and all existing code it can reuse. We call that agentic search. How the agent does that depends on the tools it has. You can give it a Bash tool so it can ripgrep its way through the codebase. You can give it some queryable codebase index, an LSP server, a vector database. In the end it doesn't matter much. The bigger the codebase, the lower the recall. Low recall means that your agent will, in fact, not find all the code it needs to do a good job.

This is also why those code smell booboos happen in the first place. The agent misses existing code, duplicates things, introduces inconsistencies. And then they blossom into a beautiful shit flower of complexity.

How do we avoid all of this?

## How we should work with agents (for now, I think)

Coding agents are sirens, luring you in with their speed of code generation and jagged intelligence, often completing a simple task with high quality at breakneck velocity. Things start falling apart when you think: "Oh golly, this thing is great. Computer, do my work!".

There's nothing wrong with delegating tasks to agents, obviously. Good agent tasks share a few properties: they can be scoped so the agent doesn't need to understand the full system. The loop can be closed, that is, the agent has a way to evaluate its own work. The output isn't mission critical, just some ad hoc tool or internal piece of software nobody's life or revenue depends on. Or you just need a rubber duck to bounce ideas against, which basically means bouncing your idea against the compressed wisdom of the internet and synthetic training data. If any of that applies, you found the perfect task for the agent, provided that you as the human are the final quality gate.

[Karpathy's][12] [auto-research][13] applied to [speeding up startup time of your app][14]? Great! As long as you understand that the code it spits out is not production-ready at all. Auto-research works because you give it an evaluation function that lets the agent measure its work against some metric, like startup time or loss. But that evaluation function only captures a very narrow metric. The agent will happily ignore any metrics not captured by the evaluation function, such as code quality, complexity, or even correctness, if your evaluation function is foobar.

The point is: let the agent do the boring stuff, the stuff that won't teach you anything new, or try out different things you'd otherwise not have time for. Then you evaluate what it came up with, take the ideas that are actually reasonable and correct, and finalize the implementation. Yes, sure, you can also use an agent for that final step.

And I would like to suggest that slowing the fuck down is the way to go. Give yourself time to think about what you're actually building and why. Give yourself an opportunity to say, fuck no, we don't need this. Set yourself limits on how much code you let the clanker generate per day, in line with your ability to actually review the code.

Anything that defines the gestalt of your system, that is architecture, API, and so on, write it by hand. Maybe use tab completion for some nostalgic feels. Or do some pair programming with your agent. Be in the code. Because the simple act of having to write the thing or seeing it being built up step by step introduces friction that allows you to better understand what you want to build and how the system "feels". This is where your experience and taste come in, something the current SOTA models simply cannot yet replace. And slowing the fuck down and suffering some friction is what allows you to learn and grow.

The end result will be systems and codebases that continue to be maintainable, at least as maintainable as our old systems before agents. Yes, those were not perfect either. Your users will thank you, as your product now sparks joy instead of slop. You'll build fewer features, but the right ones. Learning to say no is a feature in itself.

You can sleep well knowing that you still have an idea what the fuck is going on, and that you have agency. Your understanding allows you to fix the recall problem of agentic search, leading to better clanker outputs that need less massaging. And if shit hits the fan, you are able to go in and fix it. Or if your initial design has been suboptimal, you understand why it's suboptimal, and how to refactor it into something better. With or without an agent, don't fucking care.

All of this requires discipline and agency.

All of this requires humans.

This page respects your privacy by not using cookies or similar technologies and by not collecting any personally identifiable information.

[1]: #toc_0
[2]: #toc_1
[3]: #toc_2
[4]: #toc_3
[5]: #toc_4
[6]: #toc_5
[7]: https://www.ft.com/content/00c282de-ed14-4acd-a948-bc8d6bdb339d
[8]: https://www.aboutamazon.com/news/aws/aws-service-outage-ai-bot-kiro
[9]: https://www.businessinsider.com/amazon-tightens-code-controls-after-outages-including-one-ai-2026-3
[10]: https://techcrunch.com/2025/04/29/microsoft-ceo-says-up-to-30-of-the-companys-code-was-written-by-ai/
[11]: https://blogs.windows.com/windows-insider/2026/03/20/our-commitment-to-windows-quality/
[12]: https://x.com/karpathy
[13]: https://github.com/karpathy/autoresearch
[14]: https://x.com/badlogicgames/status/2035469013480869912