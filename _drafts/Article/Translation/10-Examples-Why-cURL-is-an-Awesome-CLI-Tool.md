---
title: 10 个示例：为什么 cURL 是一个很棒的 CLI 工具
date: 2024-08-01T00:00:00.000Z
authorURL: ""
originalURL: https://martinheinz.dev/blog/113
translator: ""
reviewer: ""
---

无论你是开发者、DevOps 工程师、系统管理员、QA 还是其他任何技术岗位，你肯定都熟悉 cURL —— _这个用于通过 URL 传输数据的命令行工具和库_（正如文档中所述）。

然而，大多数时候，我们实际上只使用 `curl` 来完成一些简单的任务，比如下载文件或检查网站是否可访问，但 `curl` 还可以做更多的事情！

在本文中，我们将详细介绍那些很酷的示例和技巧，以展示为什么 `curl` 是一个很棒但被低估的工具。


## 通配符匹配


首先是 _通配符匹配_，它允许我们使用单个 `curl` 命令发出多个请求： 


```shell
curl -s "https://jsonplaceholder.typicode.com/users/[1-3]" | jq -s .
curl -s "https://jsonplaceholder.typicode.com/users/[0-10:2]" | jq -s .

curl -s "https://jsonplaceholder.typicode.com/photos/{1,6,35}" | jq -s .

curl -s "https://jsonplaceholder.typicode.com/users/[1-3]" -o "file_#1.json"
```

前两个命令展示了我们如何运行一系列请求 - 第一个命令生成对 `.../users/1`、`.../users/2` 和 `.../users/3` 的请求，而另一个命令使用 _step_ 选项，它给了我们 2、4、6、8 和 10。考虑到这些特定请求返回 JSON，我们还将它与带有 `-s` (_slurp_) 运算符的 `jq ...` 组合在一起，该运算符将各个请求的响应连接到单个数组中。

第三个示例使用特定数字列表而不是范围 - 这也适用于字符和单词 - 例如，我们可以使用文件名扩展来发出具有多种协议的请求：`{http,https}://...`

最后一个示例将文件名扩展与输出变量相结合，其中文件名的 `#1` 变量指的是范围 `[1-3]`。这将生成 `file_1.json`、`file_2.json` 和 `file_3.json`。 

## 配置文件

大多数情况下，在使用 `curl` 时，我们可能希望传入相同的命令行选项，例如代理设置、请求超时、标头等。这就是 `curl` 配置文件 `.curlrc` 可能派上用场的地方： 

```shell
# cat ~/.curlrc
```
```text
# some headers
-H "Upgrade-Insecure-Requests: 1"
-H "Accept-Language: en-US,en;q=0.8"

# follow redirects
--location
```

它只是一个文本文件，其中每一行代表一个将传递给 `curl` 的选项。它会从 `~/.curlrc` 自动读取，因此您不需要任何额外的标志，但您可以使用 `-K` 来覆盖或指定不同的位置，例如： 


```shell
curl -K .curlrc https://google.com
```

与标志和选项类似，您可能还想传入凭据。这可以使用 `--user` 选项来完成，但这会将凭据留在 shell 历史记录中，因此我们可以改用 `curl` 支持的 `.netrc` 文件： 

```shell
# ~/.netrc
machine https://authenticationtest.com/HTTPAuth/
login user
password pass
```

格式包括 `machine`（URL）、`login` 和 `password` - 它们可以位于单行上，也可以如上所示，并且我们还可以在单个文件中包含多个。要使用它，只需将其传递给 `curl`，如下所示： 

```shell
curl --netrc-file .netrc https://authenticationtest.com/HTTPAuth/
```

##  并发请求
我们已经在 globbing 部分讨论了请求范围，但是并行化呢？嗯，`curl` 也可以做到： 


```shell
curl -I --parallel --parallel-immediate --parallel-max 3 --config websites.txt

curl -I --parallel --parallel-immediate --parallel-max 3 stackoverflow.com google.com example.com
```

