---
title: Engineering for Bounded Cognition
date: 2026-02-03T00:00:00.000Z
authorURL: ""
originalURL: https://shapeofthesystem.com/posts/2026/02/03/bounded-cognition
translator: ""
reviewer: ""
---

# Engineering for Bounded Cognition

<!-- more -->

The mind that builds software is far smaller than the software it builds, and almost everything good in engineering is a way of living with that gap.

[cross-cutting][1]

You've probably heard that the mind can hold seven things at once. It's the most repeated figure in popular psychology, and the man who came up with it didn't actually believe it. George Miller called his 1956 paper "The Magical Number Seven, Plus or Minus Two" half as a joke. He wrote that the way the number kept showing up across experiments that had nothing to do with each other was probably just a coincidence, and at one point he said he felt persecuted by an integer. He was right to be suspicious. Later researchers stripped away the little tricks we use to cheat - rehearsing under your breath, grouping digits into chunks - and the number fell. When you take all that away, the honest figure for how many separate things a person can hold in mind at once, with no help at all, is about four.

Four. Now set that against the device you're reading this on, which is running tens of millions of lines of code. The whole problem of software is sitting right there in that one ratio. The systems we build are some of the most intricate objects anyone has ever made, and they get built, and changed, and dragged back from the brink under pressure, by minds that can keep about four things straight at a time.

And it's worse than just being small. It's narrow too. The attention you point at those four slots isn't a floodlight thrown across a room, it's a torch beam in a dark warehouse, and almost everything is outside it. There's a famous experiment where people were asked to watch a video and count how many times a team passed a basketball. Halfway through, someone in a gorilla suit walks into the middle of the shot, turns to face the camera, beats their chest, and wanders off again. Nine seconds, in plain view. About half the people watching never see it. They're staring right at it. They aren't blind. The beam was just pointed at the ball. And it gets stranger than that. In another study people were stopped in the street and asked for directions, and while they were in the middle of answering, two workers carried a door between them, and behind that door the person who'd asked the question was swapped for someone else. Different height, different shirt, different voice. About half the time the person carries on giving directions to a complete stranger and never notices that the human being in front of them has been replaced. And even the little you do hold, you only hold for a few seconds. Show someone three random letters, stop them from rehearsing, and within twenty seconds most of it has gone. So that's four slots, a narrow beam, and a leak. That's the instrument we build software with.

Once you've felt how small the mind really is, the surprising thing about software stops being that it breaks. The surprising thing is that it works at all. We've put up these cathedrals of logic that are far too big for any one person to hold in their head, and somehow we keep them standing with the same four-slot, gorilla-missing minds we were handed at birth. The uptime is the miracle.

This puts a phrase the industry loves into a much harsher light. Human error. Read almost any incident report and there's a person at the end of it who typed the wrong command, or clicked the wrong button, or didn't see the warning, and we file the whole thing under their carelessness and promise to try harder next time. But a warning that half of attentive people will look straight at and not see isn't really a warning. It's a decoration. A button that sits one tired click away from disaster was put there by a designer, not by the person who happened to click it. The people who study this for a living, in aviation and medicine and the control rooms of power stations, where "just be more careful" is the sort of advice that gets people killed, worked this out a long time ago. When a capable person makes the same mistake over and over, the fault is hardly ever in the person. It's in a system that was built for an operator who doesn't exist. One who never gets tired, never looks away, and holds the whole machine in their head at once. That operator was never born. Every system built for them is already broken. It just hasn't had its bad day yet.

For a while you could comfort yourself with the idea that the machines would save us from all this, that a computer at least didn't have a four-slot mind. Then we built machines that read and write language, and we handed them our code, and we watched them fail in a way that felt uncomfortably familiar. A large language model has what's called a context window, which is a hard limit on how much it can take into account at any one time. It behaves less like long-term memory and more like the thing you're holding in your mind right this second, and the moment something drops out of the window it isn't dimly remembered, it's just gone. Even inside the window the model doesn't pay attention evenly. There's a study with the very flat title "Lost in the Middle", and the researchers found that these systems answer well when the fact they need is near the start or the end of a long input, and noticeably worse when it's buried in the middle. Which is exactly the shape of a tired person skimming a document, catching the opening and the conclusion and going a bit glassy in between. Give the model more to read and it can actually do worse rather than better. In some of the tests, handing it more retrieved documents pulled its accuracy down below what it managed with none at all. The model's attention is a fixed quantity, and it has to add up to one, so the more things you make it look at, the less of that attention any single earlier thing can keep. Spread it thin enough and the instruction you gave right at the top of the session goes quiet, and the assistant writes a second copy of a function it already wrote, or it breaks some decision the two of you made half an hour back. It's lost the thread. The same boundedness, only now it's running in silicon, and it goes wrong in the exact way you do on your own worst afternoons.

