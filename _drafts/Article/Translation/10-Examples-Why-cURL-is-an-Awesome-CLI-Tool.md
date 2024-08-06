---
title: 10 Examples Why cURL is an Awesome CLI Tool
date: 2024-08-01T00:00:00.000Z
authorURL: ""
originalURL: https://martinheinz.dev/blog/113
translator: ""
reviewer: ""
---

Whether you're developer, DevOps engineer, SysAdmin, QA or in any other technical role, you're surely familiar with cURL - _the command line tool and library for transferring data with URLs_ (as described in docs).

Most of the time however, we all really only use `curl` for simple tasks, such downloading a file or checking if website is accessible, yet there's some much more `curl` can do!

And in this article we will go through exactly those cool examples and tricks to showcase why `curl` is awesome and underappreciated tool...

## Globbing


First up is _globbing_, allowing us to make multiple requests with a single `curl` command:

```

curl -s "https://jsonplaceholder.typicode.com/users/[1-3]" | jq -s .
curl -s "https://jsonplaceholder.typicode.com/users/[0-10:2]" | jq -s .

curl -s "https://jsonplaceholder.typicode.com/photos/{1,6,35}" | jq -s .

curl -s "https://jsonplaceholder.typicode.com/users/[1-3]" -o "file_#1.json"
```

The first two commands show how we can run a range of requests - first command produces requests to `.../users/1`, `.../users/2`, and `.../users/3`, while the other uses a _step_ option, which gives us 2, 4, 6, 8 and 10. Considering that these particular requests return JSON, we also combine it with `jq ...` with the `-s` (_slurp_) operator, which joins responses of individual requests into single array.

Third examples uses list of specific numbers instead of range - this also works for characters and words - for example, we could use globbing to make requests with multiple protocols: `{http,https}://...`

The final example combines globbing with output variables, where the `#1` variable in filename refers to the range `[1-3]`. This will produce `file_1.json`, `file_2.json` and `file_3.json`.

## Configuration Files


Most of the time, when using `curl` we probably want to pass in same commandline options, such as proxy settings, request timeouts, headers, etc. That's where `curl` configuration file named `.curlrc` might come in handy:

```

# ~/.curlrc

# some headers
-H "Upgrade-Insecure-Requests: 1"
-H "Accept-Language: en-US,en;q=0.8"

# follow redirects
--location
```

It's just a text file, where each line represents one option that will be passed to `curl`. It's read automatically from `~/.curlrc`, so you don't need any extra flags, but you can use `-K` to override or specify different location, e.g.:

```

curl -K .curlrc https://google.com
```

Similarly to flags and options, one might also want to pass in credentials. This can be done with `--user` option, but that will leave the credentials in shell history, so instead we can utilize `.netrc` file which `curl` supports:

```

# ~/.netrc
machine https://authenticationtest.com/HTTPAuth/
login user
password pass
```

The format includes `machine` (the URL), `login` and `password` - they can be on single line or as shown above, and we can also have multiple of them in single file. To use it, just pass it to `curl` like:

```

curl --netrc-file .netrc https://authenticationtest.com/HTTPAuth/
```

## Parallel Requests


We've already talked about ranges of request in globbing section, but what about parallelization? Well, `curl` can do that too:

```

curl -I --parallel --parallel-immediate --parallel-max 3 --config websites.txt

curl -I --parallel --parallel-immediate --parallel-max 3 stackoverflow.com google.com example.com
```

All we need to do, is add `--parallel` (or `-Z`) and `curl` will open up to 50 parallel connections (can be changed with `--parallel-max N`). Also notice how we're supplying the URLs - first option is `--config` and a textfile that would look like:

```

url = "stackoverflow.com"
url = "google.com"
url = "example.com"
```

Other option is just putting all the URL on commandline. Both of these options also works with non-parallel request!

## Formatting & Variables

`curl` can output a lot of stuff, and sometimes it can be overwhelming, verbose and unnecessary. Luckily, we can use output formatting to only print the things that we're interested in:

```

curl --silent --output /dev/null --show-error -w @format.txt http://example.com/

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

We do so using the `-w` option and by passing in a format file. To produce output such as above, we can use:

```

# format.txt
Type: %{content_type}\nCode: %{response_code}\n\n

From 8.1.0:\n\n

Scheme: %{url.scheme}\n
Host: %{url.host}\n
Port: %{url.port}\n

Read header content (v7.83.0):\n
%header{date}
```

Each variable is enclosed in `%{...}`. They can be either simple ones like `response_code` or part of `url.<NAME>` which refer to URL components, such as host or port. Finally, we can also output response headers with `%header{HEADER_NAME}` variables.

One of the many nice use cases for formatting is measuring request/response time, which can be done with following format:

```

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

