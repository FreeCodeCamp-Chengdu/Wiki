---
title: 你的API不应该将HTTP重定向到HTTPS
date: 2024-05-23T00:00:00.000Z
authorURL: ""
originalURL: https://jviide.iki.fi/http-redirects
translator: ""
reviewer: ""
---

## 背景

<!-- more -->

当用户将他们的网络浏览器指向 HTTP URL 时，服务通常会将请求重定向到相应的 HTTPS 页面。这种通信流程中的未加密部分存在缺陷。共享网络中的第三方以及网络中介可能会[嗅探][1]初始 HTTP 流量中的密码和其他机密信息，甚至可能通过[中间人攻击][2]冒充网络服务器。尽管如此，重定向一直是从早期大部分未加密的网络向如今[大部分加密][3]的网络过渡的有用第一步。

后来的技术进一步加强了安全性。服务器现在可以在初始 HTTP 到 HTTPS 重定向响应中发送[HSTS][4]，告诉用户的浏览器从此只对该域名使用 HTTPS。这将简单嗅探和中间人攻击的机会窗口限制在第一个请求。浏览器随后添加了[HSTS 预加载列表][5]和[仅 HTTPS 模式][6]，允许完全跳过初始未加密请求。

从可用性-安全性权衡的角度来看，这对面向用户的网站是有意义的。但有趣的是，重定向方法似乎也被广泛应用于 API。API 主要被其他软件使用，因此相同的可用性论点在这里并不适用。此外，许多程序化 API 客户端不会像浏览器那样保持它们所见 HSTS 头部的状态。

本文认为，由于这些因素，应该重新考虑将 API 调用从 HTTP 重定向到 HTTPS 的常见做法。虽然本文主要指的是 REST API，但其观点也适用于使用 HTTP(S)作为传输机制的其他类型的 API。

## 一个简单的拼写错误就足够了

在[工作中][7]，我们正在构建一个针对第三方 API 的新集成。初始代码提交包含了一个错误输入的 API 基础 URL `"http://..."` 而不是 `"https://..."`。这是一个很容易犯的错误。

这个错误在运行时基本上被掩盖了：第三方 API 对每个请求都以 301 重定向到他们的 HTTPS 端点进行响应。Node.js 内置的[fetch][8]愉快且 _悄无声息地_ 跟随这些重定向到 HTTPS 端点。

**我们的每一个 API 请求现在都以明文形式通过网络发送 API 密钥**，然后再次将它们发送到加密端点。这一个字母的遗漏可能会在我们不知情的情况下将 API 密钥暴露给第三方。由于集成可以正常工作，这段代码很可能会在多年内泄露 API 调用中的任何机密。从长远来看，恶意行为的概率往往会累积。

幸运的是，我们在代码审查期间发现了这个错误，在错误传播到生产环境甚至测试环境之前。我们还意识到我们_自己的_API 也做了类似的 HTTP 到 HTTPS 重定向。

## 快速失败原则

当 API 将 HTTP 请求重定向到 HTTPS，而 API 客户端静默地跟随这些重定向时，它往往会隐藏像上述情况中错误输入的 URL。一个简单的单字母遗漏很容易被忽视，最终进入生产环境，并危及整个系统的机密性。

在大多数情况下，最好遵循[快速失败原则][9]：未加密的 API 调用应该以引人注目且可见的方式失败，以便开发人员能够在开发过程中尽早发现并修复拼写错误。

快速失败的一个很好的解决方案是完全禁用 API 服务器的 HTTP 接口，甚至不回应 80 端口的连接尝试。如果初始未加密连接从未建立，那么 API 密钥就不会被发送，从而减轻嗅探攻击，并将中间人攻击的机会窗口限制在极小的时间范围内。这种方法适用于在自己域名下托管的 API，如`api.example.com`。

我们自己的 API 是在与服务的 Web UI 相同的域名下的`/api`路径提供的。因此，我们不敢完全禁用 HTTP 接口，所以我们选择了第二种选择：现在所有在`/api`路径下进行的未加密 HTTP 请求都会返回一个描述性错误消息，以及 HTTP 状态码 403。在开发过程中可能会进行一些初始明文请求，但开发人员更容易注意到它们。

在本文最初发表后，Hacker News 用户 u/zepton[提出了一个很好的观点：][10]

确实！任何通过公共互联网未加密发送的 API 密钥都应该被视为已泄露，并立即自动撤销。API 响应中的错误消息是一个很好的地方，可以通知 API 消费者修复他们的 URL 并在之后获取新的密钥。

## 还有谁？

这解决了我们自己的 API 问题。我们还联系了第三方 API 提供商和几个朋友，他们可能想检查一下他们的 API。谁知道，也许有一些常用的 API 接受 API 密钥（或其他凭证）并且也从 HTTP 重定向到 HTTPS？

我列出了一堆我能想到的知名 API，并做了一个小调查。其中几个返回了 HTTP 错误或完全拒绝连接。它们在这里列出，并附有 cURL 命令以查看其详细响应：

