---
title: Your API Shouldn't Redirect HTTP to HTTPS
date: 2024-05-23T00:00:00.000Z
authorURL: ""
originalURL: https://jviide.iki.fi/http-redirects
translator: ""
reviewer: ""
---

## Background

<!-- more -->

When an user directs their web browser to an HTTP URL, it's a common practice for the service to redirect the request to a corresponding HTTPS page. This unencrypted part of the communication flow has its flaws. Third parties in shared networks, as well as network intermediaries, could [sniff][1] passwords and other secrets from the initial HTTP traffic or even impersonate the web server with a [MITM][2] attack. Nevertheless, redirection has been an useful first step in the transition from the largely unencrypted early web to the [largely encrypted][3] web of today.

Later techniques tightened the security story further. Servers can now send [HSTS][4] along with the initial HTTP-to-HTTPS redirection response, telling the user's browser to use only HTTPS for that domain from then on. This limits the window of opportunity for trivial sniffing and MITM attacks to the first request. Browsers then added [HSTS preload lists][5] and [HTTPS-Only modes][6] that allow skipping the initial unencrypted request altogether.

From the perspective of usability-security tradeoff it all makes sense for user-facing sites. But interestingly, the redirection approach also appears to be widely adopted for APIs. APIs are mostly consumed by other software so the same usability arguments don't apply there. Moreover, many programmatic API clients don't tend to keep browser-like state of things like HSTS headers they have seen.

This post argues that, due to these factors, the common practice of redirecting API calls from HTTP to HTTPS should be reconsidered. While the post mostly refers to REST APIs, its points also apply to other styles of APIs that use HTTP(S) as a transport mechanism.

## A Simple Typo Is Enugh

At [work][7], we were building a new integration against a third-party API. The initial code commit contained a mistyped API base URL `"http://..."` instead of `"https://..."`. A pretty easy mistake to make.

The error was essentially masked during runtime: The third-party API responded to every request with a 301 redirect to their HTTPS side. Node.js's built-in [fetch][8] happily and _quietly_ followed those redirects to the HTTPS endpoint.

**Every single one of our API requests now sent the API keys over the network in plaintext**, before then sending them again to the encrypted endpoint. The one letter omission could have exposed the used API keys to third parties without us realizing it. As the integration would have worked, there's a good chance that code would have leaked any secrets in the API calls for years. In the long run, the probabilities for malice tend to accumulate.

Luckily we spotted the error during the code review before the error could propagate to production or even testing. We also realized that our _own_ API also did similar HTTP-to-HTTPS redirects.

## The Fail-fast Principle

When an API redirect HTTP requests to HTTPS - and the API client silently follows those redirects - it tends to hide mistyped URLs like in the case described above. A simple one-letter omission can easily be ignored, end up in production, and compromise the entire system's confidentiality.

In most cases, it's better to adhere to the [fail-fast principle][9]: unencrypted API calls should fail in a spectacular and visible way so that the developer can easily spot and fix the typo as early as possible during the development process.

A great solution for failing fast would be to disable the API server's HTTP interface altogether and not even answer to connections attempts to port 80. If the initial unencrypted connection is never established then the API keys aren't sent, mitigating sniffing attacks and limiting the window of opportunity for MITM attacks to an extremely small time window. This approach is viable for APIs hosted under their own domains like `api.example.com`.

Our own API was served under the `/api` path on the same domain as our service's web UI. As such we didn't feel confident completely disabling the HTTP interface, so we picked the second option: All unencrypted HTTP requests made under `/api` path now return a descriptive error message along with the HTTP status code 403. Some initial plaintext requests might be made during development, but they're much easier for developers to notice.

After the initial publication of this post, Hacker News user u/zepton [made a great point:][10]

Indeed! Any API keys sent unencrypted over the public internet should be considered compromised and get automatically revoked on the spot. An error message in the API response is a great place to inform the API consumer both to fix their URL and get new keys after that.

## Who Else?

That took care of our own API. We also pinged the third-party API provider and a couple of friends that they might want to check their APIs. And who knows, maybe there were some commonly used APIs that accept API keys (or other credentials) and also redirect from HTTP to HTTPS?

I listed a bunch of well-known APIs from the top of my head and did a little survey. Several of them returned HTTP errors or declined to connections altogether. They're listed here with cURL spells for checking out their detailed responses:

1.  **Stripe API**: Responds with [403][11] ("Forbidden") and a descriptive error message.  
    `curl -i http://api.stripe.com`
2.  **Google Cloud API**: Responds with 403 and a descriptive error message.  
    `curl -i http://compute.googleapis.com/compute/v1/projects/project/regions/region/addresses`