This is true for the novice and the expert, for the well-rested and the wrung-out, for the human and the machine. The mind that changes the system is always far smaller than the system itself. There's no version of this you grow out of by being clever, or by hiring clever people, or by buying a bigger model, because the gap isn't a few orders of magnitude that you might one day close. It's the permanent condition of the work. So the central question of engineering isn't "how do we find a mind big enough to hold all of this", because there isn't one. The question is "how do we shape the thing so that a small mind can work on it without bringing it all down". Nearly everything we call good engineering is some answer to that. Give something a precise name and that's one more fact you no longer have to keep in your head. Draw a boundary and now you've got a promise you can stop re-checking. Write a test and you've parked a decision somewhere it can't fade on you. And anything you can undo is permission to be wrong. Each of these takes a thing that would otherwise have to live in a fragile four-slot mind and moves it out into the structure, where it stays put while you blink.

There's a quiet bonus in building this way, and the cleanest proof of it is sitting in a kitchen drawer. Back in 1990 a man called Sam Farber watched his wife Betsey struggling with an ordinary vegetable peeler, the thin metal handle digging into fingers that arthritis had stiffened up. He decided to make her a better one. The design they landed on was a soft, yielding grip that a painful hand could hold without having to grip hard, and it went on to become one of the best-selling kitchen tools in the world, bought by millions of people who didn't have arthritis at all, because a handle built for the hand that struggles turns out to be better for every hand. Designing for the most constrained user isn't some charity that the rest of us put up with. More often than not it just gives you a better thing. And the same goes for what we build out of code. Make a system safe for the tired engineer, and the distracted one, and the newcomer who knows nothing yet, and the machine that keeps losing the thread, and you haven't lowered some ceiling for the gifted. You've built the thing every one of them reaches for when the beam goes narrow, which it always does eventually.

None of this is a trick you apply once and then move on from. It's a habit of mind. It's a way of treating your own attention as the scarce and unreliable thing it actually is, and then building accordingly. I've spent a long time trying to turn that habit into specific moves, the concrete things you do to a system so that a small mind can change it safely, and they ended up as a manifesto. This essay is the idea sitting underneath the whole thing. If any of it rings true, if you've ever looked straight at the gorilla and missed it, then the rest is only the working out.

---

In the manifesto, this is the preamble: the whole thing is one long answer to it.

Sources

-   **\[Cowan 2001\]** Nelson Cowan, "The Magical Number 4 in Short-Term Memory: A Reconsideration of Mental Storage Capacity". Behavioral and Brain Sciences 24(1), 2001. [https://doi.org/10.1017/S0140525X01003922][2]. The working-memory limit is about four chunks, not seven; tenet [PREAMBLE][3].
-   **\[Liu et al. 2023\]** Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni & Percy Liang, "Lost in the Middle: How Language Models Use Long Contexts". arXiv:2307.03172, 2023 (later TACL vol. 12, 2024). [https://arxiv.org/abs/2307.03172][4]. Models attend to the ends and lose the middle of long contexts; tenet [PREAMBLE][5].
-   **\[Miller 1956\]** George A. Miller, "The Magical Number Seven, Plus or Minus Two". Psychological Review 63(2), 1956. [https://doi.org/10.1037/h0043158][6]. Working-memory limit behind 'what an engineer must hold in their head'; tenet [PREAMBLE][7].
-   **\[OXO Good Grips 1990\]** OXO Good Grips, founded by Sam Farber after watching his wife Betsey struggle with a peeler, designed with Smart Design. 1990. [https://en.wikipedia.org/wiki/Sam\_Farber][8]. Designing for the constrained case makes the tool better for everyone; tenet [PREAMBLE][9].
-   **\[Peterson & Peterson 1959\]** Lloyd Peterson & Margaret Peterson, "Short-Term Retention of Individual Verbal Items". Journal of Experimental Psychology 58(3), 1959. [https://doi.org/10.1037/h0049234][10]. Unrehearsed items decay within seconds; tenet [PREAMBLE][11].
-   **\[Simons & Chabris 1999\]** Daniel Simons & Christopher Chabris, "Gorillas in Our Midst: Sustained Inattentional Blindness for Dynamic Events". Perception 28(9), 1999. [https://doi.org/10.1068/p281059][12]. Focused attention makes the obvious invisible; tenet [PREAMBLE][13].
-   **\[Simons & Levin 1998\]** Daniel Simons & Daniel Levin, "Failure to Detect Changes to People During a Real-World Interaction" (the 'door study'). Psychonomic Bulletin & Review 5(4), 1998. [https://doi.org/10.3758/BF03208840][14]. Change blindness: half miss a swapped conversation partner; tenet [PREAMBLE][15].

_This is the root of a series of field notes on building software for the way minds actually work: tired, distractible, ordinary, and now partly machine. The full set of moves is the manifesto they come from, [The Shape of the System][16]._

[1]: ../../../cross-cutting/index.html
[2]: https://doi.org/10.1017/S0140525X01003922
[3]: ../../../../index.html#preamble
[4]: https://arxiv.org/abs/2307.03172
[5]: ../../../../index.html#preamble
[6]: https://doi.org/10.1037/h0043158
[7]: ../../../../index.html#preamble
[8]: https://en.wikipedia.org/wiki/Sam_Farber
[9]: ../../../../index.html#preamble
[10]: https://doi.org/10.1037/h0049234
[11]: ../../../../index.html#preamble
[12]: https://doi.org/10.1068/p281059
[13]: ../../../../index.html#preamble
[14]: https://doi.org/10.3758/BF03208840
[15]: ../../../../index.html#preamble
[16]: https://shapeofthesystem.com