我们所需要做的就是添加 `--parallel`（或 `-Z`），`curl` 将打开最多 50 个并行连接（可以使用 `--parallel-max N` 更改）。还要注意我们如何提供 URL - 第一个选项是 `--config` 和一个文本文件，如下所示： 
```shell
url = "stackoverflow.com"
url = "google.com"
url = "example.com"
```

另一个选择是将所有 URL 放在命令行上。这两个选项也适用于非并行请求！ 

## 格式化和变量 

`curl` 可以输出很多东西，有时这些内容可能过多、冗长且不必要。幸运的是，我们可以使用输出格式来只打印我们感兴趣的内容：


```shell
curl --silent --output /dev/null --show-error -w @format.txt http://example.com/
```
```text
# Type: text/html; charset=UTF-8
# Code: 200
#
# From 8.1.0:
# Scheme: http
# Host: example.com
# Port: 80
#
# Read header content (v7.83.0):
# Server: Sat, 29 Jun 2024 13:01:30 GMT
```

我们使用 `-w` 选项并传入一个格式文件来做到这一点。要生成如上所示的输出，我们可以使用：
```text
# format.txt
Type: %{content_type}\nCode: %{response_code}\n\n

From 8.1.0:\n\n

Scheme: %{url.scheme}\n
Host: %{url.host}\n
Port: %{url.port}\n

Read header content (v7.83.0):\n
%header{date}
```

每个变量都包含在 `%{...}` 中。它们可以是简单的变量，如 `response_code`，也可以是 `url.<NAME>` 的一部分，后者指的是 URL 组件，如主机或端口。最后，我们还可以使用 `%header{HEADER_NAME}` 变量输出响应头。

格式化的许多良好用例之一是测量请求/响应时间，这可以使用以下格式完成：

```text
# format.txt
     time_namelookup:  %{time_namelookup}s\n
        time_connect:  %{time_connect}s\n
     time_appconnect:  %{time_appconnect}s\n
    time_pretransfer:  %{time_pretransfer}s\n
       time_redirect:  %{time_redirect}s\n
  time_starttransfer:  %{time_starttransfer}s\n
                     ----------\n
          time_total:  %{time_total}s\n

# Output:
     time_namelookup:  0.000765s
        time_connect:  0.111908s
     time_appconnect:  0.000000s
    time_pretransfer:  0.111967s
       time_redirect:  0.000000s
  time_starttransfer:  0.223373s
                     ----------
          time_total:  0.223992s
```

