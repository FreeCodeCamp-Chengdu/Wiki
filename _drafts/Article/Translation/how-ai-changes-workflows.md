---
title: Matt Rickard
date: 2026-07-16T07:04:36.190Z
authorURL: ""
originalURL: https://blog.matt-rickard.com/p/how-ai-changes-workflows
translator: ""
reviewer: ""
---

# How AI Changes Workflows

<!-- more -->

Nov 24, 2023

7

1

Share

GitHub recently said it was “re-founding” itself on Copilot instead of git. GitHub has always been about the workflow — there are plenty of other hosted git providers, but GitHub was the first to put together pull requests, issues, and collaboration into a single workflow. Re-founding on Copilot is a way to acknowledge that AI will drastically change the developer workflow.

Some more general lessons on how AI changes workflows, using the developer workflow as an example

**The same but faster steps.** [Copilot is an incumbent business model][1] when used this way. Doing the same things that we’ve always done, but just faster with the help of AI. That means autocompleted code or AI-assisted code reviews. AI-generated commit messages.

**Compressing the workflow.** AI might help us skip steps in the workflow. Developers have tried to make pre-commit workflows work for decades, but they’ve always failed because they can’t be centralized well (if you automatically change the code before it’s committed, there’s a chance that your automated changes end up with a broken main branch).

-   What if AI could determine “low-risk” change sets that could be merged without a review?
    
-   Why have AI-generated commit messages if they [don’t matter in the first place][2]? Commit messages could be generated on-demand (or post-commit)
    
-   [Automatic merge conflict resolution and automatic linting and style checking.][3]
    

**A new workflow.** If so many of the steps don’t make sense anymore, the whole workflow might come into question.

-   Maybe [issue tracking comes before code][4] in future DevOps platforms.
    
-   AI will write most code in the future. What’s the implication? Does all the code need to be checked in?
    

**Extends the platform to support more workflows.** Especially in enterprise software, almost every company’s workflow is different in a certain way. SaaS products extend themselves into platforms in a variety of ways — letting users customize via a WYSIWYG interface, configuration, or even code. But platform extension comes with its own problems — open up too much and you can’t support your customers on a large scale. Open up too little, and niche platforms chip away at your customer base.

[DSLs often fail.][5] But products might find it easier to become platforms in the age of AI. Giving the users the ability to autogenerate DSLs or generic code to extend their platform (even if they are semi-technical, or not technical at all). Imagine every platform could be as extensible as Salesforce — its own programming language and toolchain.

7

1

Share

PreviousNext

#### Discussion about this post

CommentsRestacks

[

![Swaroop CH's avatar](https://substackcdn.com/image/fetch/$s_!ryS2!,w_32,h_32,c_fill,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack.com%2Fimg%2Favatars%2Fyellow.png)



][6]

[Swaroop CH][7]

[Nov 24, 2023][8]

Maybe the Copilot generates placeholders with dynamically generated implementation like [https://askmarvin.ai/welcome/what\_is\_marvin/][9] and engineers still have the option to write their own implementation if needed, avoiding the pitfalls of DSL?

Reply

Share

TopLatestDiscussions

No posts

### Ready for more?

[1]: https://matt-rickard.com/copilot-is-an-incumbent-business-model
[2]: https://matt-rickard.com/pull-requests-not-commits
[3]: https://matt-rickard.com/llms-for-code
[4]: https://matt-rickard.com/devops-wedge-code-or-issues
[5]: https://matt-rickard.com/why-dsls-fail
[6]: https://substack.com/profile/12940616-swaroop-ch?utm_source=comment
[7]: https://substack.com/profile/12940616-swaroop-ch?utm_source=substack-feed-item
[8]: https://blog.matt-rickard.com/p/how-ai-changes-workflows/comment/44213630 "Nov 24, 2023, 8:26 PM"
[9]: https://askmarvin.ai/welcome/what_is_marvin/