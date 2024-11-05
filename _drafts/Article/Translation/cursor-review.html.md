---
title: "How I write code using Cursor: A review"
date: 2024-11-05T08:50:21.566Z
authorURL: ""
originalURL: https://www.arguingwithalgorithms.com/posts/cursor-review.html
translator: ""
reviewer: ""
---

In forums relating to AI and AI coding in particular, I see a common inquiry from experienced software developers: _Is anyone getting value out of tools like Cursor, and is it worth the subscription price?_

<!-- more -->

A few months into using Cursor as my daily driver for both personal and work projects, I have some observations to share about whether this is a "need-to-have" tool or just a passing fad, as well as strategies to get the most benefit quickly which may help you if you'd like to trial it. Some of you may have tried Cursor and found it underwhelming, and maybe some of these suggestions might inspire you to give it another try.

I am not sponsored by Cursor, and I am not a product reviewer. I am neither championing nor dunking on this as a product, but rather sharing my own experience with it.

**Who am I, and who is the audience for this article?**

I have been writing code for 36 years in a number of languages, but professionally focused on C-heavy computer game engines and Go/Python/JS web development. I am expecting readers to be similarly reasonably comfortable and productive working in large codebases, writing and debugging code in their chosen language, etc. I would give very different advice to novices who might want an AI to teach them programming concepts or write code for them that is way beyond their level!

For me, the appeal of an AI copilot is in taking care of boilerplate and repetitive tasks for me so I can focus on the interesting logic for any given problem. I am also not especially interested in cranking out large quantities of code automatically; I am highly skeptical of "lines of code written" as an efficiency metric. I would prefer to spend less time writing the same amount of code and more time thinking through edge cases, maintainability, etc.

So, without further ado:

## What is Cursor?

Cursor[1][1] is a fork of Visual Studio Code (VS Code) which has Large Language Model (LLM) powered features integrated into the core UI. It is a proprietary product with a free tier and a subscription option; however, the pricing sheet doesn't cover what the actual subscriber benefits are and how they compare to competing products. I'll try to clarify that when discussing the features below based on my own understanding, but a quick summary:

-   **Tab completion**: This is a set of proprietary fine-tuned models that both provide code completion in the editor, as well as navigate to the next recommended action, all triggered by the Tab key. Only available to subscribers.
-   **Inline editing**: This is a chat-based interface for making edits to selected code with a simple diff view using a foundation model such as GPT or Claude. Available to free and paid users.
-   **Chat sidebar**: This is also a chat-based interface for making larger edits in a sidebar view, allowing more room for longer discussion, code sample suggestions across multiple files, etc. using a foundation model such as GPT or Claude. Available to free and paid users.
-   **Composer**: This is yet another chat-based interface specifically meant for larger cross-codebase refactors, generating diffs for multiple files that you can page through and approve, also using a foundation model such as GPT or Claude. Available to free and paid users.

## Tab completion

While other LLM-powered coding tools focus on a chat experience, so far in my usage of Cursor it's the tab completion that fits most naturally into my day-to-day practice of coding and saves the most time. A lot of thought and technical research has apparently gone into this feature, so that it can not only suggest completions for a line, several lines, or a whole function, but it can also suggest the next line to go to for the next edit. What this amounts to is being able to make part of a change, and then auto-complete related changes throughout the entire file just by repeatedly pressing Tab.

One way to use this is as a code refactoring tool on steroids. For example, suppose I have a block of code with variable names in `under_score` notation that I want to convert to `camelCase`. It is sufficient to rename one instance of one variable, and then tab through all the lines that should be updated, including the other related variables. Many tedious, error-prone tasks can be automated in this way without having to write a script to do so:

<video controls=""><source src="../videos/cursor-review/example1.webm" type="video/webm"><p>Your browser doesn't support HTML video. Here is a <a href="../videos/cursor-review/example1.webm" download="example1.webm">link to the video</a> instead.</p></video>

