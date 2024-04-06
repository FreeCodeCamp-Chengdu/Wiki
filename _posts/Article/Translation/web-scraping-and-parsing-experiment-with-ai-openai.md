---
title: Web scraping experiment with AI (Parsing HTML with GPT-4)
author: ""
authorURL: https://serpapi.com/blog/author/hilman/
originalURL: https://serpapi.com/blog/web-scraping-and-parsing-experiment-with-ai-openai/
translator: ""
reviewer: ""
---

从网络搜索结果中解析数据往往是一件麻烦事。但如果有一种方法能让这一艰苦的过程变得轻而易举呢？让我们尝试一下 OpenAI 的新人工智能模型吧。

#### [Hilman Ramadhan](/blog/author/hilman/)

2023 年 11 月 10 日 10 分钟阅读时间

我一直很惊讶 OpenAI 的 chatGPT 在回答问题方面的表现，以及 Dall-e 3 制作精美图片的能力。现在，有了新的模型，让我们看看人工智能如何处理我们的网络搜索任务，特别是解析搜索引擎结果。我们都知道，从原始 HTML 中提取解析数据通常会很麻烦。但是，如果有一种方法可以将这一艰苦的过程变得轻而易举呢？

> 最近（2023 年 11 月），OpenAI 团队召开了 [首次开发者大会：DevDay（欢迎先观看）](https://www.youtube.com/watch?v=U9mJuUkhUzk)。其中一个令人兴奋的公告是 GPT-4 的更大背景。新的 GPT-4 Turbo 型号功能更强、更便宜，而且支持 128K 上下文窗口。

![](https://serpapi.com/blog/content/images/2023/11/web-scraping-search-results-with-AI-1.webp)

封面插图：OpenAI 的 AI 网络爬虫。

## 我们的小实验

在过去，我们[比较了一些开源和付费 LLM 将纯文本数据转换成简单格式的能力](https://serpapi.com/blog/llms-vs-serpapi/)，并开发了一个[人工智能驱动的解析器](https://serpapi.com/blog/real-world-example-of-ai-powered-parsing/)。

这一次，我们将提升挑战的难度。

-   直接从原始 HTML 数据中抓取。
-   转换成我们需要的特定 JSON 格式。
-   只用很少的开发时间。

**我们的目标:**

-   抓取一个结构良好的网站（作为热身）。
-   从 Google 搜索结果页面返回自然搜索结果（organic results）。
-   从谷歌 SERP 返回 "人们还问（相关问题）"部分。
-   从 Google MAPS 返回当地搜索结果。

> 请记住，人工智能的任务只是解析原始 HTML 数据，而不是自己进行 `网页抓取`。

## TLDR(简而言之)

如果您不想阅读整篇文章，下面是我们使用 OpenAI API（新 GPT-4）模型进行网页抓取实验的利弊总结：

**优点**

-   新模型 `gpt-4-1106-preview` 能够完美地抓取原始 `HTML` 数据。更大的令牌窗口使得只需传递原始 `HTML` 数据即可进行抓取。
-   `OpenAI` 的 `函数调用` 可以准确返回我们需要的响应格式。
-   OpenAI 的 `多函数调用` 可以从多个数据点返回数据。
-   与手动解析所需的开发时间相比，能够抓取原始 HTML 绝对是一个巨大优势。

**缺点**

-   与使用其他 SERP API 提供商相比，成本很高。
-   在传递整个原始 HTML 时要注意成本。我们仍然需要进行修剪 HTML，以便只抓取相关部分。否则，你必须为使用 token（口令）支付高额费用。.
-   将其用于生产时，速度太慢。
-   对于通常在脚本标签、额外的 AJAX 请求或执行操作（如点击、滚动）时发现的 "隐藏数据"，我们仍然需要手动操作。

## 工具和准备

-   由于我们要使用 OpenAI 的 API，因此请务必先注册并获得您的 api_key。您可能还需要 OpenAI 组织 ID。
-   我在这个实验中使用的是 Python，但你也可以随意使用任何编程语言。
-   由于我们希望返回统一的 JSON 格式，因此我们将使用 [OpenAI 的函数调用功能](https://platform.openai.com/docs/guides/function-calling)，在这里我们可以用顺眼的格式定义响应的键和值。
-   我们将使用以下模型 `gpt-4-1106-preview .`

**基础代码**

确保先安装 [OpenAI 库](https://github.com/openai/openai-python)。由于我使用的是 Python，我需要

```shell
pip install openai
```

我还将安装 `requests` 包，以获取原始 HTML 代码

```shell
pip install requests
```

我们的代码库如下所示

```python
import json
import requests
from openai import OpenAI

client = OpenAI(
  organization='YOUR-OPENAI-ORG-ID',
  api_key='YOUR-OPENAI-API-KEY'
)

targetUrl = 'https://books.toscrape.com/' # Target URL will always changes
response = requests.get(targetUrl)
html_text = response.text
```
注：国内用户可能要设置 `base_url`，来使用代理或者第三方 API。

## 第 1 级：使用人工智能对漂亮/简单的结构化网页进行抓取

让我们先热热身。我们首先针对 [https://books.toscrape.com/](https://books.toscrape.com/)网站，因为它的结构非常简洁，便于阅读。

![](https://serpapi.com/blog/content/images/2023/11/web-scraping-target-from-toscrape-books.webp)

截图 toscrape books，是第一个网络搜索目标

下面是我们的代码（下面有解释）

```python
# 来自 OpenAI 的 ChatCompletion API
completion = client.chat.completions.create(
  model="gpt-4-1106-preview", # 请将模型改为 gpt-3.5-turbo-1106
  messages=[
    {"role": "system", "content": "You are a master at scraping and parsing raw HTML."},
    {"role": "user", "content": html_text}
  ],
  tools=[
          {
            "type": "function",
            "function": {
              "name": "parse_data",
              "description": "Parse raw HTML data nicely",
              "parameters": {
                'type': 'object',
                'properties': {
                    'data': {
                        'type': 'array',
                        'items': {
                            'type': 'object',
                            'properties': {
                                'title': {'type': 'string'},
                                'rating': {'type': 'number'},
                                'price': {'type': 'number'}
                            }
                        }
                    }
                }
              }
          }
        }
    ],
   tool_choice={
       "type": "function",
       "function": {"name": "parse_data"}
   }
)

# 数据结果的调用
argument_str = completion.choices[0].message.tool_calls[0].function.arguments
argument_dict = json.loads(argument_str)
data = argument_dict['data']

# 打印格式化
for book in data:
    print(book['title'], book['rating'], book['price'])
```

-   我们使用 OpenAI 的 `ChatCompletion` API
-   使用模型: gpt-4-1106-preview
-   使用提示语 `您是抓取和解析原始 HTML 的高手`，并传递要分析的 `raw_html`。
-   在 `tools` 参数中，我们定义了用于解析原始数据的虚函数（imaginary function）。不要忘记调整参数的属性，以准确返回您想要的格式。

**结果如下**  
我们可以抓取每本书的标题、评分和价格 _（正是我们在上述__函数参数中定义的数据）_。

**运行完成时间**: ~15s

![](https://serpapi.com/blog/content/images/2023/11/Compare-web-scraping-results.webp)

比较网络抓取结果

**使用 gpt-3.5**  
当切换到 `gpt-3.5-turbo-1106` 时，我必须调整提示词，使其更加具体：

```json
messages: {"role": "system", "content": "You are a master at scraping and parsing raw HTML. Scrape ALL the book data results"},

#  function 描述
"function": {
              "name": "parse_data",
              "description": "Get all books data from raw HTML data",
}
```

如果不提及 "抓取所有图书数据"，就只能得到前几个结果。

**运行完成时间**: ~9s

## 第 2 层：利用人工智能解析 Google SERP 中的自然搜索结果（Organic results）

The Google search results page is not like the previous site. It has a more complicated structure, unclear CSS class names, and includes many unknown data in the raw HTML.

Target URL: 'https://www.google.com/search?q=coffee&gl=us'

> WARNING! At first, I just parse everything from Google raw HTML, it turns out it contains too many characters, which means more tokens and more costs!

![](https://serpapi.com/blog/content/images/2023/11/openai-usage-billing.webp)

Watchout your OpenAI billing usage

So, after few tries, I decided to trim only the body part and remove the `style` and `script` tag contents.

I've adjusted the prompt and function parameters like this:

```python
import re #additional import for regex

response = requests.get('https://www.google.com/search?q=coffee&gl=us')
html_text = response.text

# Remove unnecessary part to prevent HUGE TOKEN cost!
# Remove everything between <head> and </head>
html_text = re.sub(r'<head.*?>.*?</head>', '', html_text, flags=re.DOTALL)
# Remove all occurrences of content between <script> and </script>
html_text = re.sub(r'<script.*?>.*?</script>', '', html_text, flags=re.DOTALL)
# Remove all occurrences of content between <style> and </style>
html_text = re.sub(r'<style.*?>.*?</style>', '', html_text, flags=re.DOTALL)

completion = client.chat.completions.create(
  model="gpt-4-1106-preview",
  messages=[
    {"role": "system", "content": "You are a master at scraping Google results data. Scrape top 10 organic results data from Google search result page."},
    {"role": "user", "content": html_text}
  ],
  tools=[
          {
          "type": "function",
          "function": {
            "name": "parse_data",
            "description": "Parse organic results from Google SERP raw HTML data nicely",
            "parameters": {
              'type': 'object',
              'properties': {
                  'data': {
                      'type': 'array',
                      'items': {
                          'type': 'object',
                          'properties': {
                              'title': {'type': 'string'},
                              'original_url': {'type': 'string'},
                              'snippet': {'type': 'string'},
                              'position': {'type': 'integer'}
                          }
                      }
                  }
              }
            }
          }
        }
    ],
   tool_choice={
       "type": "function",
       "function": {"name": "parse_data"}
   }
)

argument_str = completion.choices[0].message.tool_calls[0].function.arguments
argument_dict = json.loads(argument_str)
data = argument_dict['data']

for result in data:
    print(result['title'])
    print(result['original_url'] or '')
    print(result['snippet']  or '')
    print(result['position'])
    print('---')
```

-   First, we trim only the selected part.
-   Adjust the prompt to "You are a master at scraping Google results data. Scrape top 10 organic results data from Google search result page."
-   Adjust the function parameters to any format you need.

![](https://serpapi.com/blog/content/images/2023/11/basic-google-SERP-scraping-with-AI.webp)

basic web scraping on Google SERP with AI

Ta-da! we get exactly the data we need despite of the complicated format from Google raw HTML.

**Time to finish**: ~28s

> Notes: My original prompt was "Parse organic results from Google SERP raw HTML data nicely" It only returns the first 3-5 results, so I adjust the prompt to get more preices amount of results.

**Using gpt-3.5 model**  
I'm not able to do this since the raw HTML data amount exceeds the token length.

## Level 3: Parse local place results from Google Maps with AI

Now, let's scrape another Google product, which is Google Maps. This is our target page: [https://www.google.com/maps/search/coffee/@40.7455096,-74.0083012,14z?hl=en&entry=ttu](https://www.google.com/maps/search/coffee/@40.7455096,-74.0083012,14z?hl=en&entry=ttu)

![](https://serpapi.com/blog/content/images/2023/11/Google-maps-target-page.webp)

Google Maps screenshot

As you can see, each of the items includes many information. We'll scrape:  
\- Name  
\- Rating average  
\- Total rating  
\- Price  
\- Address  
\- Extras  
\- Hours  
\- Additional service  
\- Thumbnail image

> **Warning!** It turns out, Google Maps load this data via Javascript, so I have to change my method to get the raw_html from using `requests` to `selenium` for python.

**Code**

Install Selenium on Python. More instructions on installation are [here](https://selenium-python.readthedocs.io/installation.html#introduction).

```
pip install selenium
```

Import Selenium

```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
```

Create a headless browser instance to surf the web

```python
target_url = 'https://www.google.com/maps/search/coffee/@40.7455096,-74.0083012,14z?hl=en'
op = webdriver.ChromeOptions()
op.add_argument('headless')
driver = webdriver.Chrome(options=op)
driver.get(target_url)

driver.implicitly_wait(1) # seconds

# get raw html
html_text = driver.page_source

# You can continue like previous method, where we trim only the body part first
```

I wait 1 second with `implicitly_wait` to make sure the data is already there to scrape.

Now, here's the OpenAI API function:

```python
completion = client.chat.completions.create(
  model="gpt-4-1106-preview",
  messages=[
    {"role": "system", "content": "You are a master at scraping Google Maps results. Scrape all local places results data"},
    {"role": "user", "content": html_text}
  ],
  tools=[
          {
          "type": "function",
          "function": {
            "name": "parse_data",
            "description": "Parse local results detail from Google MAPS raw HTML data nicely",
            "parameters": {
              'type': 'object',
              'properties': {
                  'data': {
                      'type': 'array',
                      'items': {
                          'type': 'object',
                          'properties': {
                              'position': {'type': 'integer'},
                              'title': {'type': 'string'},
                              'rating': {'type': 'string'},
                              'total_reviews': {'type': 'string'},
                              'price': {'type': 'string'},
                              'type': {'type': 'string'},
                              'address': {'type': 'string'},
                              'phone': {'type': 'string'},
                              'hours': {'type': 'string'},
                              'service_options': {'type': 'string'},
                              'image_url': {'type': 'string'},
                          }
                      }
                  }
              }
            }
          }
        }
    ],
   tool_choice={
       "type": "function",
       "function": {"name": "parse_data"}
   }
)


argument_str = completion.choices[0].message.tool_calls[0].function.arguments
argument_dict = json.loads(argument_str)
data = argument_dict['data']

print(data)
```

**Result:**

![](https://serpapi.com/blog/content/images/2023/11/local-maps-AI-scraping-results.webp)

local maps API scraping with AI results

It turns out perfect! I'm able to get the exact data for each of this local_results.

**Time to finish with Selenium**: ~47s  
**Time to finish (exclude Selenium time)**: ~34s

## Level 4: Parsing two different data (organic results and people-also-ask section) from Google SERP with AI

As you might know, Google SERP doesn't just display organic results, but also other data like ads, people-also-ask (related questions), knowledge graphs, and so on.

_Let's see how to_ _target multiple data points with multiple functions calling features from OpenAI._

Here's the code

```python
completion = client.chat.completions.create(
  model="gpt-4-1106-preview",
  messages=[
    {"role": "system", "content": "You are a master at scraping Google results data. Scrape two things: 1st. Scrape top 10 organic results data and 2nd. Scrape people_also_ask section from Google search result page."},
    {"role": "user", "content": html_text}
  ],
  tools=[
          {
          "type": "function",
          "function": {
            "name": "parse_organic_results",
            "description": "Parse organic results from Google SERP raw HTML data nicely",
            "parameters": {
              'type': 'object',
              'properties': {
                  'data': {
                      'type': 'array',
                      'items': {
                          'type': 'object',
                          'properties': {
                              'title': {'type': 'string'},
                              'original_url': {'type': 'string'},
                              'snippet': {'type': 'string'},
                              'position': {'type': 'integer'}
                          }
                      }
                  }
              }
            }
          }
        },
          {
          "type": "function",
          "function": {
            "name": "parse_people_also_ask_section",
            "description": "Parse `people also ask` section from Google SERP raw HTML",
            "parameters": {
              'type': 'object',
              'properties': {
                  'data': {
                      'type': 'array',
                      'items': {
                          'type': 'object',
                          'properties': {
                              'question': {'type': 'string'},
                              'original_url': {'type': 'string'},
                              'answer': {'type': 'string'},
                          }
                      }
                  }
              }
            }
          }
        }
    ],
    tool_choice="auto"
)


# Organic_results
argument_str = completion.choices[0].message.tool_calls[0].function.arguments
argument_dict = json.loads(argument_str)
organic_results = argument_dict['data']

print('Organic results:')

for result in organic_results:
    print(result['title'])
    print(result['original_url'] or '')
    print(result['snippet']  or '')
    print(result['position'])
    print('---')

# People also ask
argument_str = completion.choices[0].message.tool_calls[1].function.arguments
argument_dict = json.loads(argument_str)
people_also_ask = argument_dict['data']

print('People also ask:')
for result in people_also_ask:
    print(result['question'])
    print(result['original_url'] or '')
    print(result['answer']  or '')
    print('---')
```

**Code Explanation:**

-   Adjust the prompt to include specific information on what to scrape: "You are a master at scraping Google results data. Scrape two things: 1st. Scrape top 10 organic results data and 2nd. Scrape the _the_ people_also_ask section from the Google search result page."
-   Adding and separating functions, one for organic results*,* and one for people-also-ask section.
-   Testing the output in two different formats.

Here's the result:

![](https://serpapi.com/blog/content/images/2023/11/multiple-data-points-on-AI-scraping.webp)

Scraping multiple data points with AI

**Success:**  
I'm able to scrape both the organic_results and people_also_ask separately. Kudos to OpenAI!

**Problem:**  
I'm not able to scrape the answer and original URL for the people*also_ask section. \_The reason is* that this information is hidden somewhere in the script tag. We could try this by providing that specific part of the script content, but I would consider it `cheating` for this experiment, as we want to pass the raw HTML without pinpointing or giving a hint.

**Time to finish**: ~30s

If you want to learn how to scrape these data cheaper, faster, and more accurate. You can read these posts:

-   [Scrape Google search results using Python](https://serpapi.com/blog/how-to-scrape-google-search-results-with-python/)
-   [Scrape Google Maps places results using Python](https://serpapi.com/blog/using-google-maps-place-results-api-from-serpapi/)

## Table comparison with SerpApi

Here's the timetable comparison using OpenAI's new GPT-4 model for web scraping VS [SerpApi](https://serpapi.com/?ref=web-scraping-with-ai-experiment-blog) . We're comparing with the 'normal speed'; SerpApi has faster (roughly twice as fast) when using [Ludicrous Speed](https://serpapi.com/ludicrous-speed).

| Subject                                | gpt-4-1106-preview | SerpApi |
| -------------------------------------- | ------------------ | ------- |
| Organic results                        | 15s                | 2.4s    |
| Organic results with Related questions | 30s                | 2.4s    |
| Maps local results                     | 47s                | 2.7s    |

## Conclusion

OpenAI definitely improves a lot of over time. Now, it's possible to scrape a website and collect relevant data with the API. But based on the time it takes, it's not yet ready for production, commercial purpose, or scale. While the accuracy of the data and response format turns out perfect, it's still far behind in terms of cost and speed.

Let me know if you have any thoughts, spot any mistakes, and other experiments you want to add that are relevant to this post to hilman(at)serpapi(dot)com.

Thank you for reading this post!