3.  **Shopify API**: Responds with 403 and a descriptive error message.  
    `curl -i http://shop.myshopify.com/admin/api/2021-07/shop.json`
4.  **NPM Registry API**: Responds with [426][12] ("Upgrade Required") and a descriptive error message.  
    `curl -i -X PUT -H 'content-type: application/json' -d '{}' 'http://registry.npmjs.org/-/user/org.couchdb.user:npm'`
5.  [Fastmail JMAP API][13]: The whole HTTP interface seems to be disabled.  
    `curl -i -H 'Authorization: Bearer foo' http://api.fastmail.com/jmap/session`
6.  **Mailjet**: The socket responds with completely empty payload.  
    `curl -i -X POST --user "user:pass" http://api.mailjet.com/v3.1/send -H 'Content-Type: application/json' -d '{}'`

However, the following APIs _did_ respond with HTTP-to-HTTPS redirects:

1.  [ActiveCampaign API][14]  
    `curl -i -H "Api-Token: 123abc-def-ghi" http://123456demo.api-us1.com/api/3/accounts`
2.  [Atlassian Jira REST API][15]  
    `curl -i http://jira.atlassian.com/rest/api/latest/issue/JRA-9`
3.  [Anthropic API][16]  
    `curl -i http://api.anthropic.com/v1/messages --header "x-api-key: 1" --header "anthropic-version: 2023-06-01" --header "content-type: application/json" --data '{}'`
4.  [Auth0][17]  
    `curl -i 'http://login.auth0.com/api/v2/organizations' -H 'Accept: application/json' -H 'Authorization: Bearer foo'`
5.  [Cloudflare API][18]  
    `curl -i http://api.cloudflare.com/client/v4/accounts/abf9b32d38c5f572afde3336ec0ce302/rulesets`
6.  [Datadog][19]  
    `curl -i http://api.datadoghq.com/api/v2/integration/gcp/accounts`
7.  [Deno Subhosting API][20]  
    `curl -i http://api.deno.com/v1/organizations/11111111-2222-3333-4444-555555555555/projects`
8.  [DigitalOcean][21]  
    `curl -i -X GET "http://api.digitalocean.com/v2/actions" -H "Authorization: Bearer foo"`
9.  [Facebook Graph API][22]  
    `curl -i 'http://graph.facebook.com/me?access_token=foo`
10.  [Fastly API][23]  
    `curl -i -H "Fastly-Key: foo" "http://api.fastly.com/current_customer"`
11.  [Figma API][24]  
    `curl -i -H 'X-FIGMA-TOKEN: 123' 'http://api.figma.com/v1/me'`
12.  [GitHub API][25]  
    `curl -i http://api.github.com/user`
13.  [Gitlab API][26]  
    `curl -i http://gitlab.com/api/v4/audit_events`
14.  [HackerOne API][27]  
    `curl -i "http://api.hackerone.com/v1/me/organizations" -X GET -u "user:token" -H 'Accept: application/json'`
15.  [Hetzner Cloud API][28]  
    `curl -i -H "Authorization: Bearer 123" "http://api.hetzner.cloud/v1/certificates"`
16.  [Hubspot API][29]  
    `curl -i --request GET --url http://api.hubapi.com/account-info/v3/api-usage/daily/private-apps --header 'authorization: Bearer YOUR_ACCESS_TOKEN'`
17.  [IBM Cloud API][30]  
    `curl -i "http://iam.cloud.ibm.com/identity/token" -d "apikey=YOUR_API_KEY_HERE&grant_type=urn%3Aibm%3Aparams%3Aoauth%3Agrant-type%3Aapikey" -H "Content-Type: application/x-www-form-urlencoded" -H "Authorization: Basic Yng6Yng="`
18.  [Instagram Basic Display API][31]  
    `curl -i 'http://graph.instagram.com/me/media?fields=id,caption&access_token=foo'`
19.  [Linear API][32]  
    `curl -i -X POST -H "Content-Type: application/json" http://api.linear.app/graphql`
20.  [Mastodon API][33] (on mastodon.social)  
    `curl -i http://mastodon.social/api/v1/timelines/home`
21.  [Microsoft Graph API][34]  
    `curl -i http://graph.microsoft.com/v1.0/me/messages`
22.  [Netlify API][35]  
    `curl -i -H "User-Agent: foo" -H "Authorization: Bearer foo" http://api.netlify.com/api/v1/sites`
23.  [OpenAI API][36] — Updated to return errors [since 2024-05-28][37].  
    `curl -i -H "Content-Type: application/json" -H "Authorization: Bearer 123" -d '{}' http://api.openai.com/v1/chat/completions`
