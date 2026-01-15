---
title: Why I don’t like “Correction of Error”
date: 2025-12-21T00:24:58.000Z
author: View all posts by Lorin Hochstein
authorURL: https://surfingcomplexity.blog/author/lorinh/
originalURL: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/
translator: ""
reviewer: ""
---

[Skip to content][1]

<!-- more -->

[![Unknown's avatar](https://0.gravatar.com/avatar/f2641f12e815a54896f8f2ac04660c52efb896e09131390ad2a6f2f5fca81432?s=80&d=identicon&r=G)][2]

[Surfing Complexity][3]

Lorin Hochstein's ramblings about software, complex systems, and incidents.

# Why I don’t like “Correction of Error”

[Lorin Hochstein][4] [incidents][5] December 20, 2025 5 Minutes

Like many companies, AWS has a defined process for reviewing incidents. They call their process **_Correction of Error_**. For example, there’s a [page on Correction of Error][6] in their Well-Architected Framework docs, and there’s an AWS blog entry titled [Why you should develop a correction of error (COE)][7].

In [my previous blog post][8] on the AWS re:Invent talk about the us-east-1 incident that happened back in October, I wrote the following:

> Finally, I still grit my teeth whenever I hear the Amazonian term for their post-incident review process: _**Correction of Error**_.

On LinkedIn, [Michael Fisher asked me][9]: “Why does the name Correction of Error make you grit your teeth?” Rather than just reply in a LinkedIn comment, I thought it would be better if I wrote a blog post instead. Nothing in this post will be new to regular readers of this blog.

---

I hate the term “Correction of Error” because it implies that **_incidents occur as a result of errors_**. As a consequence, it suggests that the function of a post-incident review process is to identify the errors that occurred and to fix them. I think this view of incidents is wrong, and dangerously so: It limits the benefits we can get out of an incident review process.

Now, I will agree that, in just about every incident that happens, during the post-incident review work, you can identify _errors_ that happened. This almost always involves identifying one or more bugs in code (I’ll call these _software defects_ here). You can also identify _process errors_: things that people did that, in hindsight, we can recognize as having contributed to an incident. (As an aside, see Scott Snook’s book [Friendly Fire: The Accidental Shootdown of U.S. Black Hawks over Northern Iraq][10] for a case study of an incident where there were no such errors even identified. But I’ll still assume here that we can always identify errors after an incident).

So, if I agree that there are always _errors_ involved in incidents, what’s my problem with “Correction of Error”? In short, my problem with this view is that it fundamentally misunderstands the nature the role of both software defects and human work in incidents.

## More software defects than you can count

> Wall Street indexes predicted nine out of the last five recessions! And its mistakes were beauties. – Paul Samuelson

Yes, you can always find software defects in the wake of an incident. The problem with attributing an incident to a software defect is that modern software systems running in production are absolutely _riddled_ with defects. I cannot count the times I’ve read about an incident that involved a software defect that had been present for months or even years, and had gone completely undetected until conditions arose where it contributed to an outage. My claim is that there are many, many, many such defects in your system, that have yet to manifest as outages. Indeed, I think that most of this defects will _never_ manifest as outages.

If your system is currently up (which I bet it is), and if your system currently has multiple undetected defects in it (which I also bet it does), then it cannot be the case that defects are a sufficient condition for incidents to occur. In other words, **_defects alone can’t explain incidents_**. Yes, they are part of the story of incidents, but only a part of it. By focusing solely on the _defects_, the errors, means that you won’t look at the systemic nature of system failures. You will not see how the existing defects interact with other behaviors in the system to enable the incident. You’re looking for errors, and unexpected interactions aren’t “errors”.

For more of my thoughts on this point, see my previous posts [The problem with a root cause is that it explains too much][11], [Component defects: RCA vs RE][12] and [Not causal chains, but interactions and adaptations][13].

## On human work and correcting errors

Another concept that _error_ invokes in my mind is what I called _process error_ above. In my experience, this typically refers to either insufficient validation during development work that led to a defect making it into production, or an operational action that led to the system getting into a bad state (for example, a change to network configuration that results in services accidentally becoming inaccessible). In these cases, **_correction of error_** implies making a change to development or operational processes in order to prevent similar errors happening in the future. That sounds good, right? What’s the problem with looking at how the current process led to an error, and changing the process to prevent future errors?

There are two problems here. One problem is that there might not actually be any kind of “error” in the normal work processes, that these processes are actually successful in virtually every circumstance. Imagine if I asked, “let’s tally up the number of times the current process did not lead to an incident, versus the number of times that the current process did lead to an incident, and use that to score how effective the process is.” _Correction of error_ implies to me that you’re looking to identify an error in the work process, it does not imply “let’s understand how the normal work actually gets done, and how it was a reasonable thing for people to typically work that way.” In fact, changing the process may actually increase the risk of future incidents! You could be adding constraints, which could potentially lead to new dangerous workarounds. What I want is a focus on understanding how the work normally gets done. _Correction of error_ implies the focus is specifically on identifying the error and correcting it, not on understanding the nature of the work and how decisions made sense in the moment.

Now, sometimes people need to use _workarounds_ in order to get their work done because there are constraints that are preventing them from doing the work the way they are _supposed to do it_, and the workaround is dangerous in some way, and that contributes to the incident. And this is an important insight to take away from an incident! But this type of workaround isn’t an _error_, it’s an adaptation. _Correction of error_ to me implies changing work practices identified as erroneous. Changing work practices is typically done by adding constraints on the way work is done. And it’s these exact type of constraints that can increase risk!

---

Remember, errors happen every single day, but incidents don’t. _Correction of error_ evokes the idea that incidents are caused by errors. But until you internalize that errors aren’t enough to explain incidents, you won’t understand [how incidents actually happen in complex systems][14]. And that lack of understanding will limit how much you can genuinely improve the reliability of your system.

Finally, I think that correcting errors is such a feeble goal for a post-incident process. We can get so much more out of post-incident work. Let’s set our sights higher.

### Share this:

-   [Share on X (Opens in new window) X][15]
-   [Share on Facebook (Opens in new window) Facebook][16]

Like Loading...

![Unknown's avatar](https://0.gravatar.com/avatar/f2641f12e815a54896f8f2ac04660c52efb896e09131390ad2a6f2f5fca81432?s=80&d=identicon&r=G)

## Published by Lorin Hochstein

[View all posts by Lorin Hochstein][17]

**Published** December 20, 2025December 20, 2025

## Post navigation

[Previous Post AWS re:Invent talk on their Oct ’25 incident][18]

[Next Post Quick takes on the Triple Zero Outage at Optus – the Schott Review][19]

## One thought on “Why I don’t like “Correction of Error””

1.  ![Hamed's avatar](https://2.gravatar.com/avatar/b933d23ff46ee19070c4547528230a3412a1c19b44f5dea90b57263515c4c021?s=48&d=identicon&r=G) **Hamed** says:
    
    [January 12, 2026 at 3:56 am][20]
    
    I totally agree Lorin, Mongodb vulnerability is a good example
    
    [Reply][21]
    

### Leave a comment [Cancel reply][22]

[Blog at WordPress.com.][31]

  

-   [Comment][32]
-   Reblog
-   Subscribe Subscribed
    
    -    [![](https://surfingcomplexity.blog/wp-content/uploads/2020/05/surfer_1f3c4.png?w=50) Surfing Complexity][33]
    
    -   Already have a WordPress.com account? [Log in now.][34]
        
    
-   -    [![](https://surfingcomplexity.blog/wp-content/uploads/2020/05/surfer_1f3c4.png?w=50) Surfing Complexity][35]
    -   Subscribe Subscribed
    -   [Sign up][36]
    -   [Log in][37]
    -   [Copy shortlink][38]
    -   [Report this content][39]
    -   [View post in Reader][40]
    -   [Manage subscriptions][41]
    -   Collapse this bar
    

         <iframe src="https://widgets.wp.com/likes/master.html?ver=20260115#ver=20260115" scrolling="no" id="likes-master" name="likes-master" style="display:none;"></iframe>

%d

![](https://pixel.wp.com/b.gif?v=noscript)

 

[1]: #content
[2]: https://surfingcomplexity.blog/
[3]: https://surfingcomplexity.blog/
[4]: https://surfingcomplexity.blog/author/lorinh/ "Posts by Lorin Hochstein"
[5]: https://surfingcomplexity.blog/category/systems/incidents/
[6]: https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.concept.coe.en.html
[7]: https://aws.amazon.com/blogs/mt/why-you-should-develop-a-correction-of-error-coe/
[8]: /2025/12/14/aws-reinvent-talk-on-their-oct-25-incident/
[9]: https://www.linkedin.com/feed/update/urn:li:activity:7406226445292756992?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A7406226445292756992%2C7406344730428715008%29&dashCommentUrn=urn%3Ali%3Afsd_comment%3A%287406344730428715008%2Curn%3Ali%3Aactivity%3A7406226445292756992%29
[10]: https://press.princeton.edu/books/paperback/9780691095189/friendly-fire
[11]: /2024/05/26/the-problem-with-a-root-cause-is-that-it-explains-too-much/
[12]: /2025/07/09/component-defects-rca-vs-re/
[13]: /2025/05/19/not-causal-chains-but-interactions-and-adaptations/
[14]: https://www.adaptivecapacitylabs.com/HowComplexSystemsFail.pdf
[15]: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/?share=twitter
[16]: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/?share=facebook
[17]: https://surfingcomplexity.blog/author/lorinh/
[18]: https://surfingcomplexity.blog/2025/12/14/aws-reinvent-talk-on-their-oct-25-incident/
[19]: https://surfingcomplexity.blog/2025/12/21/quick-takes-on-the-triple-zero-outage-at-optus-the-schott-review/
[20]: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/#comment-9784
[21]: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/?replytocom=9784#respond
[22]: /2025/12/20/why-i-dont-like-correction-of-error/#respond
[23]: https://lorinhochstein.org
[24]: https://bsky.app/profile/norootcause.surfingcomplexity.com
[25]: https://hachyderm.io/@norootcause
[26]: https://surfingcomplexity.blog/2025/12/28/on-intuition-and-anxiety/
[27]: https://surfingcomplexity.blog/2025/12/27/the-dangers-of-ssl-certificates/
[28]: https://surfingcomplexity.blog/2025/12/23/saturation-waymo-edition/
[29]: https://surfingcomplexity.blog/2025/12/22/another-way-to-rate-incidents/
[30]: https://surfingcomplexity.blog/2025/12/21/quick-takes-on-the-triple-zero-outage-at-optus-the-schott-review/
[31]: https://wordpress.com/?ref=footer_blog
[32]: https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/#comments
[33]: https://surfingcomplexity.blog
[34]: https://wordpress.com/log-in?redirect_to=https%3A%2F%2Fr-login.wordpress.com%2Fremote-login.php%3Faction%3Dlink%26back%3Dhttps%253A%252F%252Fsurfingcomplexity.blog%252F2025%252F12%252F20%252Fwhy-i-dont-like-correction-of-error%252F
[35]: https://surfingcomplexity.blog
[36]: https://wordpress.com/start/
[37]: https://wordpress.com/log-in?redirect_to=https%3A%2F%2Fr-login.wordpress.com%2Fremote-login.php%3Faction%3Dlink%26back%3Dhttps%253A%252F%252Fsurfingcomplexity.blog%252F2025%252F12%252F20%252Fwhy-i-dont-like-correction-of-error%252F
[38]: https://wp.me/p236df-1Pq
[39]: https://wordpress.com/abuse/?report_url=https://surfingcomplexity.blog/2025/12/20/why-i-dont-like-correction-of-error/
[40]: https://wordpress.com/reader/blogs/30291541/posts/7032
[41]: https://subscribe.wordpress.com/