For full list of variables see [docs](https://everything.curl.dev/usingcurl/verbose/writeout.html#available---write-out-variables).

## Testing & Troubleshooting


Most common way to use `curl` is for (network) troubleshooting - oftentimes just making a request to particular URL will provide enough information, but there are more things we can do, for example we can force usage of specific local network interface:

```

ip link show
# ...
# 3: wlp5s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000

curl --interface wlp5s0 https://example.com
```

Similarly, specific DNS server can be forced:

```

curl --dns-ipv4-addr 1.1.1.1 https://example.com
```

Or we can test for timeout and capture exit code ([exit codes](https://everything.curl.dev/cmdline/exitcode.html)):

```

curl --connect-timeout 30 --silent --output /dev/null \
  --show-error -w 'Total: %{time_total}s\n' http://google.com/ || EXIT_CODE=$?

if [ $EXIT_CODE = 28 ]
then
  echo "Cannot connect (Timeout)."
else
  echo "Can connect."
fi
```

This can be useful - for example - for testing if proxy server is working (with `-x http://proxy.example.com:80`).

Trurl
-----

`curl` is more than just the CLI tool - the project includes `libcurl` as well as `trurl` which I want showcase here.

[`trurl`](https://curl.se/trurl/) is a dedicated tool for parsing URLs and is a sibling project to `curl`. It can be installed from source:

```

sudo apt-get install libcurl4-openssl-dev
git clone https://github.com/curl/trurl.git
cd trurl
make
# ...
```

And here are couple examples for using it:

```

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

First one shows how we can extract URL component, here it's path, but can be e.g. url, scheme, user, password, options or host.

The second one uses the _append_ feature, to add query parameter to the URL.

And final example shows the `--json` option which outputs the parsed URL as JSON, which is great for further processing.

There's lot more `trurl` can do - you can check out [this video](https://www.youtube.com/watch?v=oDL7DVszr2w) or [manual](https://curl.se/trurl/manual.html) (examples at the bottom).

## Sending/Uploading Data


Most of the time we use `curl` to download or request data, but it can (obviously) also send data. POSTing data with `curl` is nothing new right?

```

curl -X POST "https://httpbin.org/post" -H "accept: application/json" --json '{"key": "value"}'
```

But sending JSON like this, having to alternate the single and double quotes gets annoying pretty quickly, but there's a better way:

```

jo -p key=value | curl -X POST "https://httpbin.org/post" -H "accept: application/json" --json @-
```

I think we're all familiar with parsing JSON output of `curl` with `jq`, but what about the opposite? - Above we use `jo` tool which easily creates JSON which we can then pass to `curl` using the `--json` option.

Naturally, the `--json` option can also take input from file, e.g. with `--json @data.json`.

Protocols
---------

Last but not least here are protocols - usually we would only use HTTP or HTTPS, but `curl` supports [_a lot_ of protocols](https://everything.curl.dev/protocols/protocols.html#what-other-protocols-are-there).

One I want to mention in particular is `telnet`, which is handy for testing if server listens on specific port, but what if you're on a server/machine where you don't have and can't install `telnet`? Simply use `curl`:

```

# Same as telnet example.com 1234
curl telnet://example.com:1234
```

Some of the more obscure (funny) protocol options are IMAP, POP3 and SMTP for emails, which means that you _can_ read and send emails with `curl`. To read them:

```

curl --url "imaps://imap.gmail.com:993/Inbox;UID=1" --user "user@gmail.com:PASSWORD"
```

To make this work with GMail specifically, you would need to create [app password](https://support.google.com/mail/answer/185833?hl=en) which is less secure than normal password. If you _actually_ want to try this, do check out also [Gmail IMAP docs](https://developers.google.com/gmail/imap/imap-extensions) and [these queries](https://gist.github.com/akpoff/53ac391037ae2f2d376214eac4a23634) for inspiration.

For sending email, you can use:

```

curl smtp://mail.example.com \
  --mail-from me@example.com \
  --mail-rcpt someone@example.com \
  --upload-file message.txt \
  -u "me@example.com:PASSWORD"
```

Here the `message.txt` is the actual email, which needs to follow a specific format, check out [this page](https://everything.curl.dev/usingcurl/smtp.html) for examples.

## Conclusion


We're at the end, and I'm pretty sure that was at least 10 examples (I stopped counting). But honestly, this really barely scratches the surface - we haven't even touched the `libcurl` which is the _big_ part of `curl`.

There's so much more you can do with `curl` so I recommend exploring both [docs](https://curl.se/docs/manpage.html) and [https://everything.curl.dev/](https://everything.curl.dev/).