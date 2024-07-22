---
title: COMMUNICATION TIPS FOR REMOTE WORK
date: 2024-07-22T02:02:06.422Z
authorURL: ""
originalURL: https://thorstenball.com/remote-communication/
translator: ""
reviewer: ""
---

# COMMUNICATION TIPS FOR REMOTE WORK

<!-- more -->

# \==================================

_Author: [Thorsten Ball][1]_

## Goals

## —————

1.  **Nobody is blocked due to lack of input from others**. We want to reduce or eliminate the time our teammates wait for others.
2.  **Provide clarity of action**. We want to avoid teammates asking themselves what they should do next, what we meant when we said something, what we want to do next, what we expect them to do next, or what the next steps are they could choose to take.

## Assumptions

## —————

1.  Team is remote. No office.
2.  Team is distributed, across the world and multiple time zones.

1 + 2 equals “high latency”. That means communication is not cheap. There is no “Walk over to bob and ask him”. It’s “Anna lives on the US west coast and will come online in ~9 hours”.

## How do we achieve these goals?

## —————

1.  **Leverage time zones**. Time zones are nasty and nobody likes them, least of all programmers. But I think there is a way to leverage time zones and that is: let everyone work while you sleep soundly. Sounds as good as that sit-on-the-couch-and-get-abs-trainer commercial, right? It’s even better, but there’s a requirement: **hand off everything with as much context as possible** so that your colleagues don’t need any more input for the next ~12+ hours. That’s hard but it is possible.
    
    _Example_: code reviews. When you leave a code review, leave it in such a way that the author of the PR doesn’t have wait for more input, i.e. write “Approve, but please add a test before merging” or “Request changes, because there’s a bug in X. Once that’s fixed, feel free to merge & deploy”.
    
2.  **Share things while they’re still in work-in-progress**. Yes, they’re not finished, yes, you’d love to add more, but: you can already get feedback and comments while you sleep. If you anticipate that something will drastically change between WIP and Done status: add a little note. “WIP: The ‘proposal’ section is half-finished and the ‘status quo’ section is missing X, Y, and Z. But please feel free to look at the Option1 in ‘proposal’ and leave comments”.
    
3.  **Write out why you _didn’t_ do something**. If you do something (leave a review, merge a PR, deploy code) it’s usually easy to see, but if you don’t do it, nobody can see why.
    
    Say you have a PR that your team wanted to merge for 3 days. It’s now reviewed, build is green — looks like it’s ready to go. But then you discover something that makes you think you should delay merging (I’m making stuff up here, but say another team raised concerns around that area of the code and you want their eyes on it too before merging). **Now you need to write your thought process down**. Just one sentence: “hey, team X needs to look at this, that’s why I’m not merging it. If they give us thumbs-up, feel free to hit merge”. If you don’t, you might log off for the day, your teammate comes online and thinks: “oh, why isn’t this merged?” and hits the big fat button.
    
4.  **Optimistic execution**. You’re done with your task, but you’re not sure how to do your next task. “Do we write docs in this format now? Do I need screenshots?” or “I bet I can hack together something in TypeScript, but I’m not sure whether the CSS is correct”. Instead of waiting: give it a try! Luckily for us, material is cheap in the digital realm. We don’t lose money when we create a UI without CSS. We don’t have to throw it away and start over and pay for another bag of HTML that has been wasted. Give it try, start and assume that what you’ll do will be somewhere in the ballpark and if not, it can be fixed.
    
    Prepare options! Not sure whether your analytics query should include the count of customers or the count of employees at those customers and the only person to tell you is Sophie from the analytics team but she only gets up in 5 hours? It’s usually possible to simply write both queries and present them as options: “Hey Sophie, couldn’t remember exactly what you needed, so here’s 3 options: (1) num of customers (2) total num of employees at all customers (3) both in the same query”.
    
5.  **Never assume** that
    
    (a) somebody else knows what you know if you don’t have confirmation they do
    
    (b) somebody else read something or saw something if you want them to look at it.
    
    The mantra to use here: make the implicit explicit.
    
6.  **Don’t make others hunt for information**
    
    If you’re communicating with someone in medium 1 and say something like “I’ll write that up”: drop a link! Don’t assume the other person scans the rest of your online meeting grounds to where you dropped some information.
    

[1]: https://thorstenball.com