有关变量的完整列表，查阅[文档](https://everything.curl.dev/usingcurl/verbose/writeout.html#available---write-out-variables).

## 测试和故障排除
使用 `curl` 最常见的情况是进行（网络）故障排除 - 通常情况下，只需向特定 URL 发出请求就足以提供足够的信息，但我们还可以做更多的事情，例如，我们可以强制使用特定的本地网络接口：

```shell
ip link show
```
```text
# ...
# 3: wlp5s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000

```shell
curl --interface wlp5s0 https://example.com
```

同样，可以强制使用特定的 DNS 服务器：
```shell
curl --dns-ipv4-addr 1.1.1.1 https://example.com
```

或者我们可以测试超时并捕获 [退出状态码](https://everything.curl.dev/cmdline/exitcode.html)：


```shell
curl --connect-timeout 30 --silent --output /dev/null \
  --show-error -w 'Total: %{time_total}s\n' http://google.com/ || EXIT_CODE=$?

if [ $EXIT_CODE = 28 ]
then
  echo "Cannot connect (Timeout)."
else
  echo "Can connect."
fi
```

这很有用 - 例如 - 用于测试代理服务器是否正常工作（使用 `-x http://proxy.example.com:80`）。 

##  Trurl

`curl` 不仅仅是一个命令行工具——该项目还包括 `libcurl` 以及我想在这里展示的 `trurl`。

[`trurl`](https://curl.se/trurl/) 是一个专门用于解析 URL 的工具，是 `curl` 的兄弟项目。它可以从源代码安装：


```shell
sudo apt-get install libcurl4-openssl-dev
git clone https://github.com/curl/trurl.git
cd trurl
make
# ...
```

以下是一些使用它的示例：

```shell
trurl --url https://example.com/some/path/to/file.html --get '{path}'
# /some/path/to/file.html

trurl --url "https://example.com/?name=hello" --append query=key=value
# https://example.com/?name=hello&key=value

# Parse as JSON:
./trurl --url "https://example.com/?name=hello" --json
# [
#   {
#     "url": "https://example.com/?name=hello",
#     "parts": {
#       "scheme": "https",
#       "host": "example.com",
#       "path": "/",
#       "query": "name=hello"
#     },
#     "params": [
#       {
#         "key": "name",
#         "value": "hello"
#       }
#     ]
#   }
# ]
```

第一个例子展示了我们如何提取 URL 组件，这里提取的是路径，但也可以是例如 url、scheme、user、password、options 或 host。

第二个例子使用了 _append_ 功能，向 URL 添加查询参数。

最后一个例子展示了 `--json` 选项，它将解析后的 URL 作为 JSON 输出，这对于进一步处理非常有用。

`trurl` 还可以做更多的事情——你可以查看[这个视频](https://www.youtube.com/watch?v=oDL7DVszr2w)或[手册](https://curl.se/trurl/manual.html)（示例在底部）。


## 发送/上传数据

大多数情况下，我们使用 `curl` 来下载或请求数据，但它（显然）也可以发送数据。使用 `curl` 发送 POST 数据并不是什么新鲜事，对吧？ 

```shell
curl -X POST "https://httpbin.org/post" -H "accept: application/json" --json '{"key": "value"}'
```

但是像这样发送 JSON，必须交替使用单引号和双引号很快就会变得很烦人，但是有一种更好的方法：

```shell
jo -p key=value | curl -X POST "https://httpbin.org/post" -H "accept: application/json" --json @-
```

我想我们都熟悉使用 `jq` 解析 `curl` 的 JSON 输出，但是反过来呢？- 在上面，我们使用 `jo` 工具轻松创建 JSON，然后可以使用 `--json` 选项将其传递给 `curl`。

当然，`--json` 选项也可以从文件中获取输入，例如使用 `--json @data.json`。


## 协议

最后但并非最不重要的是协议 - 通常我们只使用 HTTP 或 HTTPS，但 `curl` 支持[_很多协议_](https://everything.curl.dev/protocols/protocols.html#what-other-protocols-are-there)。

我想特别提一下的是 `telnet`，它可以方便地测试服务器是否在特定端口上监听，但是如果您在没有并且无法安装 `telnet` 的服务器/机器上怎么办？只需使用 `curl`： 

```shell
# Same as telnet example.com 1234
curl telnet://example.com:1234
```

一些更不为人知（有趣）的协议选项是用于电子邮件的 IMAP、POP3 和 SMTP，这意味着您可以使用 `curl` _读取_和发送电子邮件。要阅读它们：

```shell
curl --url "imaps://imap.gmail.com:993/Inbox;UID=1" --user "user@gmail.com:PASSWORD"
```

要使其专门用于 Gmail，您需要创建[应用密码](https://support.google.com/mail/answer/185833?hl=en)，它不如普通密码安全。如果您_真的_想尝试这个，请查看[Gmail IMAP 文档](https://developers.google.com/gmail/imap/imap-extensions)和[这些查询](https://gist.github.com/akpoff/53ac391037ae2f2d376214eac4a23634)以获取灵感。

要发送电子邮件，您可以使用：

```shell
curl smtp://mail.example.com \
  --mail-from me@example.com \
  --mail-rcpt someone@example.com \
  --upload-file message.txt \
  -u "me@example.com:PASSWORD"
```

这里的 `message.txt` 是实际的电子邮件，它需要遵循特定的格式，请查看[此页面](https://everything.curl.dev/usingcurl/smtp.html)以获取示例。 

## 总结

我们到最后了，我敢肯定至少有 10 个例子（我不再计数了）。但老实说，这真的只是触及了皮毛——我们甚至还没有触及 `libcurl`，它是 `curl` 的_重要_部分。

使用 `curl` 还可以做更多的事情，所以我建议同时浏览 [文档](https://curl.se/docs/manpage.html) 和 [https://everything.curl.dev/](https://everything.curl.dev/)。 