Sometimes tab completion will indepedently find a bug and propose a fix. Many times it will suggest imports when I add a dependency in Python or Go. If I wrap a string in quotes, it will escape the contents appropriately. And, as with other tools, it can write whole functions based on just the function signature and optional docstring:

<video controls=""><source src="../videos/cursor-review/example2.webm" type="video/webm"><p>Your browser doesn't support HTML video. Here is a <a href="../videos/cursor-review/example2.webm" download="example2.webm">link to the video</a> instead.</p></video>

All in all, this tool feels like it is reading my mind, guessing at my next action, and allowing me to think less about the code and more about the architecture of I am building.

Also worth noting: The completions are _incredibly fast_, and I never felt a delay waiting for a suggestion. They appear basically as soon as I stop typing. Having too long a wait would surely be a deal-breaker for me.

So, what are my complaints with Tab completion? One is a minor annoyance: Sometimes I don't see the suggestion in time and continue typing, and the completion disappears. Once it is gone, there doesn't appear to be any way to get it to come back, so I have to type something else and hope.

My other complaint is the exact opposite situation: Sometimes a completion is dead wrong, and I intentionally dismiss it. Subsequently, but very infrequently, I will accept a totally different completion and the previously-declined suggestion will quietly be applied as well. This has already caused some hard-to-track-down bugs because I wasn't aware the wrong logic had been accepted. I haven't found these cases to be frequent enough to cancel out the productivity boost of tab completion, but they do detract from it.

## Inline editing, chat sidebar, and composer

As far as I can tell, these features are all very similar in their interaction with a foundational model - I use Claude 3.5 Sonnet almost exclusively - and the variance is in the user interface.

Inline editing can be invoked by selecting some code and pressing Ctrl-K/Cmd-K. I type in the desired changes, and get a nice diff in the file that I can accept or reject. I use this mostly to implement bits of code inside a function or make minor refactors.

A good example of where this works great is if I have a loop over some tasks and I want to parallelize them:

<video controls=""><source src="../videos/cursor-review/example3.webm" type="video/webm"><p>Your browser doesn't support HTML video. Here is a <a href="../videos/cursor-review/example3.webm" download="example3.webm">link to the video</a> instead.</p></video>

The chat sidebar is opened with Ctrl+L/Cmd+L, and gives more real estate for a multi-turn conversation, though one pet peeve I have with the LLM models I've tested so far is they will _always_ return code first, rather than ask for clarification if there is any ambiguity. The suggested code has an Apply button that will create a diff in the currently selected file. This is useful for larger refactors within a single file, or creating a brand new file based on the file I have open. If additional files are relevant they can be added manually to the context, but Cursor will try to guess which files are relevant based on the query and an index it generates in the background.

Here is an example which takes an application's database API and creates a REST API to access it, with parameter validation and correct HTTP status codes, _then_ writes a client library to access that REST API:

<video controls=""><source src="../videos/cursor-review/example4.webm" type="video/webm"><p>Your browser doesn't support HTML video. Here is a <a href="../videos/cursor-review/example4.webm" download="example4.webm">link to the video</a> instead.</p></video>

As another example, here I am using the chat sidebar to convert the client library from Python to Go. Note how the loosely-typed Python is converted to well-defined struct types and idiomatic Go including error handling! This is not a 1:1 rewrite at all:

<video controls=""><source src="../videos/cursor-review/example5.webm" type="video/webm"><p>Your browser doesn't support HTML video. Here is a <a href="../videos/cursor-review/example5.webm" download="example5.webm">link to the video</a> instead.</p></video>

Finally, Composer is specifically meant for cross-file refactors. This is also the feature I use least, but provides a better user experience for reviewing multiple file diffs one at a time.

## .cursorrules file