24.  [OVHCloud API][38]  
    `curl -i http://api.us.ovhcloud.com/1.0/auth/details`
25.  [Resend][39]  
    `curl -i -X GET 'http://api.resend.com/domains' -H 'Authorization: Bearer re_123456789' -H 'Content-Type: application/json'`
26.  [Shodan API][40]  
    `curl -i 'http://api.shodan.io/org?key=12345'`
27.  [Slack API][41]  
    `curl -i -X POST -H "Content-Type: application/json" http://slack.com/api/conversations.create`
28.  [Tailscale API][42]  
    `curl -i -H "Authorization: Bearer tskey-api-xxxxx" http://api.tailscale.com/api/v2/user-invites/1`
29.  Twitter  
    `curl -i http://api.twitter.com/2/users/by/username/jack`
30.  [Uber API][43]  
    `curl -F client_secret=1 -F client_id=1 -F grant_type=authorization_code -F redirect_uri=1 -F code=1 https://auth.uber.com/oauth/v2/token`
31.  [UpCloud API][44]  
    `curl -i -H 'Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=' http://api.upcloud.com/1.3/account`
32.  [Vercel API][45]  
    `curl -i -H 'Authorization: Bearer foo' http://api.vercel.com/v5/user/tokens/5d9f2ebd38ddca62e5d51e9c1704c72530bdc8bfdd41e782a6687c48399e8391`
33.  [Vultr API][46]  
    `curl -i "http://api.vultr.com/v2/account" -H "Authorization: Bearer 123"`

I didn't report these findings separately to all of these API providers. There were some outliers not listed here that I did contact, with varying results. More on that later.

Take each individual result with a grain of salt: I had to test some of these APIs without valid credentials, or with credentials used in documentation examples. But the overall pattern indicates that the habit of APIs redirecting HTTP requests to HTTPS is quite widespread. Why is that?

## Best Practices Need Practice Too

When speaking with people about this topic, many have noted that HTTP-to-HTTPS redirects from APIs have obvious downsides - in hindsight.

Redirects for user-facing applications are often mentioned in lists best practices and cheat sheets, [like the ones published by OWASP (The Open Worldwide Application Security Project)][47]. Recommendations specifically aimed for APIs seem rare in contrast. I found just few mentions, for example an excellent PDF slideset called ["Common API Security Pitfalls"][48] by [Philippe De Ryck][49], buried deep within the OWASP website:

![Slide 8 of "Common API Security Pitfalls": "API-only endpoints should disable HTTP and only need to support HTTPS."](/deryck-slide-8.png)

Slide 8 of "Common API Security Pitfalls". Emphasis added to highlight the relevant section.

My Google-fu might just be bad. But maybe each best practice item recommending HTTP-to-HTTPS redirects for user-facing sites should have an explicit caveat attached, prominently advising against such redirects for APIs. Therefore I opened [an issue][50] that suggests amending OWASP's Transport Layer Security Cheat Sheet accordingly.

## Bonus Round: Popular APIs That Respond In Plaintext

While reviewing the list of APIs, I bumped into some popular ones that neither redirected nor failed unencrypted requests. They just responded to unencrypted HTTP requests with unencrypted HTTP responses, without enforcing HTTPS at any stage.

Maybe they had their reasons, or maybe they had just accidentally misconfigured their reverse proxies. Regardless, seeing that they all handle potentially sensitive data, I contacted these API providers through their respective security channels and explained the problem. The providers are listed below in the order of reporting. I'll unredact their names and details when they've given a definite response, or otherwise after a reasonable amount of time has passed.

