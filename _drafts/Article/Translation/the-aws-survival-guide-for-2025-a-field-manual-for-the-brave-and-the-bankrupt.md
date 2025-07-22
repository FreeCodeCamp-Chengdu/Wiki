---
title: "The AWS Survival Guide for 2025: A Field Manual for the Brave and
  the Bankrupt"
date: 2025-07-09T00:00:00.000Z
author: By Corey Quinn
authorURL: https://www.lastweekinaws.com/blog/author/cquinn/
originalURL: https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/
translator: ""
reviewer: ""
---

-   [Skip to primary navigation][1]
-   [Skip to main content][2]

<!-- more -->

[![Last Week in AWS Logo](https://www.lastweekinaws.com/wp-content/uploads/2019/04/last-week-in-aws-logo.svg)][3]

[Lower My AWS Bill][4] ![Mobile Navigation Icon](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%2036%2034'%3E%3C/svg%3E)

![Mobile Navigation Icon](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/utility/mobile-nav-closed.svg)

![Close Mobile Navigation Icon](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%2032%2032'%3E%3C/svg%3E)

![Close Mobile Navigation Icon](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/utility/mobile-nav-open.svg)

-   [About][5]![Submenu Down Arrow](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/utility/caret-down-solid-orange.svg)
    -   [Community][6]
    -   [Contact][7]
    -   [Contribute][8]
-   [Blog][9]
-   [Newsletter][10]
-   [Podcasts][11]![Submenu Down Arrow](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/utility/caret-down-solid-orange.svg)
    -   [Last Week in AWS][12]
    -   [Screaming in the Cloud][13]
    -   [Nominate a Guest][14]
-   [Merch][15]
-   [Resources][16]![Submenu Down Arrow](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/utility/caret-down-solid-orange.svg)
    -   [AWS Network Map][17]
-   [Sponsorships][18]

![](https://www.lastweekinaws.com/wp-content/uploads/2025/07/pexels-towfiqu-barbhuiya-3440682-8515596-scaled.jpg)

# The AWS Survival Guide for 2025: A Field Manual for the Brave and the Bankrupt

[By Corey Quinn][19]

Welcome, intrepid cloud explorer! You’ve decided to venture into the AWS jungle in 2025, where the services multiply faster than your monthly bill. Forget those quaint relics like S3, EC2, and RDS…

[Facebook][20][Tweet][21][LinkedIn][22][Reddit][23]

[Home][24] [Blog][25] The AWS Survival Guide for 2025: A Field Manual for the Brave and the Bankrupt

[Prev][26]

Welcome, intrepid cloud explorer! You’ve decided to venture into the AWS jungle in 2025, where the services multiply faster than your monthly bill. Forget those quaint relics like S3, EC2, and RDS that everyone always gravitates towards—despite being the lion’s share of AWS revenue, they’re practically stone tablets now when it comes to interest and attention. Let’s talk about navigating the _real_ AWS experience.

## Chapter 1: The Service Name Generator

First, you’ll need to understand modern AWS service naming. They’ve clearly hired a team of Scrabble champions who’ve been hitting the espresso too hard and have used up all the good letters / names in the early game. Need serverless AI-powered quantum computing? That’s AWS QuantumLambdaForgeMaxProUltra™ (but they call it “Amazon Q” for short). Want to deploy a simple API? You’ll need AWS HyperGatewayMeshFabricOrchestrator360™.

Pro tip: If the service name doesn’t sound like a rejected Transformer, it’s probably deprecated, by which I of course mean “a Google product.”

## Chapter 2: The Documentation Labyrinth

AWS documentation in 2025 is an immersive experience in much the same way as being waterboarded. Think of it like an escape room where the prize is understanding what the service actually does. Each doc page contains:

-   47 architecture diagrams that look like someone sneezed on a circuit board with a nose full of cheap clipart
-   3 contradictory “Getting Started” guides that veer wildly between one another, and assume you’ve either read the other 2, or at the very least spent 5 years working on an AWS service team
-   1 example that worked in 2023, for one person, rushing to hit a re:Invent deadline
-   0 explanations of pricing. (If you think I’m shitposting, tell me what an Aurora DSQL Compute DPU represents in the real world. I’ll wait.)

Remember: If you understand the documentation on first read, you’re reading the docs for the wrong service. If it’s any consolation, the one you’re reading almost certainly runs containers.

## Chapter 3: IAM Policies—The Dark Arts

Writing IAM policies is like playing 4D chess while blindfolded and riding a unicycle. In 2025, you need permissions to request permissions to view the permissions you need. The principle of least privilege has evolved into the principle of “good luck figuring out why this doesn’t work, beat your head on this until you give up and allow \*.”

Sample modern IAM policy:

{
  "Effect": "Deny",
  "Action": "Everything:YouActuallyNeed",
  "Resource": "\*",
  "Condition": {
    "StringEquals": {
      "aws:PleaseWork": "false"
    }
  }
}

## Chapter 4: The Billing Dashboard—A Horror Story

Opening your AWS bill is the modern equivalent of opening Pandora’s box. You’ll discover charges for:

-   Services you’ve never heard of
-   Regions you can’t pronounce—also known as _Mr. Jassy’s Geography Class_
-   “AWS ThoughtAboutProcessingYourRequest” fees
-   Data egress from your nightmares

The Cost Explorer now requires its own Cost Explorer to understand why the Cost Explorer costs so much. Obnoxiously, there’s still no cohesive overview of everything in your account, despite the mediocre efforts of Resource Explorer to fumble its way into that gap.

## Chapter 5: re:Invent Announcements

Every re:Invent (_re:Invent: It’s the Week After Thanksgiving Because We Hate Our Families and Figure You Do, Too_™), AWS announces 73 new services that all do slightly different versions of the same thing. By 2025, there are [51 different ways to run containers on AWS][27], each with its own pricing model that would make a derivatives trader weep. You’ll need to choose between:

-   AWS ContainerThing
-   AWS KubernetesButSlightlyDifferent
-   AWS DockerWithExtraSteps
-   Amazon JustUseOurManagedEverything
-   AWS ContainerContainerContainer
-   Amazon ScrewItJustUseLambda

## Chapter 6: Support—The Mythical Creature

AWS Support in 2025 is a game of telephone played through Google Translate. Your simple question about network connectivity will result in a 3-week email chain discussing banana import regulations in Peru. The folks in AWS Support are amazing and precious (seriously, they’re incredible), which is why AWS has taken significant steps to wall off any and all access to them that doesn’t first pass through the gauntlet of useless GenAI. This is to test your mettle and ensure you’re determined to solve a problem, rather than just idly wishing a service would do what the documentation says it should.

### Support Tier Guide:

-   Basic: Thoughts and prayers
-   Developer: Automated responses that blame your code
-   Business: Real humans who’ve never used AWS
-   Enterprise On-Ramp: AWS learned to only rip out one of your kidneys at a time
-   Enterprise: Andy Jassy personally ignores your tickets
-   Shitposting Crisis: You’re reading it right now

## Chapter 7: The Certification Treadmill

AWS now releases new certifications faster than you can earn them. By the time you pass the “AWS Certified Quantum Blockchain Solutions Architect—Associate Level 3.5 Beta,” it’s already obsolete, because their testing partner Pearson Vue gets its corporate self off on abusing test-takers purely out of malice. Your LinkedIn profile will need its own CDN to host all your certification badges, but they’re too busy stuffing _that_ product with insipid GenAI, too.

## Survival Tips

1.  **Budget like you’re planning for the apocalypse**—because your AWS bill might cause one
2.  **Learn to love acronyms**—Your life now is EKS, ECS, ECR, EMR, EBS, EFS, and crying
3.  **Embrace the chaos**—If something works on the first try, you’ve definitely done it wrong
4.  **Keep a therapist on speed dial**—Preferably one who accepts payment in AWS credits; you can find them in the AWS Marketplace
5.  **Remember the golden rule**—It’s always DNS. Even when it’s not DNS, it’s DNS. Which is a database.

## Epilogue: The Path Forward

Congratulations! You’re now ready to embark on your AWS journey. Remember, every expert was once a beginner who wondered why their [“simple” WordPress site][28] costs $3,000 a month to run.

May your lambdas be warm, your regions be close, and your bills be… well, let’s just focus on the first two.

_Disclaimer: This guide is not responsible for any emotional damage, financial ruin, or existential crises resulting from using AWS. Side effects may include: compulsive dashboard refreshing, nightmares about cascading failures, and an irrational fear of the words “data transfer costs.”_

![Corey Quinn Headshot](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20160%20160'%3E%3C/svg%3E)

![Corey Quinn Headshot](https://www.lastweekinaws.com/wp-content/uploads/2019/09/Corey_Quinn_2019_02-150x150.jpg)

by Corey Quinn

Corey is the Chief Cloud Economist at The Duckbill Group, where he specializes in helping companies improve their AWS bills by making them smaller and less horrifying. He also hosts the "Screaming in the Cloud" and "AWS Morning Brief" podcasts; and curates "Last Week in AWS," a weekly newsletter summarizing the latest in AWS news, blogs, and tools, sprinkled with snark and thoughtful analysis in roughly equal measure.

## More Posts from Corey

[Back to the Blog][29]

[![](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20350%20200'%3E%3C/svg%3E)

![](https://www.lastweekinaws.com/wp-content/uploads/2025/06/pexels-olly-3760809-350x200.jpg)

][30]

### [AWS Certificate Manager Has Announced Exportable TLS Certificates, and I’m Mostly Okay With It][31]

[By Corey Quinn][32]

I don’t think it’s going too far to say that free TLS certificate offerings like Let’s Encrypt and AWS Certificate Manager have taken encrypted connections mainstream.

[Read More about AWS Certificate Manager Has Announced Exportable TLS Certificates, and I’m Mostly Okay With It][33]

[![](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20350%20200'%3E%3C/svg%3E)

![](https://www.lastweekinaws.com/wp-content/uploads/2025/06/pexels-lilartsy-1925536-350x200.jpg)

][34]

### [A Day in the Life of Server #47B-2: An AWS Data Center Memoir][35]

[By Corey Quinn][36]

Or: Surprise! I Contain Multitudes and Also Your “Serverless” Functions

[Read More about A Day in the Life of Server #47B-2: An AWS Data Center Memoir][37]

[![Photo by Pei Peng on Unsplash](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20350%20200'%3E%3C/svg%3E)

![Photo by Pei Peng on Unsplash](https://www.lastweekinaws.com/wp-content/uploads/2025/05/pei-peng-3DJfbRW__Fo-unsplash-350x200.png)

][38]

### [Cloud Repatriation is Getting Complicated][39]

[By Corey Quinn][40]

Five years ago, I fairly confidently stated that Cloud Repatriation Isn’t a Thing, and I by and large stand by what I wrote. That said, it’s 2025, and the story has changed somewhat.

[Read More about Cloud Repatriation is Getting Complicated][41]

![Billie Holding Mail Email Subscribe Icon](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20123%20140'%3E%3C/svg%3E)

![Billie Holding Mail Email Subscribe Icon](https://www.lastweekinaws.com/wp-content/themes/lwiaws-2022/assets/images/blocks/billie-newsletter.svg)

## Get the newsletter!

Stay up to date on the latest AWS news, opinions, and tools, all lovingly sprinkled with a bit of snark.

<iframe style="display:none;width:0px;height:0px;" src="about:blank" name="gform_ajax_frame_1" id="gform_ajax_frame_1" title="This iframe contains the logic required to handle Ajax powered Gravity Forms."></iframe>

The world of cloud takes itself far too seriously. We aim to change that.

[Lower my AWS bill, please!][42]

[][43][][44][][45][][46]

-   [Newsletter][47]
-   [Podcasts][48]
-   [Blog][49]
-   [Merch][50]
-   [Contribute][51]

-   [About][52]
-   [Contact][53]
-   [Sponsorships][54]
-   [Disclosures][55]

![Billie Footer](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%20400%20330'%3E%3C/svg%3E)

![Billie Footer](https://www.lastweekinaws.com/wp-content/uploads/2022/01/billie-lwiaws-footer.svg)

footprint-orange

© 2025 The Duckbill Group. All Rights Reserved.

[Privacy Policy][56] [Cookie Policy][57]

           

[1]: #genesis-nav-primary
[2]: #genesis-content
[3]: https://www.lastweekinaws.com
[4]: https://duckbillgroup.com
[5]: https://www.lastweekinaws.com/about/
[6]: https://www.lastweekinaws.com/community/
[7]: https://www.lastweekinaws.com/contact/
[8]: https://www.lastweekinaws.com/contribute/
[9]: https://www.lastweekinaws.com/blog/
[10]: https://www.lastweekinaws.com/newsletter/
[11]: https://www.lastweekinaws.com/podcast/
[12]: https://www.lastweekinaws.com/podcast/aws-morning-brief/
[13]: https://www.lastweekinaws.com/podcast/screaming-in-the-cloud/
[14]: https://www.lastweekinaws.com/nominate-a-guest/
[15]: https://store.lastweekinaws.com/collections/all
[16]: https://www.lastweekinaws.com/resources/
[17]: https://awsnetwork.lastweekinaws.com/
[18]: https://www.lastweekinaws.com/sponsorship/
[19]: https://www.lastweekinaws.com/blog/author/cquinn/
[20]: https://www.facebook.com/sharer/sharer.php?u=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/&display=popup&ref=plugin&src=share_button "Share on Facebook"
[21]: https://twitter.com/share?url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/&text=The%20AWS%20Survival%20Guide%20for%202025%3A%20A%20Field%20Manual%20for%20the%20Brave%20and%20the%C2%A0Bankrupt "Share on Twitter"
[22]: https://www.linkedin.com/shareArticle?mini=true&url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/ "Share on LinkedIn"
[23]: https://www.reddit.com/submit?url=https://www.lastweekinaws.com/blog/the-aws-survival-guide-for-2025-a-field-manual-for-the-brave-and-the-bankrupt/ "Share on Reddit"
[24]: https://www.lastweekinaws.com/
[25]: https://www.lastweekinaws.com/blog/
[26]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[27]: https://www.lastweekinaws.com/blog/17-final-ways-to-run-containers/
[28]: https://bsky.app/profile/quinnypig.com/post/3lqfctagyue2y
[29]: https://www.lastweekinaws.com/blog/
[30]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[31]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[32]: https://www.lastweekinaws.com/blog/author/cquinn/
[33]: https://www.lastweekinaws.com/blog/aws-certificate-manager-has-announced-exportable-tls-certificates-and-im-mostly-okay-with-it/
[34]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[35]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[36]: https://www.lastweekinaws.com/blog/author/cquinn/
[37]: https://www.lastweekinaws.com/blog/a-day-in-the-life-of-server-47b-2-an-aws-data-center-memoir/
[38]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[39]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[40]: https://www.lastweekinaws.com/blog/author/cquinn/
[41]: https://www.lastweekinaws.com/blog/cloud-repatriation-is-getting-complicated/
[42]: https://www.duckbillgroup.com/
[43]: https://twitter.com/LastWeekinAWS
[44]: https://www.linkedin.com/company/last-week-in-aws/
[45]: https://www.lastweekinaws.com/feed/
[46]: https://www.lastweekinaws.com/community
[47]: https://www.lastweekinaws.com/newsletter/
[48]: https://www.lastweekinaws.com/podcast/
[49]: https://www.lastweekinaws.com/blog/
[50]: https://www.lastweekinaws.com/products/
[51]: https://www.lastweekinaws.com/contribute/
[52]: https://www.lastweekinaws.com/about/
[53]: https://www.lastweekinaws.com/contact/
[54]: https://www.lastweekinaws.com/sponsorship/
[55]: https://www.lastweekinaws.com/disclosures/
[56]: https://www.lastweekinaws.com/privacy-policy/
[57]: https://www.lastweekinaws.com/cookie-policy/