I did not realize this feature existed until I came across it in the (in my opinion too minimal) documentation, but the various chat modalities always include the contents of a `.cursorrules` file located at the root of the workspace to provide additional context. I've been experimenting with using this to inform the LLM of the repository's coding standards, common packages, and other documentation.

This feature might help to solve one of the big roadblocks I have observed with Cursor: It does not follow coding styles and patterns unless they already exist in the same file you are editing. For example, at Khan Academy we use a proprietary library [2][7] for passing context between functions in Go. This is used for logging, HTTP requests, etc. so the LLM needs to be able to use it. This has been difficult in the past, but perhaps a well-written `.cursorrules` is a good first step.

One current limitation is that there is only one of these files per workspace, so a monorepo like ours containing code in multiple languages is going to be more difficult to set up than a small repository with a small set of very consistently styled code.

Also the documentation suggests that the `.cursorrules` file is only used for the chat modalities, not the tab completion. However I've experimented with having that file open in a pinned tab in the workspace and confirmed that it is possible to include it in the tab completion context that way at least.

## Changes to my workflow

The most exciting thing about a tool like Cursor is not that I can write code faster, because honestly the actual writing of code is not the bottleneck; in fact, I often have to slow myself down to avoid focusing too much on the code and not enough on the high-level problem being solved. The real value is in changing _how_ I code.

It's still early days with this technology, but this is what I've found has changed about how I work and what I expect to see changing in the near future:

1.  I am _much_ less likely to reach for a new library or a framework. No, I'm not going to start writing my own crypto libraries, but for small utilities it's easy enough to let the LLM write them to my bespoke needs than to pull in a general-purpose library. These libraries tend to start small and lightweight and then, because they are open and used by many people, accumulate functionality and cruft that I don't need.
    
    Many of these libraries only exist to reduce boilerplate, which felt like a necessary tradeoff when balanced against my time writing and maintaining that boilerplate but now that I can have the LLM do it for me it feels less worth the cost. And the cost can be substantial: Have you tried getting a Node.js project running a year or more after it was written? You may as well start from scratch.
    
2.  I also worry less about adhering to DRY (Don't Repeat Yourself) in my own code. Prematurely defining abstractions can create a lot of technical debt later on, so being able to create a lot of code with reference to other code without trying to pull it into a function or class allows me more flexibility, and I know that if I have to refactor shared logic out later, the LLM can help with that too.
    
3.  My willingness to use a language or framework I am less familiar with is much higher. For example, I've dabbled in R for years, especially for visualizing data. However, to be frank, I suck at it. I don't have a deep understanding of `dplyr` and it seems like there are always a dozen different ways to accomplish the same task. Now I describe the visualization I want, and I get correct data manipulation and `ggplot` visualization for it. Tasks that took an hour or more now take five minutes, so I am much less likely to give up and do it in Python instead.
    
    Maybe one of these days I'll even write something in Rust. Maybe.
    
4.  I find myself iterating quickly on small components before integrating them into the larger codebase. This is partly to work around the limitations of LLMs when working with larger codebases, but it also opens up interesting ways of working I hadn't considered before. As per the example above, I can prototype some logic in a dynamically typed language like Python, work out the technical details and then convert it to well-typed Go instantly to integrate into a web application. I can have the LLM generate test data automatically, or mock up a backend for me to write a frontend against. Why pay the tax of working in a mature codebase while I'm still proving out an idea?
    

## Summary

Whether I'll be using Cursor in a few years or have moved on to another tool, I can't really tell. I am confident that at the time of writing this, Cursor is the best example of the potential of LLM coding assistants, and if you want to explore how this type of tool might be of value I suggest you give it a spin.

[1]: #fn:1
[2]: ../videos/cursor-review/example1.webm
[3]: ../videos/cursor-review/example2.webm
[4]: ../videos/cursor-review/example3.webm
[5]: ../videos/cursor-review/example4.webm
[6]: ../videos/cursor-review/example5.webm
[7]: #fn:2