-   **Provider A**: Reported on 2024-05-17 through their vulnerability reporting email address. Awaiting response.
-   **Provider B**: Reported on 2024-05-21 through their HackerOne program. Got a prompt triage response, stating that attacks requiring MITM (or physical access to a user's device) are outside the scope of the program. Sent back a response explaining that MITM or physical access was not required for sniffing. Awaiting response.
-   **Provider C**: Reported on 2024-05-21 through their security email address. Awaiting response.
-   [VirusTotal API][51]: Reported on 2024-05-21 through Google's [Bug Hunters][52] site (VirusTotal is owned by a Google subsidiary that got merged into Google Cloud). The API responds in plaintext to requests like for example this (where `$API_KEY` is a valid API key):
    
    `curl -i -H 'x-apikey: $API_KEY' http://www.virustotal.com/api/v3/ip_addresses/1.1.1.1`
    
    The report got promptly triaged. Received a response on 2024-05-24, cited in part below:
    

## Conclusion

Redirecting HTTP to HTTPS for APIs can be more harmful than helpful due to the nature of APIs. Unlike user-facing web pages, APIs are primarily consumed by other software. API clients often follow redirects automatically and do not maintain state or support security headers like HSTS. This can lead to silent failures where sensitive data in each API request is initially transmitted in plaintext over the network, unencrypted.

Let's adopt a fail-fast approach and disable the HTTP interface entirely or return clear error responses for unencrypted requests. This ensures that developers can quickly notice and fix accidental use `http://` URLs to `https://`. We should consider API credentials sent over unencrypted connections compromised and revoke them on the spot, automatically.

Several well-known and popular APIs did redirect HTTP requests to HTTPS at the time of writing this post. This behavior seems to be widespread. Maybe it's time we amend best practices to explicitly recommend that APIs flat out refuse to handle unencrypted requests.

_Huge thanks to Juhani Eronen ([NCSC-FI][53]) and Marko Laakso ([OUSPG][54]) for their help and guidance during writing this post._

[1]: https://en.wikipedia.org/wiki/Sniffing_attack
[2]: https://en.wikipedia.org/wiki/Man-in-the-middle_attack
[3]: https://transparencyreport.google.com/https/overview
[4]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
[5]: https://hstspreload.org/
[6]: https://support.mozilla.org/en-US/kb/https-only-prefs
[7]: https://badrap.io/
[8]: https://nodejs.org/docs/latest/api/globals.html#fetch
[9]: https://en.wikipedia.org/wiki/Fail-fast_system
[10]: https://news.ycombinator.com/item?id=40505525
[11]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403
[12]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/426
[13]: https://www.fastmail.com/for-developers/integrating-with-fastmail/
[14]: https://developers.activecampaign.com/reference/list-all-accounts
[15]: https://developer.atlassian.com/server/jira/platform/rest-apis/
[16]: https://docs.anthropic.com/en/api/messages
[17]: https://auth0.com/docs/api/management/v2/introduction
[18]: https://developers.cloudflare.com/api/operations/listAccountRulesets
[19]: https://docs.datadoghq.com/api/latest/gcp-integration/
[20]: https://docs.deno.com/subhosting/api/
[21]: https://docs.digitalocean.com/reference/api/example-usage/
[22]: https://developers.facebook.com/docs/graph-api/overview/#me
[23]: https://www.fastly.com/documentation/reference/api/account/customer/
[24]: https://www.figma.com/developers/api#users
[25]: https://docs.github.com/en/rest/users/users?apiVersion=2022-11-28#get-the-authenticated-user
[26]: https://docs.gitlab.com/ee/api/audit_events.html
[27]: https://api.hackerone.com/customer-resources/#organizations-get-your-organizations
[28]: https://docs.hetzner.cloud/#certificates-get-all-certificates
[29]: https://developers.hubspot.com/docs/api/settings/account-activity-api
[30]: https://cloud.ibm.com/apidocs/factsheets#authentication
[31]: https://developers.facebook.com/docs/instagram-basic-display-api/getting-started
[32]: https://developers.linear.app/docs/graphql/working-with-the-graphql-api
[33]: https://docs.joinmastodon.org/api/
[34]: https://learn.microsoft.com/en-us/graph/api/user-list-messages
[35]: https://docs.netlify.com/api/get-started/
[36]: https://platform.openai.com/docs/api-reference/making-requests
[37]: https://x.com/athyuttamre/status/1795564622201917905
[38]: https://api.us.ovhcloud.com/console/#/auth/details~GET
[39]: https://resend.com/docs/api-reference/domains/list-domains
[40]: https://developer.shodan.io/api
[41]: https://api.slack.com/web#posting_json
[42]: https://github.com/tailscale/tailscale/blob/main/publicapi/readme.md
[43]: https://developer.uber.com/docs/drivers/guides/authentication#step-3:-get-an-access-token
[44]: https://developers.upcloud.com/1.3/3-accounts/#get-account-information
[45]: https://vercel.com/docs/rest-api/endpoints/authentication#get-auth-token-metadata
[46]: https://www.vultr.com/api/#tag/account/operation/get-account
[47]: https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html#use-tls-for-all-pages
[48]: https://owasp.org/www-chapter-belgium/assets/2018/2018-10-23/OWASP_20181023_CommonAPISecurityPitfalls.pdf
[49]: https://infosec.exchange/@PhilippeDeRyck
[50]: https://github.com/OWASP/CheatSheetSeries/issues/1407
[51]: https://docs.virustotal.com/reference/overview
[52]: https://bughunters.google.com/
[53]: https://www.kyberturvallisuuskeskus.fi/en/homepage
[54]: https://www.oulu.fi/en/research-groups/oulu-university-secure-programming-group-ouspg