1.  **Stripe API**：响应[403][11]（"禁止"）和一个描述性错误消息。  
    ```bash
    curl -i http://api.stripe.com
    ```
2.  **Google Cloud API**：响应 403 和一个描述性错误消息。  
    ```bash
    curl -i http://compute.googleapis.com/compute/v1/projects/project/regions/region/addresses
    ```
3.  **Shopify API**：响应 403 和一个描述性错误消息。  
    ```bash
    curl -i http://shop.myshopify.com/admin/api/2021-07/shop.json
    ```
4.  **NPM Registry API**：响应[426][12]（"需要升级"）和一个描述性错误消息。  
    ```bash
    curl -i -X PUT -H 'content-type: application/json' -d '{}' 'http://registry.npmjs.org/-/user/org.couchdb.user:npm
    ```
5.  [Fastmail JMAP API][13]：整个 HTTP 接口似乎被禁用。  
    ```bash
    curl -i -H 'Authorization: Bearer foo' http://api.fastmail.com/jmap/session
    ```
6.  **Mailjet**：套接字响应完全空的负载。  
    ```bash
    curl -i -X POST --user "user:pass" http://api.mailjet.com/v3.1/send -H 'Content-Type: application/json' -d '{}'
    ```

然而，以下 API_确实_以 HTTP 到 HTTPS 重定向进行响应：

1.  [ActiveCampaign API][14]  
    ```bash
    curl -i -H "Api-Token: 123abc-def-ghi" http://123456demo.api-us1.com/api/3/accounts
    ```
2.  [Atlassian Jira REST API][15]  
    ```bash
    curl -i http://jira.atlassian.com/rest/api/latest/issue/JRA-9
    ```
3.  [Anthropic API][16]  
    ```bash
    curl -i http://api.anthropic.com/v1/messages --header "x-api-key: 1" --header "anthropic-version: 2023-06-01" --header "content-type: application/json" --data '{}'
    ```

4.   [HackerOne API][27]  
     ```bash
     curl -i "http://api.hackerone.com/v1/me/organizations" -X GET -u "user:token" -H 'Accept: application/json'
     ```
5.   [Hetzner Cloud API][28]  
     ```bash
     curl -i -H "Authorization: Bearer 123" "http://api.hetzner.cloud/v1/certificates"
     ```
6.   [Hubspot API][29]  
     ```bash
     curl -i --request GET --url http://api.hubapi.com/account-info/v3/api-usage/daily/private-apps --header 'authorization: Bearer YOUR_ACCESS_TOKEN'
     ```
7.   [IBM Cloud API][30]  
     ```bash
     curl -i "http://iam.cloud.ibm.com/identity/token" -d "apikey=YOUR_API_KEY_HERE&grant_type=urn%3Aibm%3Aparams%3Aoauth%3Agrant-type%3Aapikey" -H "Content-Type: application/x-www-form-urlencoded" -H "Authorization: Basic Yng6Yng="
     ```
8.   [Instagram Basic Display API][31]  
     ```bash
     curl -i 'http://graph.instagram.com/me/media?fields=id,caption&access_token=foo'
     ```
9.   [Linear API][32]  
     ```bash
     curl -i -X POST -H "Content-Type: application/json" http://api.linear.app/graphql
     ```
10.  [Mastodon API][33] (on mastodon.social)  
     ```bash
     curl -i http://mastodon.social/api/v1/timelines/home
     ```
11.  [Microsoft Graph API][34]  
     ```bash
     curl -i http://graph.microsoft.com/v1.0/me/messages
     ```
12.  [Netlify API][35]  
     ```bash
     curl -i -H "User-Agent: foo" -H "Authorization: Bearer foo" http://api.netlify.com/api/v1/sites
     ```
13.  [OpenAI API][36] — 更新为返回错误 [自 2024-05-28 起][37]。  
     ```bash
     curl -i -H "Content-Type: application/json" -H "Authorization: Bearer 123" -d '{}' http://api.openai.com/v1/chat/completions
     ```
14.  [OVHCloud API][38]  
      ```bash
     curl -i http://api.us.ovhcloud.com/1.0/auth/details
     ```
15.  [Resend][39]  
     ```bash
     curl -i -X GET 'http://api.resend.com/domains' -H 'Authorization: Bearer re_123456789' -H 'Content-Type: application/json'
     ```
16.  [Shodan API][40]  
     ```bash
     curl -i 'http://api.shodan.io/org?key=12345'
     ```
17.  [Slack API][41]  
     ```bash
     curl -i -X POST -H "Content-Type: application/json" http://slack.com/api/conversations.create
     ```
18.  [Tailscale API][42]  
     ```bash
     curl -i -H "Authorization: Bearer tskey-api-xxxxx" http://api.tailscale.com/api/v2/user-invites/1
     ```
19.  Twitter  
     ```bash
     curl -i http://api.twitter.com/2/users/by/username/jack
     ```
20.  [Uber API][43]  
     ```bash
     curl -F client_secret=1 -F client_id=1 -F grant_type=authorization_code -F redirect_uri=1 -F code=1 http://auth.uber.com/oauth/v2/token
     ```
