---
title: 💰 My Frugal Indie Dev Startup Stack
authorURL: ""
originalURL: https://getwaitlist.com/blog/solo-dev-startup-stack
translator: ""
reviewer: ""
---

By [Maya Kyler](https://twitter.com/getwaitlist) on December 5, 2022

Indie developers today stand on the shoulders of giants. There are tons of great SAAS products that have made otherwise expensive, enterprise-grade features cheap and accessible. This enables entrepreneurship. I've been able to build [getwaitlist.com](https://getwaitlist.com) on a shoestring budget to serve thousands of users and millions of signups on Waitlists. I thought it might be nice to share the tools I've used to get here, and the favors I've pulled to spend as little as possible.

<!-- more -->

I'll give you the summary up-front, and go into detail afterwards. My monthly spend breaks down as follows:

-   1Password: $2.99
-   Ahrefs: $99.00
-   Amazon Web Services: $0.00
-   Better Uptime: $0.00
-   Cloudflare: $0.00
-   Delaware Franchise Tax: $25.00
-   Figma: $0.00
-   Github: $0.00
-   Google Analytics: $0.00
-   Google Domains: $1.00
-   Google Suites: $6.00
-   Intercom: $89.00
-   Linear: $10.00
-   Mercury: $0.00
-   Meta SEO Inspector: $0.00
-   Retool: $0.00
-   Sendgrid: $0.00
-   Sentry: $0.00
-   Stoplight: $0.00
-   Stripe: $0.00
-   Stripe Atlas: $8.33
-   Vercel: $40.00

**Total spend: $281.32 per month.**

[1Password](https://1password.com)

Credential management is important. I just use the personal plan, billed annually. **Monthly cost: $2.99**.

[Ahrefs](https://ahrefs.com)

Expensive, but worth it! SEO is really important for any indie startup. $99 is Ahrefs' cheapest tier, and I've found a lot of value in it. It's the best SEO/Website optimization tool on the market. **Monthly cost: $99.00**.

[Amazon Web Services](https://aws.amazon.com)

I use AWS EC2 for Waitlist's backend python webserver, RDS for the postgres database, S3 for file storage, and an Elastic IP. I use T2 and T3 instances. My AWS bill would be $200-300 a month, except it's free because I got $5,000 in credits from AWS Activate. These credits are easy to obtain, they are distributed by AWS partners like Retool, Stripe, Brex, YC, Mercury, etc. among many others. Right now, my credits run through 2024. **Monthly cost: $0**.

[Better Uptime](https://betteruptime.com)

You need uptime monitoring and alerting for your SAAS product: both for yourself, and for your customers. You want to be able to show off that 99%+ uptime. Both Cronitor and Better Uptime have great free tiers that support multiple monitors and status pages. I think Better Uptime's product is a little more polished. **Monthly cost: $0**.

[Cloudflare](https://cloudflare.com)

I use Cloudflare to handle all my DNS, SSL certificates, and of course also CDN to optimize bandwidth. The best part is that for a small business like mine, it's all free. I'm conflicted between Cloudflare being an amazing product/UX and also MITMing the internet, but I'll take it. **Monthly cost: $0**.

[Delaware Franchise Tax](https://corp.delaware.gov)

The great state of Delaware charges me $300 per year for the luxury of domiciling my LLC there. **Monthly cost: $25.00**.

[Figma](https://figma.com)

Delightful software. I use Figma whenever I need to mock up a new interface or design a new graphical asset. The free tier is just about sufficient for an indie dev. **Monthly cost: $0.00**.

[GitHub](https://github.com)

Goes without saying. Graciously, the good folks at GitHub have become more generous with free private repositories and organizations. **Monthly cost: $0.00**.

[Google Analytics](https://analytics.google.com)

No surprises here. I used to use [Plausible](https://plausible.io), but that costs money and you can't beat free. **Monthly cost: $0**.

[Google Domains](https://domains.google.com)

[GetWaitlist.com](https://getwaitlist.com) is registered with Google Domains for a grand $12/year. If I were using [Dynadot.com](https://dynadot.com) or another domain registrar, I could get the cost under $10/year. **Monthly cost: $1**.

[Google Suites](https://suites.google.com)

I have just the one email account, plus I use Google Drive and Docs for occasional file storage. Gmail is good, but the rest of the Google Suites experience sucks. Unfortunately I haven't found a better alternative yet, productivity and office software is a great sea of garbage. **Monthly cost: $6.00**.

[Intercom](https://intercom.com)

Even at its cheapest tier, Intercom is not cheap, but it's worth it! I haven't found another messaging widget that's as good. Bonus point for the Intercom iOS app that lets me do customer support on the fly. **Monthly cost: $89.00**.

[Linear](https://linear.app)

The best project management software I have ever used. Could Waitlist scrape by on the free tier, or without Linear altogether? Probably. But this is a delight to use, and I'm happy to pay for it. **Monthly cost: $10.00**.

[Mercury](https://mercury.com)

Easy business banking for startups. It's not as fully featured as other banks, but it suffices for a SAAS business, the interface is nice, and there are no monthly fees. **Monthly cost: $0.00**.

[Meta SEO Inspector](https://www.omiod.com/meta-seo-inspector/)

This is a cute little free browser extension that I use to assess the SEO of my webpages locally, without needing to deploy to production for higher-powered Ahrefs. This extension helps me with keywords, as well as basics like meta tags and various HTML best practices. **Monthly cost: $0.00**.

[Retool](https://retool.com)

I like using Retool as a simple Postgres interface and to do basic data visualization. One seat would cost me $10/month, except I asked them nicely for a startup credit, so now I have it for free. **Monthly cost: $0**.

[Sendgrid](https://sendgrid.com)

At Waitlist, we send lots of emails, usually around 100,000 a month, to notify folks that they've landed on Waitlists or moved up in life thanks to a referral. This would cost me $89.95, but Twilio (Sendgrid's parent) also has a startup program, via which I got $150/month in credits. **Monthly cost: $0**.

[Sentry](https://sentry.io)

Let's face it, we all ship bugs sometimes. If you do, Sentry will tell you. It's my favorite platform for error monitoring and reporting. The tiers are based on the numbers of reports, so the free tier gets tight if you ship too many bugs. 😉 But it hasn't been a problem for us. **Monthly cost: $0**.

[Stoplight](https://stoplight.io)

There's a crazy number of tools out there for writing documentation. Somehow, I've found all of them inadequate. Stoplight's intelligent design for APIs and Schemas is concise, clear, and excellent to read as a developer. It's like Swagger or Postman's better-looking cousin. I migrated all of Waitlist's API documentation to Stoplight, as well as written guides for using Waitlist's features, like referral and email marketing. Stoplight's free tier is tight, but just about suffices for my needs. **Monthly cost: $0.00**.

[Stripe](https://stripe.com)

Normally, Stripe is not cheap: 2.7% of every transaction is no small slice. Thankfully, I was able to get $70,000 of credits for which Stripe's fees wouldn't be applied. Right now, Waitlist is still within that batch of credits. **Monthly cost: $0.00**.

[Stripe Atlas](https://atlas.stripe.com)

We incorporated Waitlist LLC via Stripe Atlas, which charges me $100/year for the required Delaware registered legal agent. **Monthly cost: $8.33**.

[Vercel](https://vercel.com)

I'm paying for two seats on Vercel because I have a collaborator on the frontend. Part of me thinks I shouldn't be paying for this at all and hosting on AWS, which I have for free — but self-administering the modern React deployment chain is tedious. I love how convenient Vercel makes everything. Push to the branch and forget about it, no concerns about load balancing, build tools, or anything else. Wonderful. **Monthly cost: $40.00**.

The final note I'll make is that I've hand-rolled a few additional features that people are normally paying for. For example, I don't use Mailchimp or any tool like that — I send my marketing mail straight via Sendgrid. I dont use [customer.io](https://customer.io) for my email sequences — even though it's very good — because I was able to just build my own email sequencing system. Time will tell whether I end up needing to replace my homespun solutions with more sophisticated third-party SAAS, but so far, my infrastructure for Waitlist is holding up strong!

[Back to Blog](/blog)

[![Waitlist API - Quick and easy waitlist with built in referral. | Product Hunt](https://api.producthunt.com/widgets/embed-image/v1/top-post-badge.svg?post_id=241074&theme=neutral&period=daily)](https://www.producthunt.com/posts/waitlist-api?utm_source=badge-top-post-badge&utm_medium=badge&utm_souce=badge-waitlist-api)

## Get started today.No Credit Card required.

We have the best free tier in the industry. Get started today and launch your waitlist in less than 5 minutes.

[Sign up for free](/signup)

![Application Screenshot Demo](/appImages/demo_image.png)

## Footer

![Waitlist Logo](/logo_waitlist.svg)

Waitlist is viral pre-launch referral marketing software. Use your fans to launch the next big thing!

[![Waitlist API - Quick and easy waitlist with built in referral. | Product Hunt](https://api.producthunt.com/widgets/embed-image/v1/top-post-badge.svg?post_id=241074&theme=neutral&period=daily)](https://www.producthunt.com/posts/waitlist-api?utm_source=badge-top-post-badge&utm_medium=badge&utm_souce=badge-waitlist-api)

© 2023 Waitlist LLC.

### Our Products

-   [Link Shortener](https://app.y.gy)
-   [Waitlist](https://getwaitlist.com)

### Waitlist

-   [Platforms and Features](https://getwaitlist.com/platforms)
-   [Demo](https://getwaitlist.com/demo)
-   [Documentation](https://docs.getwaitlist.com/)
-   [Pricing](/pricing)
-   [Marketplace](/marketplace)
-   [Uptime Status](https://waitlist.betteruptime.com/)

### Company

-   [Blog](https://getwaitlist.com/blog)
-   [Twitter](https://twitter.com/getwaitlist)
-   [Email](mailto:admin@getwaitlist.com)

### Legal

-   [Privacy](/privacy)
-   [Terms](/terms)
