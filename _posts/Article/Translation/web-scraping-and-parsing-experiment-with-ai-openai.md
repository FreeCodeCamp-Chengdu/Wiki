---
title: Web scraping experiment with AI (Parsing HTML with GPT-4)
author: ""
authorURL: https://serpapi.com/blog/author/hilman/
originalURL: https://serpapi.com/blog/web-scraping-and-parsing-experiment-with-ai-openai/
translator: ""
reviewer: ""
---

[Artificial Intelligence](https://serpapi.com/blog/tag/artificial-intelligence/)

# Web scraping experiment with AI (Parsing HTML with GPT-4)

Parsing data from web scraping results can often be cumbersome. But what if there's a way to turn this painstaking process into a breeze? Let's experiment with new AI model by OpenAI

-   [![Hilman Ramadhan](/blog/content/images/size/w100/2023/10/profile-picture.png)](/blog/author/hilman/)

#### [Hilman Ramadhan](/blog/author/hilman/)

Nov 10, 2023 • 10 min read

I've always been surprised at how good chatGPT by OpenAI is on answering questions and the ability of Dall-e 3 to create stunning images. Now, with the new model, let's see how AI can handle our web scraping tasks, specifically on parsing search engine results. We all know the drill—parsing data from raw HTML can often be cumbersome. _But what if there's a way to turn this painstaking process into a breeze?_

> Recently (November 2023), the OpenAI team had their [first developer conference: DevDay (Feel free to watch it first)](https://www.youtube.com/watch?v=U9mJuUkhUzk). One of the exciting announcements is a larger context for GPT-4. The new GPT-4 Turbo model is more capable, cheaper, and supports a 128K context window.

![](https://serpapi.com/blog/content/images/2023/11/web-scraping-search-results-with-AI-1.webp)

Cover illustration for web scraping with AI from OpenAI.

## Our little experiment

In the past, we've [compared some open-source and paid LLMs' ability to scrape](https://serpapi.com/blog/llms-vs-serpapi/) "clean text" data into a simple format and developed an [AI powered parser](https://serpapi.com/blog/real-world-example-of-ai-powered-parsing/).

This time, we'll level up the challenge.

-   Scrape directly from raw HTML data.
-   Turn into a specific JSON format that we need.
-   Use little development time.

**Our Target:**

-   Scrape a nice structured website (as a warm-up).
-   Return organic results from the Google search results page.
-   Return the people-also-ask (related questions) section from Google SERP.
-   Return local results data from Google MAPS.

> Remember that the AI is only tasked with parsing the raw HTML data, not doing the `web scraping` itself.

## TLDR;

If you don't want to read the whole post, here is the summary of the pros and cons of our experiment using the OpenAI API (new GPT-4) model for web scraping:

**Pros**

-   New model `gpt-4-1106-preview` is able to scrape raw HTML data perfectly. The larger token window makes it possible just to pass raw HTML to scrape.
-   OpenAI "function calling" can return the exact response format that we need.
-   OpenAI "multiple function calling" can return data from multiple data points.
-   The ability to scrape raw HTML definitely a huge plus compared to the development time when parsing manually.

**Cons**

-   The cost is still high compared to using other SERP API providers.
-   Watch out for the cost when passing the whole raw HTML. We still need to trim to scrape only relevant parts. Otherwise, you have to pay a lot for the token usage.
-   The speed is still too long to use it for production.
-   For "hidden data" that is normally found at the script tag, extra AJAX request, or upon performing an action (e.g., clicking, scrolling), we still need to do it manually.

## Tools and Preparation

-   Since we're going to use OpenAI's API, make sure to register and have your api\_key first. You might need your OpenAI org ID as well.
-   I'm using Python for this experiment, but feel free to use any programming language you want.
-   Since we want to return a consistent JSON format, we'll be using [new function calling feature from OpenAI](https://platform.openai.com/docs/guides/function-calling), where we can define the response's keys and values with a nice format.
-   We're going to use this model `gpt-4-1106-preview .`

**Basic Code**

Make sure to install the [OpenAI library first](https://github.com/openai/openai-python). Since I'm using Python, I need

```
pip install openai
```

I'll also install `requests` package to get the raw HTML

```
pip install requests
```

  
Here's what our code base will look like

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

## Level 1: Scraping on nice/simple structured web page with AI

Let's warm up first. We'll target the [https://books.toscrape.com/](https://books.toscrape.com/) site first since it has a very clean structure that makes it easy to read.

![](https://serpapi.com/blog/content/images/2023/11/web-scraping-target-from-toscrape-books.webp)

Screenshot toscrape books, first web scraping targets

Here's what our code looks like (with explanations below)

```python
# Chat Completion API from OpenAI
completion = client.chat.completions.create(
  model="gpt-4-1106-preview", # Feel free to change the model to gpt-3.5-turbo-1106
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

# Calling the data results
argument_str = completion.choices[0].message.tool_calls[0].function.arguments
argument_dict = json.loads(argument_str)
data = argument_dict['data']

# Print in a nice format
for book in data:
    print(book['title'], book['rating'], book['price'])
```

-   We're using the `ChatCompletion` API from OpenAI
-   Use model: gpt-4-1106-preview
-   Using the prompt "You are a master at scraping and parsing raw HTML" and passing the raw\_html to be analyzed.
-   At `tools` parameter, we're defining our imaginary function to parse the raw data. Don't forget to adjust the properties of parameters to return the exact format you want.

**Here i**s **the result**  
We're able to scrape the title, rating, and price _(exactly the data we defined in the_ _function parameters above)_ of each book.

**Time to finish**: ~15s

![](https://serpapi.com/blog/content/images/2023/11/Compare-web-scraping-results.webp)

Compare web scraping results

**Using gpt-3.5**  
When switching to `gpt-3.5-turbo-1106` , I have to adjust the prompt to be more specific:

```
messages: {"role": "system", "content": "You are a master at scraping and parsing raw HTML. Scrape ALL the book data results"},

# And the function description
"function": {
              "name": "parse_data",
              "description": "Get all books data from raw HTML data",
}
```

Without mentioning "scrape ALL book data," it will just get the first few results.

**Time to finish**: ~9s

## Level 2: Parse organic results from Google SERP with AI

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

> **Warning!** It turns out, Google Maps load this data via Javascript, so I have to change my method to get the raw\_html from using `requests` to `selenium` for python.

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

It turns out perfect! I'm able to get the exact data for each of this local\_results.

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

-   Adjust the prompt to include specific information on what to scrape: "You are a master at scraping Google results data. Scrape two things: 1st. Scrape top 10 organic results data and 2nd. Scrape the _the_ people\_also\_ask section from the Google search result page."
-   Adding and separating functions, one for organic results_,_ and one for people-also-ask section.
-   Testing the output in two different formats.

Here's the result:

![](https://serpapi.com/blog/content/images/2023/11/multiple-data-points-on-AI-scraping.webp)

Scraping multiple data points with AI

**Success:**  
I'm able to scrape both the organic\_results and people\_also\_ask separately. Kudos to OpenAI!

**Problem:**  
I'm not able to scrape the answer and original URL for the people\_also\_ask section. _The reason is_ that this information is hidden somewhere in the script tag. We could try this by providing that specific part of the script content, but I would consider it `cheating` for this experiment, as we want to pass the raw HTML without pinpointing or giving a hint.

**Time to finish**: ~30s

If you want to learn how to scrape these data cheaper, faster, and more accurate. You can read these posts:

-   [Scrape Google search results using Python](https://serpapi.com/blog/how-to-scrape-google-search-results-with-python/)
-   [Scrape Google Maps places results using Python](https://serpapi.com/blog/using-google-maps-place-results-api-from-serpapi/)

## Table comparison with SerpApi

Here's the timetable comparison using OpenAI's new GPT-4 model for web scraping VS [SerpApi](https://serpapi.com/?ref=web-scraping-with-ai-experiment-blog) . We're comparing with the 'normal speed'; SerpApi has faster (roughly twice as fast) when using [Ludicrous Speed](https://serpapi.com/ludicrous-speed).

| Subject | gpt-4-1106-preview | SerpApi |
| --- | --- | --- |
| Organic results | 15s | 2.4s |
| Organic results with Related questions | 30s | 2.4s |
| Maps local results | 47s | 2.7s |

## Conclusion

OpenAI definitely improves a lot of over time. Now, it's possible to scrape a website and collect relevant data with the API. But based on the time it takes, it's not yet ready for production, commercial purpose, or scale. While the accuracy of the data and response format turns out perfect, it's still far behind in terms of cost and speed.

Let me know if you have any thoughts, spot any mistakes, and other experiments you want to add that are relevant to this post to hilman(at)serpapi(dot)com.

Thank you for reading this post!