21.  [UpCloud API][44]  
     ```bash
        curl -i -H 'Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=' http://api.upcloud.com/1.3/account
     ```
22.  [Vercel API][45]  
     ```bash
     curl -i -H 'Authorization: Bearer foo' http://api.vercel.com/v5/user/tokens/5d9f2ebd38ddca62e5d51e9c1704c72530bdc8bfdd41e782a6687c48399e8391
     ```
23.  [Vultr API][46]  
     ```bash
     curl -i "http://api.vultr.com/v2/account" -H "Authorization: Bearer 123"
     ```

我没有单独向所有这些 API 提供商报告这些发现。这里没有列出的一些特例我确实联系了，结果各不相同。稍后会详细介绍。

对每个单独的结果持保留态度：我不得不在没有有效凭证的情况下测试一些 API，或者使用文档示例中的凭证。但总体模式表明，API 重定向 HTTP 请求到 HTTPS 的习惯相当普遍。为什么会这样呢？

## 最佳实践也需要实践

在与人们讨论这个话题时，许多人指出 API 从 HTTP 到 HTTPS 的重定向有明显的缺点 - 事后看来是这样。

面向用户的应用程序的重定向经常在最佳实践列表和备忘单中提及，[比如 OWASP（开放式 Web 应用程序安全项目）发布的那些][47]。相比之下，专门针对 API 的建议似乎很少。我只找到了几处提及，例如[Philippe De Ryck][49] 的一个名为["常见 API 安全陷阱"][48]的优秀 PDF 幻灯片集，深藏在 OWASP 网站内：

![幻灯片8，"常见API安全陷阱"："仅API的端点应禁用HTTP，只需支持HTTPS。"][55]

"常见 API 安全陷阱"的幻灯片 8。添加强调以突出相关部分。

我的 Google 搜索能力可能不太好。但也许每个推荐面向用户的站点进行 HTTP 到 HTTPS 重定向的最佳实践项目都应该附带一个明确的警告，明确建议 API 不要进行此类重定向。因此，我开了[一个问题][50]，建议相应修改 OWASP 的传输层安全备忘单。

## 额外内容：以明文响应的流行 API

在审查 API 列表时，我遇到了一些既不重定向也不拒绝未加密请求的流行 API。它们只是以未加密的 HTTP 响应回应未加密的 HTTP 请求，没有在任何阶段强制使用 HTTPS。

也许他们有他们的理由，或者也许他们只是意外地错误配置了他们的反向代理。无论如何，看到它们都处理潜在的敏感数据，我通过各自的安全渠道联系了这些 API 提供商并解释了问题。提供商按照报告顺序列出。当他们给出明确回应时，或者在合理的时间过去后，我会取消对他们名称和详细信息的编辑。

-   **提供商 A**：2024-05-17 通过他们的漏洞报告电子邮件地址报告。等待回应。
-   **提供商 B**：2024-05-21 通过他们的 HackerOne 计划报告。得到了迅速的分类响应，声明需要中间人攻击（或对用户设备的物理访问）的攻击超出了计划范围。发回解释不需要中间人攻击或物理访问进行嗅探的回应。等待回应。
-   **提供商 C**：2024-05-21 通过他们的安全电子邮件地址报告。等待回应。
-   [VirusTotal API][51]：2024-05-21 通过 Google 的[Bug Hunters][52]网站报告（VirusTotal 由 Google 子公司拥有，该子公司已合并到 Google Cloud）。API 以明文响应请求，例如（其中`$API_KEY`是有效的 API 密钥）：
    
    ```bash
    curl -i -H 'x-apikey: $API_KEY' http://www.virustotal.com/api/v3/ip_addresses/1.1.1.1
    ```
    
    报告迅速被分类。2024-05-24 收到回应，部分引用如下：
    

## 结论

由于 API 的性质，将 HTTP 重定向到 HTTPS 对 API 的危害可能大于帮助。与面向用户的网页不同，API 主要由其他软件使用。API 客户端通常会自动跟随重定向，并且不维护状态或支持 HSTS 等安全头部。这可能导致静默失败，每个 API 请求中的敏感数据最初以明文形式通过网络传输，未加密。

让我们采用快速失败方法，完全禁用 HTTP 接口或对未加密的请求返回明确的错误响应。这确保开发人员可以快速注意到并修复意外使用`http://`URL 到`https://`的情况。我们应该认为通过未加密连接发送的 API 凭证已被泄露，并立即自动撤销它们。

在撰写本文时，几个知名和流行的 API 确实将 HTTP 请求重定向到 HTTPS。这种行为似乎很普遍。也许是时候我们修改最佳实践，明确建议 API 完全拒绝处理未加密的请求。

_非常感谢 Juhani Eronen ([NCSC-FI][53])和 Marko Laakso ([OUSPG][54])在撰写本文期间的帮助和指导。_

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
[55]: https://jviide.iki.fi/deryck-slide-8.png