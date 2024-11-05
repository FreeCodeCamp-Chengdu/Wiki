---
title: Embeddings are underrated#
date: 2024-11-05T08:32:45.430Z
authorURL: ""
originalURL: https://technicalwriting.dev/data/embeddings.html
translator: ""
reviewer: ""
---

# Embeddings are underrated[#][1]

<!-- more -->

Machine learning (ML) has the potential to advance the state of the art in technical writing. No, I’m not talking about text generation models like Claude, Gemini, LLaMa, GPT, etc. The ML technology that might end up having the biggest impact on technical writing is **embeddings**.

Embeddings aren't exactly new, but they have become much more widely accessible in the last couple years. What embeddings offer to technical writers is **_the ability to discover connections between texts at previously impossible scales_**.

## Building intuition about embeddings[#][2]

Here’s an overview of how you use embeddings and how they work. It’s geared towards technical writers who are learning about embeddings for the first time.

### Input and output[#][3]

Someone asks you to “make some embeddings”. What do you input? You input text.1 You don’t need to provide the same amount of text every time. E.g. sometimes your input is a single paragraph while at other times it’s a few sections, an entire document, or even multiple documents.

What do you get back? If you provide a single word as the input, the output will be an array of numbers like this:

\[-0.02387, -0.0353, 0.0456\]

Now suppose your input is an entire set of documents. The output turns into this:

\[0.0451, -0.0154, 0.0020\]

One input was drastically smaller than the other, yet they both produced an array of 3 numbers. Curiouser and curiouser. (When you work with real embeddings, the arrays will have hundreds or thousands of numbers, not 3. More on that later.)

Here’s the first key insight. Because we always get back the same amount of numbers no matter how big or small the input text, **we now have a way to mathematically compare any two pieces of arbitrary text to each other**.

But what do those numbers _MEAN_?

1 Some embedding models are “multimodal”, meaning you can also provide images, videos, and audio as input. This post focuses on text since that’s the medium that we work with the most as technical writers. Haven’t seen a multimodal model support taste, touch, or smell yet!

### First, how to literally make the embeddings[#][4]

The big service providers have made it easy. Here’s how it’s done with Gemini:

import google.generativeai as gemini

gemini.configure(api\_key\='…')

text \= 'Hello, world!'
response \= gemini.embed\_content(
    model\='models/text-embedding-004',
    content\=text,
    task\_type\='SEMANTIC\_SIMILARITY'
)
embedding \= response\['embedding'\]

The size of the array depends on what model you’re using. Gemini’s [text-embedding-004][5] model returns an array of 768 numbers whereas Voyage AI’s [voyage-3][6] model returns an array of 1024 numbers. This is one of the reasons why you can’t use embeddings from different providers interchangeably. (The main reason is that the numbers from one model mean something completely different than the numbers from another model.)

#### Does it cost a lot of money?[#][7]

No.

#### Is it terrible for the environment?[#][8]

I don’t know. After the model has been created (trained), I’m pretty sure that generating embeddings is much less computationally intensive than generating text. But it also seems to be the case that embedding models are trained in similar ways as text generation models2, with all the energy usage that implies. I’ll update this section when I find out more.

2 From [You Should Probably Pay Attention to Tokenizers][9]: “Embeddings are byproduct of transformer training and are actually trained on the heaps of tokenized texts. It gets better: embeddings are what is actually fed as the input to LLMs when we ask it to generate text.”

#### What model is best?[#][10]

Ideally, your embedding model can accept a huge amount of input text, so that you can generate embeddings for complete pages. If you try to provide more input than a model can handle, you usually get an error. As of October 2024 `voyage-3` seems to the clear winner in terms of input size3:

| 
Organization

 | 

Model name

 | 

Input limit ([tokens][11])

 |
| --- | --- | --- |
| 

Voyage AI

 | 

[voyage-3][12]

 | 

32000

 |
| 

Nomic

 | 

[Embed][13]

 | 

8192

 |
| 

OpenAI

 | 

[text-embedding-3-large][14]

 | 

81914

 |
| 

Mistral

 | 

[Embed][15]

 | 

8000

 |
| 

Google

 | 

[text-embedding-004][16]

 | 

2048

 |
| 

Cohere

 | 

[embed-english-v3.0][17]

 | 

512

 |

For my particular use cases as a technical writer, large input size is an important factor. However, your use cases may not need large input size, or there may be other factors that are more important. See the [Massive Text Embedding Benchmark][18] (MTEB) [leaderboard][19].

3 These input limits are based on [tokens][20], and each service calculates tokens differently, so don’t put too much weight into these exact numbers. E.g. a token for one model may be approximately 3 characters, whereas for another one it may be approximately 4 characters.

4 Previously, I incorrectly listed this model’s input limit as 3072. Sorry for the mistake.

### Very weird multi-dimensional space[#][21]

Back to the big mystery. What the hell do these numbers **MEAN**?!?!?!

Let’s begin by thinking about **coordinates on a map**. Suppose I give you three points and their coordinates:

| 
Point

 | 

X-Coordinate

 | 

Y-Coordinate

 |
| --- | --- | --- |
| 

A

 | 

3

 | 

2

 |
| 

B

 | 

1

 | 

1

 |
| 

C

 | 

\-2

 | 

\-2

 |

There are 2 dimensions to this map: the X-Coordinate and the Y-Coordinate. Each point lives at the intersection of an X-Coordinate and a Y-Coordinate.

Is A closer to B or C?

![../_images/embeddings-1.png](/_images/embeddings-1.png)

A is much closer to B.

Here’s the mental leap. _Embeddings are similar to points on a map_. Each number in the embedding array is a _dimension_, similar to the X-Coordinates and Y-Coordinates from earlier. When an embedding model sends you back an array of 1000 numbers, it’s telling you the point where that text _semantically_ lives in its 1000-dimension space, relative to all other texts. When we compare the distance between two embeddings in this 1000-dimension space, what we’re really doing is **figuring out how semantically close or far apart those two texts are from each other**.

![../_images/mindblown.gif](/_images/mindblown.gif)

The concept of positioning items in a multi-dimensional space like this, where related items are clustered near each other, goes by the wonderful name of [latent space][22].

The most famous example of the weird utility of this technology comes from the [Word2vec paper][23], the foundational research that kickstarted interest in embeddings 11 years ago. In the paper they shared this anecdote:

embedding("king") - embedding("man") + embedding("woman") ≈ embedding("queen")

Starting with the embedding for `king`, subtract the embedding for `man`, then add the embedding for `woman`. When you look around this vicinity of the latent space, you find the embedding for `queen` nearby. In other words, embeddings can represent semantic relationships in ways that feel intuitive to us humans. If you asked a human “what’s the female equivalent of a king?” that human would probably answer “queen”, the same answer we get from embeddings. For more explanation of the underlying theories, see [Distributional semantics][24].

The 2D map analogy was a nice stepping stone for building intuition but now we need to cast it aside, because embeddings operate in hundreds or thousands of dimensions. It’s impossible for us lowly 3-dimensional creatures to visualize what “distance” looks like in 1000 dimensions. Also, we don’t know what each dimension represents, hence the section heading “Very weird multi-dimensional space”.5 One dimension might represent something close to color. The `king - man + woman ≈ queen` anecdote suggests that these models contain a dimension with some notion of gender. And so on. [Well Dude, we just don’t know][25].

The mechanics of converting text into very weird multi-dimensional space are complex, as you might imagine. They are teaching _machines_ to _LEARN_, after all. [The Illustrated Word2vec][26] is a good way to start your journey down that rabbithole.

5 I borrowed this phrase from [Embeddings: What they are and why they matter][27].

### Comparing embeddings[#][28]

After you’ve generated your embeddings, you’ll need some kind of “database” to keep track of what text each embedding is associated to. In the experiment discussed later, I got by with just a local JSON file:

{
    "authors": {
        "embedding": \[…\]
    },
    "changes/0.1": {
        "embedding": \[…\]
    },
    …
}

`authors` is the name of a page. `embedding` is the embedding for that page.

Comparing embeddings involves a lot of linear algebra. I learned the basics from [Linear Algebra for Machine Learning and Data Science][29]. The big math and ML libraries like [NumPy][30] and [scikit-learn][31] can do the heavy lifting for you (i.e. very little math code on your end).

## Applications[#][32]

I could tell you exactly how I think we might advance the state of the art in technical writing with embeddings, but where’s the fun in that? You now know why they’re such an interesting and useful new tool in the technical writer toolbox… go connect the rest of the dots yourself!

Let’s cover a basic example to put the intuition-building ideas into practice and then wrap up this post.

### Related pages[#][33]

Some docs sites have a recommendation system that makes you aware of other relevant docs. The system looks at whatever page you’re currently on, finds other pages related to this one, and then recommends other pages to visit. Embeddings provide a new way to support this feature, probably at a fraction of the cost of previous methods. Here’s how it works:

1.  Generate an embedding for each page on your docs site.
    
2.  For each page, compare its embedding against all other page embeddings. If the two embeddings are mathematically similar, then the contents on the two pages are probably related to each other.
    

This can be done as a batch operation. A page’s embedding only needs to change when the page’s content changes.

I ran this experiment on the [Sphinx][34] docs. The results were pretty good. [Implementation][35] and [Results][36] have the details.

See [Related content using embeddings][37] for another example of this approach.

### Let a thousand embeddings bloom?[#][38]

As docs site owners, I wonder if we should start freely providing embeddings for our content to anyone who wants them, via REST APIs or [well-known URIs][39]. Who knows what kinds of cool stuff our communities can build with this extra type of data about our docs?

## Parting words[#][40]

Three years ago, if you had asked me what 768-dimensional space is, I would have told you that it’s just some abstract concept that physicists and mathematicians need for unfathomable reasons, probably something related to string theory. Embeddings gave me a reason to think about this idea more deeply, and actually apply it to my own work. I think that’s pretty cool.

Order-of-magnitude improvements in our ability to maintain our docs may very well still be possible after all… perhaps we just need an order-of-magnitude-more dimensions!!

## Appendix[#][41]

### Implementation[#][42]

I created a [Sphinx extension][43] to generate an embedding for each doc. Sphinx automatically invokes this extension as it builds the docs.

import json
import os

import voyageai

VOYAGE\_API\_KEY \= os.getenv('VOYAGE\_API\_KEY')
voyage \= voyageai.Client(api\_key\=VOYAGE\_API\_KEY)

def on\_build\_finished(app, exception):
    with open(srcpath, 'w') as f:
        json.dump(data, f, indent\=4)

def embed\_with\_voyage(text):
    try:
        embedding \= voyage.embed(\[text\], model\='voyage-3', input\_type\='document').embeddings\[0\]
        return embedding
    except Exception as e:
        return None

def on\_doctree\_resolved(app, doctree, docname):
    text \= doctree.astext()
    embedding \= embed\_with\_voyage(text)  \# Generate an embedding for each document!
    data\[docname\] \= {
        'embedding': embedding
    }

\# Use some globals because this is just an experiment and you can't stop me
def init\_globals(srcdir):
    global filename
    global srcpath
    global data
    filename \= 'embeddings.json'
    srcpath \= f'{srcdir}/{filename}'
    data \= {}

def setup(app):
    init\_globals(app.srcdir)
    \# https://www.sphinx-doc.org/en/master/extdev/appapi.html#sphinx-core-events
    app.connect('doctree-resolved', on\_doctree\_resolved)  \# This event fires on every doc that's processed
    app.connect('build-finished', on\_build\_finished)
    return {
        'version': '0.0.1',
        'parallel\_read\_safe': True,
        'parallel\_write\_safe': True,
    }

When the build finishes, the embeddings data is stored in `embeddings.json` like this:

{
    "authors": {
        "embedding": \[…\]
    },
    "changes/0.1": {
        "embedding": \[…\]
    },
    …
}

`authors` and `changes/0.1` are docs. `embedding` contains the embedding for that doc.

The last step is to find the closest neighbor for each doc. I.e. to find the other page that is considered relevant to the page you’re currently on. As mentioned earlier, [Linear Algebra for Machine Learning and Data Science][44] was the class that taught me the basics.

import json

import numpy as np
from sklearn.metrics.pairwise import cosine\_similarity

def find\_docname(data, target):
    for docname in data:
        if data\[docname\]\['embedding'\] \== target:
            return docname
    return None

\# Adapted from the Voyage AI docs
\# https://web.archive.org/web/20240923001107/https://docs.voyageai.com/docs/quickstart-tutorial
def k\_nearest\_neighbors(target, embeddings, k\=5):
    \# Convert to numpy array
    target \= np.array(target)
    embeddings \= np.array(embeddings)
    \# Reshape the query vector embedding to a matrix of shape (1, n) to make it
    \# compatible with cosine\_similarity
    target \= target.reshape(1, \-1)
    \# Calculate the similarity for each item in data
    cosine\_sim \= cosine\_similarity(target, embeddings)
    \# Sort the data by similarity in descending order and take the top k items
    sorted\_indices \= np.argsort(cosine\_sim\[0\])\[::\-1\]
    \# Take the top k related embeddings
    top\_k\_related\_embeddings \= embeddings\[sorted\_indices\[:k\]\]
    top\_k\_related\_embeddings \= \[
        list(row\[:\]) for row in top\_k\_related\_embeddings
    \]  \# convert to list
    return top\_k\_related\_embeddings

with open('doc/embeddings.json', 'r') as f:
    data \= json.load(f)
embeddings \= \[data\[docname\]\['embedding'\] for docname in data\]
print('.. csv-table::')
print('   :header: "Target", "Neighbor"')
print()
for target in embeddings:
    dot\_products \= np.dot(embeddings, target)
    neighbors \= k\_nearest\_neighbors(target, embeddings, k\=3)
    \# ignore neighbors\[0\] because that is always the target itself
    nearest\_neighbor \= neighbors\[1\]
    target\_docname \= find\_docname(data, target)
    target\_cell \= f'\`{target\_docname} <https://www.sphinx-doc.org/en/master/{target\_docname}.html>\`\_'
    neighbor\_docname \= find\_docname(data, nearest\_neighbor)
    neighbor\_cell \= f'\`{neighbor\_docname} <https://www.sphinx-doc.org/en/master/{neighbor\_docname}.html>\`\_'
    print(f'   "{target\_cell}", "{neighbor\_cell}"')

As you may have noticed, I did not actually implement the recommendation UI in this experiment. My main goal was to get basic data on whether the embeddings approach generates decent recommendations or not.

### Results[#][45]

How to interpret the data: `Target` would be the page that you’re currently on. `Neighbor` would be the recommended page.

| 
Target

 | 

Neighbor

 |
| --- | --- |
| 

[authors][46]

 | 

[changes/0.6][47]

 |
| 

[changes/0.1][48]

 | 

[changes/0.5][49]

 |
| 

[changes/0.2][50]

 | 

[changes/1.2][51]

 |
| 

[changes/0.3][52]

 | 

[changes/0.4][53]

 |
| 

[changes/0.4][54]

 | 

[changes/1.2][55]

 |
| 

[changes/0.5][56]

 | 

[changes/0.6][57]

 |
| 

[changes/0.6][58]

 | 

[changes/1.6][59]

 |
| 

[changes/1.0][60]

 | 

[changes/1.3][61]

 |
| 

[changes/1.1][62]

 | 

[changes/1.2][63]

 |
| 

[changes/1.2][64]

 | 

[changes/1.1][65]

 |
| 

[changes/1.3][66]

 | 

[changes/1.4][67]

 |
| 

[changes/1.4][68]

 | 

[changes/1.3][69]

 |
| 

[changes/1.5][70]

 | 

[changes/1.6][71]

 |
| 

[changes/1.6][72]

 | 

[changes/1.5][73]

 |
| 

[changes/1.7][74]

 | 

[changes/1.8][75]

 |
| 

[changes/1.8][76]

 | 

[changes/1.6][77]

 |
| 

[changes/2.0][78]

 | 

[changes/1.8][79]

 |
| 

[changes/2.1][80]

 | 

[changes/1.2][81]

 |
| 

[changes/2.2][82]

 | 

[changes/1.2][83]

 |
| 

[changes/2.3][84]

 | 

[changes/2.1][85]

 |
| 

[changes/2.4][86]

 | 

[changes/3.5][87]

 |
| 

[changes/3.0][88]

 | 

[changes/4.3][89]

 |
| 

[changes/3.1][90]

 | 

[changes/3.3][91]

 |
| 

[changes/3.2][92]

 | 

[changes/3.0][93]

 |
| 

[changes/3.3][94]

 | 

[changes/3.1][95]

 |
| 

[changes/3.4][96]

 | 

[changes/4.3][97]

 |
| 

[changes/3.5][98]

 | 

[changes/1.3][99]

 |
| 

[changes/4.0][100]

 | 

[changes/3.0][101]

 |
| 

[changes/4.1][102]

 | 

[changes/4.4][103]

 |
| 

[changes/4.2][104]

 | 

[changes/4.4][105]

 |
| 

[changes/4.3][106]

 | 

[changes/3.0][107]

 |
| 

[changes/4.4][108]

 | 

[changes/7.4][109]

 |
| 

[changes/4.5][110]

 | 

[changes/4.4][111]

 |
| 

[changes/5.0][112]

 | 

[changes/3.5][113]

 |
| 

[changes/5.1][114]

 | 

[changes/5.0][115]

 |
| 

[changes/5.2][116]

 | 

[changes/3.5][117]

 |
| 

[changes/5.3][118]

 | 

[changes/5.2][119]

 |
| 

[changes/6.0][120]

 | 

[changes/6.2][121]

 |
| 

[changes/6.1][122]

 | 

[changes/6.2][123]

 |
| 

[changes/6.2][124]

 | 

[changes/6.1][125]

 |
| 

[changes/7.0][126]

 | 

[extdev/deprecated][127]

 |
| 

[changes/7.1][128]

 | 

[changes/7.2][129]

 |
| 

[changes/7.2][130]

 | 

[changes/7.4][131]

 |
| 

[changes/7.3][132]

 | 

[changes/7.4][133]

 |
| 

[changes/7.4][134]

 | 

[changes/7.3][135]

 |
| 

[changes/8.0][136]

 | 

[changes/8.1][137]

 |
| 

[changes/8.1][138]

 | 

[changes/1.8][139]

 |
| 

[changes/index][140]

 | 

[changes/8.0][141]

 |
| 

[development/howtos/builders][142]

 | 

[usage/extensions/index][143]

 |
| 

[development/howtos/index][144]

 | 

[development/tutorials/index][145]

 |
| 

[development/howtos/setup\_extension][146]

 | 

[usage/extensions/index][147]

 |
| 

[development/html\_themes/index][148]

 | 

[usage/theming][149]

 |
| 

[development/html\_themes/templating][150]

 | 

[development/html\_themes/index][151]

 |
| 

[development/index][152]

 | 

[usage/index][153]

 |
| 

[development/tutorials/adding\_domain][154]

 | 

[extdev/domainapi][155]

 |
| 

[development/tutorials/autodoc\_ext][156]

 | 

[usage/extensions/autodoc][157]

 |
| 

[development/tutorials/examples/README][158]

 | 

[tutorial/end][159]

 |
| 

[development/tutorials/extending\_build][160]

 | 

[usage/extensions/todo][161]

 |
| 

[development/tutorials/extending\_syntax][162]

 | 

[extdev/markupapi][163]

 |
| 

[development/tutorials/index][164]

 | 

[development/howtos/index][165]

 |
| 

[examples][166]

 | 

[index][167]

 |
| 

[extdev/appapi][168]

 | 

[extdev/index][169]

 |
| 

[extdev/builderapi][170]

 | 

[usage/builders/index][171]

 |
| 

[extdev/collectorapi][172]

 | 

[extdev/envapi][173]

 |
| 

[extdev/deprecated][174]

 | 

[changes/1.8][175]

 |
| 

[extdev/domainapi][176]

 | 

[usage/domains/index][177]

 |
| 

[extdev/envapi][178]

 | 

[extdev/collectorapi][179]

 |
| 

[extdev/event\_callbacks][180]

 | 

[extdev/appapi][181]

 |
| 

[extdev/i18n][182]

 | 

[usage/advanced/intl][183]

 |
| 

[extdev/index][184]

 | 

[extdev/appapi][185]

 |
| 

[extdev/logging][186]

 | 

[extdev/appapi][187]

 |
| 

[extdev/markupapi][188]

 | 

[development/tutorials/extending\_syntax][189]

 |
| 

[extdev/nodes][190]

 | 

[extdev/domainapi][191]

 |
| 

[extdev/parserapi][192]

 | 

[extdev/appapi][193]

 |
| 

[extdev/projectapi][194]

 | 

[extdev/envapi][195]

 |
| 

[extdev/testing][196]

 | 

[internals/contributing][197]

 |
| 

[extdev/utils][198]

 | 

[extdev/appapi][199]

 |
| 

[faq][200]

 | 

[usage/configuration][201]

 |
| 

[glossary][202]

 | 

[usage/quickstart][203]

 |
| 

[index][204]

 | 

[usage/quickstart][205]

 |
| 

[internals/code-of-conduct][206]

 | 

[internals/index][207]

 |
| 

[internals/contributing][208]

 | 

[usage/advanced/intl][209]

 |
| 

[internals/index][210]

 | 

[usage/index][211]

 |
| 

[internals/organization][212]

 | 

[internals/contributing][213]

 |
| 

[internals/release-process][214]

 | 

[extdev/deprecated][215]

 |
| 

[latex][216]

 | 

[usage/configuration][217]

 |
| 

[man/index][218]

 | 

[usage/index][219]

 |
| 

[man/sphinx-apidoc][220]

 | 

[man/sphinx-autogen][221]

 |
| 

[man/sphinx-autogen][222]

 | 

[usage/extensions/autosummary][223]

 |
| 

[man/sphinx-build][224]

 | 

[usage/configuration][225]

 |
| 

[man/sphinx-quickstart][226]

 | 

[tutorial/getting-started][227]

 |
| 

[support][228]

 | 

[tutorial/end][229]

 |
| 

[tutorial/automatic-doc-generation][230]

 | 

[usage/extensions/autosummary][231]

 |
| 

[tutorial/deploying][232]

 | 

[tutorial/first-steps][233]

 |
| 

[tutorial/describing-code][234]

 | 

[usage/domains/index][235]

 |
| 

[tutorial/end][236]

 | 

[usage/index][237]

 |
| 

[tutorial/first-steps][238]

 | 

[tutorial/getting-started][239]

 |
| 

[tutorial/getting-started][240]

 | 

[tutorial/index][241]

 |
| 

[tutorial/index][242]

 | 

[tutorial/getting-started][243]

 |
| 

[tutorial/more-sphinx-customization][244]

 | 

[usage/theming][245]

 |
| 

[tutorial/narrative-documentation][246]

 | 

[usage/quickstart][247]

 |
| 

[usage/advanced/intl][248]

 | 

[internals/contributing][249]

 |
| 

[usage/advanced/websupport/api][250]

 | 

[usage/advanced/websupport/quickstart][251]

 |
| 

[usage/advanced/websupport/index][252]

 | 

[usage/advanced/websupport/quickstart][253]

 |
| 

[usage/advanced/websupport/quickstart][254]

 | 

[usage/advanced/websupport/api][255]

 |
| 

[usage/advanced/websupport/searchadapters][256]

 | 

[usage/advanced/websupport/api][257]

 |
| 

[usage/advanced/websupport/storagebackends][258]

 | 

[usage/advanced/websupport/api][259]

 |
| 

[usage/builders/index][260]

 | 

[usage/configuration][261]

 |
| 

[usage/configuration][262]

 | 

[changes/1.2][263]

 |
| 

[usage/domains/c][264]

 | 

[usage/domains/cpp][265]

 |
| 

[usage/domains/cpp][266]

 | 

[usage/domains/c][267]

 |
| 

[usage/domains/index][268]

 | 

[extdev/domainapi][269]

 |
| 

[usage/domains/javascript][270]

 | 

[usage/domains/python][271]

 |
| 

[usage/domains/mathematics][272]

 | 

[usage/referencing][273]

 |
| 

[usage/domains/python][274]

 | 

[extdev/domainapi][275]

 |
| 

[usage/domains/restructuredtext][276]

 | 

[extdev/markupapi][277]

 |
| 

[usage/domains/standard][278]

 | 

[usage/domains/index][279]

 |
| 

[usage/extensions/autodoc][280]

 | 

[tutorial/automatic-doc-generation][281]

 |
| 

[usage/extensions/autosectionlabel][282]

 | 

[usage/quickstart][283]

 |
| 

[usage/extensions/autosummary][284]

 | 

[tutorial/automatic-doc-generation][285]

 |
| 

[usage/extensions/coverage][286]

 | 

[usage/extensions/autodoc][287]

 |
| 

[usage/extensions/doctest][288]

 | 

[tutorial/describing-code][289]

 |
| 

[usage/extensions/duration][290]

 | 

[tutorial/more-sphinx-customization][291]

 |
| 

[usage/extensions/example\_google][292]

 | 

[usage/extensions/example\_numpy][293]

 |
| 

[usage/extensions/example\_numpy][294]

 | 

[usage/extensions/example\_google][295]

 |
| 

[usage/extensions/extlinks][296]

 | 

[usage/extensions/intersphinx][297]

 |
| 

[usage/extensions/githubpages][298]

 | 

[tutorial/deploying][299]

 |
| 

[usage/extensions/graphviz][300]

 | 

[usage/extensions/math][301]

 |
| 

[usage/extensions/ifconfig][302]

 | 

[usage/extensions/doctest][303]

 |
| 

[usage/extensions/imgconverter][304]

 | 

[usage/extensions/math][305]

 |
| 

[usage/extensions/index][306]

 | 

[development/index][307]

 |
| 

[usage/extensions/inheritance][308]

 | 

[usage/extensions/graphviz][309]

 |
| 

[usage/extensions/intersphinx][310]

 | 

[usage/quickstart][311]

 |
| 

[usage/extensions/linkcode][312]

 | 

[usage/extensions/viewcode][313]

 |
| 

[usage/extensions/math][314]

 | 

[usage/configuration][315]

 |
| 

[usage/extensions/napoleon][316]

 | 

[usage/extensions/example\_google][317]

 |
| 

[usage/extensions/todo][318]

 | 

[development/tutorials/extending\_build][319]

 |
| 

[usage/extensions/viewcode][320]

 | 

[usage/extensions/linkcode][321]

 |
| 

[usage/index][322]

 | 

[tutorial/end][323]

 |
| 

[usage/installation][324]

 | 

[tutorial/getting-started][325]

 |
| 

[usage/markdown][326]

 | 

[extdev/parserapi][327]

 |
| 

[usage/quickstart][328]

 | 

[index][329]

 |
| 

[usage/referencing][330]

 | 

[usage/restructuredtext/roles][331]

 |
| 

[usage/restructuredtext/basics][332]

 | 

[usage/restructuredtext/directives][333]

 |
| 

[usage/restructuredtext/directives][334]

 | 

[usage/restructuredtext/basics][335]

 |
| 

[usage/restructuredtext/domains][336]

 | 

[usage/domains/index][337]

 |
| 

[usage/restructuredtext/field-lists][338]

 | 

[usage/restructuredtext/directives][339]

 |
| 

[usage/restructuredtext/index][340]

 | 

[usage/restructuredtext/basics][341]

 |
| 

[usage/restructuredtext/roles][342]

 | 

[usage/referencing][343]

 |
| 

[usage/theming][344]

 | 

[development/html\_themes/index][345]

 |

[1]: #embeddings-are-underrated "Link to this heading"
[2]: #building-intuition-about-embeddings "Link to this heading"
[3]: #input-and-output "Link to this heading"
[4]: #first-how-to-literally-make-the-embeddings "Link to this heading"
[5]: https://ai.google.dev/gemini-api/docs/models/gemini#text-embedding
[6]: https://docs.voyageai.com/docs/embeddings
[7]: #does-it-cost-a-lot-of-money "Link to this heading"
[8]: #is-it-terrible-for-the-environment "Link to this heading"
[9]: https://cybernetist.com/2024/10/21/you-should-probably-pay-attention-to-tokenizers/
[10]: #what-model-is-best "Link to this heading"
[11]: https://seantrott.substack.com/p/tokenization-in-large-language-models
[12]: https://docs.voyageai.com/docs/embeddings
[13]: https://www.nomic.ai/blog/posts/nomic-embed-text-v1
[14]: https://platform.openai.com/docs/guides/embeddings/#embedding-models
[15]: https://docs.mistral.ai/getting-started/models/models_overview/#premier-models
[16]: https://ai.google.dev/gemini-api/docs/models/gemini#text-embedding
[17]: https://docs.cohere.com/v2/docs/models#embed
[18]: https://arxiv.org/abs/2210.07316
[19]: https://huggingface.co/spaces/mteb/leaderboard
[20]: https://seantrott.substack.com/p/tokenization-in-large-language-models
[21]: #very-weird-multi-dimensional-space "Link to this heading"
[22]: https://en.wikipedia.org/wiki/Latent_space
[23]: https://arxiv.org/pdf/1301.3781
[24]: https://en.m.wikipedia.org/wiki/Distributional_semantics
[25]: https://youtu.be/7ZYqjaLaK08
[26]: https://jalammar.github.io/illustrated-word2vec/
[27]: https://simonwillison.net/2023/Oct/23/embeddings/
[28]: #comparing-embeddings "Link to this heading"
[29]: https://www.coursera.org/learn/machine-learning-linear-algebra
[30]: https://numpy.org/doc/stable/
[31]: https://scikit-learn.org/stable/
[32]: #applications "Link to this heading"
[33]: #related-pages "Link to this heading"
[34]: https://www.sphinx-doc.org/en/master/
[35]: #embeddings-appendix-implementation
[36]: #embeddings-appendix-results
[37]: https://simonwillison.net/2023/Oct/23/embeddings/#related-content-using-embeddings
[38]: #let-a-thousand-embeddings-bloom "Link to this heading"
[39]: https://en.wikipedia.org/wiki/Well-known_URI
[40]: #parting-words "Link to this heading"
[41]: #appendix "Link to this heading"
[42]: #implementation "Link to this heading"
[43]: https://www.sphinx-doc.org/en/master/development/tutorials/extending_build.html
[44]: https://www.coursera.org/learn/machine-learning-linear-algebra
[45]: #results "Link to this heading"
[46]: https://www.sphinx-doc.org/en/master/authors.html
[47]: https://www.sphinx-doc.org/en/master/changes/0.6.html
[48]: https://www.sphinx-doc.org/en/master/changes/0.1.html
[49]: https://www.sphinx-doc.org/en/master/changes/0.5.html
[50]: https://www.sphinx-doc.org/en/master/changes/0.2.html
[51]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[52]: https://www.sphinx-doc.org/en/master/changes/0.3.html
[53]: https://www.sphinx-doc.org/en/master/changes/0.4.html
[54]: https://www.sphinx-doc.org/en/master/changes/0.4.html
[55]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[56]: https://www.sphinx-doc.org/en/master/changes/0.5.html
[57]: https://www.sphinx-doc.org/en/master/changes/0.6.html
[58]: https://www.sphinx-doc.org/en/master/changes/0.6.html
[59]: https://www.sphinx-doc.org/en/master/changes/1.6.html
[60]: https://www.sphinx-doc.org/en/master/changes/1.0.html
[61]: https://www.sphinx-doc.org/en/master/changes/1.3.html
[62]: https://www.sphinx-doc.org/en/master/changes/1.1.html
[63]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[64]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[65]: https://www.sphinx-doc.org/en/master/changes/1.1.html
[66]: https://www.sphinx-doc.org/en/master/changes/1.3.html
[67]: https://www.sphinx-doc.org/en/master/changes/1.4.html
[68]: https://www.sphinx-doc.org/en/master/changes/1.4.html
[69]: https://www.sphinx-doc.org/en/master/changes/1.3.html
[70]: https://www.sphinx-doc.org/en/master/changes/1.5.html
[71]: https://www.sphinx-doc.org/en/master/changes/1.6.html
[72]: https://www.sphinx-doc.org/en/master/changes/1.6.html
[73]: https://www.sphinx-doc.org/en/master/changes/1.5.html
[74]: https://www.sphinx-doc.org/en/master/changes/1.7.html
[75]: https://www.sphinx-doc.org/en/master/changes/1.8.html
[76]: https://www.sphinx-doc.org/en/master/changes/1.8.html
[77]: https://www.sphinx-doc.org/en/master/changes/1.6.html
[78]: https://www.sphinx-doc.org/en/master/changes/2.0.html
[79]: https://www.sphinx-doc.org/en/master/changes/1.8.html
[80]: https://www.sphinx-doc.org/en/master/changes/2.1.html
[81]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[82]: https://www.sphinx-doc.org/en/master/changes/2.2.html
[83]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[84]: https://www.sphinx-doc.org/en/master/changes/2.3.html
[85]: https://www.sphinx-doc.org/en/master/changes/2.1.html
[86]: https://www.sphinx-doc.org/en/master/changes/2.4.html
[87]: https://www.sphinx-doc.org/en/master/changes/3.5.html
[88]: https://www.sphinx-doc.org/en/master/changes/3.0.html
[89]: https://www.sphinx-doc.org/en/master/changes/4.3.html
[90]: https://www.sphinx-doc.org/en/master/changes/3.1.html
[91]: https://www.sphinx-doc.org/en/master/changes/3.3.html
[92]: https://www.sphinx-doc.org/en/master/changes/3.2.html
[93]: https://www.sphinx-doc.org/en/master/changes/3.0.html
[94]: https://www.sphinx-doc.org/en/master/changes/3.3.html
[95]: https://www.sphinx-doc.org/en/master/changes/3.1.html
[96]: https://www.sphinx-doc.org/en/master/changes/3.4.html
[97]: https://www.sphinx-doc.org/en/master/changes/4.3.html
[98]: https://www.sphinx-doc.org/en/master/changes/3.5.html
[99]: https://www.sphinx-doc.org/en/master/changes/1.3.html
[100]: https://www.sphinx-doc.org/en/master/changes/4.0.html
[101]: https://www.sphinx-doc.org/en/master/changes/3.0.html
[102]: https://www.sphinx-doc.org/en/master/changes/4.1.html
[103]: https://www.sphinx-doc.org/en/master/changes/4.4.html
[104]: https://www.sphinx-doc.org/en/master/changes/4.2.html
[105]: https://www.sphinx-doc.org/en/master/changes/4.4.html
[106]: https://www.sphinx-doc.org/en/master/changes/4.3.html
[107]: https://www.sphinx-doc.org/en/master/changes/3.0.html
[108]: https://www.sphinx-doc.org/en/master/changes/4.4.html
[109]: https://www.sphinx-doc.org/en/master/changes/7.4.html
[110]: https://www.sphinx-doc.org/en/master/changes/4.5.html
[111]: https://www.sphinx-doc.org/en/master/changes/4.4.html
[112]: https://www.sphinx-doc.org/en/master/changes/5.0.html
[113]: https://www.sphinx-doc.org/en/master/changes/3.5.html
[114]: https://www.sphinx-doc.org/en/master/changes/5.1.html
[115]: https://www.sphinx-doc.org/en/master/changes/5.0.html
[116]: https://www.sphinx-doc.org/en/master/changes/5.2.html
[117]: https://www.sphinx-doc.org/en/master/changes/3.5.html
[118]: https://www.sphinx-doc.org/en/master/changes/5.3.html
[119]: https://www.sphinx-doc.org/en/master/changes/5.2.html
[120]: https://www.sphinx-doc.org/en/master/changes/6.0.html
[121]: https://www.sphinx-doc.org/en/master/changes/6.2.html
[122]: https://www.sphinx-doc.org/en/master/changes/6.1.html
[123]: https://www.sphinx-doc.org/en/master/changes/6.2.html
[124]: https://www.sphinx-doc.org/en/master/changes/6.2.html
[125]: https://www.sphinx-doc.org/en/master/changes/6.1.html
[126]: https://www.sphinx-doc.org/en/master/changes/7.0.html
[127]: https://www.sphinx-doc.org/en/master/extdev/deprecated.html
[128]: https://www.sphinx-doc.org/en/master/changes/7.1.html
[129]: https://www.sphinx-doc.org/en/master/changes/7.2.html
[130]: https://www.sphinx-doc.org/en/master/changes/7.2.html
[131]: https://www.sphinx-doc.org/en/master/changes/7.4.html
[132]: https://www.sphinx-doc.org/en/master/changes/7.3.html
[133]: https://www.sphinx-doc.org/en/master/changes/7.4.html
[134]: https://www.sphinx-doc.org/en/master/changes/7.4.html
[135]: https://www.sphinx-doc.org/en/master/changes/7.3.html
[136]: https://www.sphinx-doc.org/en/master/changes/8.0.html
[137]: https://www.sphinx-doc.org/en/master/changes/8.1.html
[138]: https://www.sphinx-doc.org/en/master/changes/8.1.html
[139]: https://www.sphinx-doc.org/en/master/changes/1.8.html
[140]: https://www.sphinx-doc.org/en/master/changes/index.html
[141]: https://www.sphinx-doc.org/en/master/changes/8.0.html
[142]: https://www.sphinx-doc.org/en/master/development/howtos/builders.html
[143]: https://www.sphinx-doc.org/en/master/usage/extensions/index.html
[144]: https://www.sphinx-doc.org/en/master/development/howtos/index.html
[145]: https://www.sphinx-doc.org/en/master/development/tutorials/index.html
[146]: https://www.sphinx-doc.org/en/master/development/howtos/setup_extension.html
[147]: https://www.sphinx-doc.org/en/master/usage/extensions/index.html
[148]: https://www.sphinx-doc.org/en/master/development/html_themes/index.html
[149]: https://www.sphinx-doc.org/en/master/usage/theming.html
[150]: https://www.sphinx-doc.org/en/master/development/html_themes/templating.html
[151]: https://www.sphinx-doc.org/en/master/development/html_themes/index.html
[152]: https://www.sphinx-doc.org/en/master/development/index.html
[153]: https://www.sphinx-doc.org/en/master/usage/index.html
[154]: https://www.sphinx-doc.org/en/master/development/tutorials/adding_domain.html
[155]: https://www.sphinx-doc.org/en/master/extdev/domainapi.html
[156]: https://www.sphinx-doc.org/en/master/development/tutorials/autodoc_ext.html
[157]: https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
[158]: https://www.sphinx-doc.org/en/master/development/tutorials/examples/README.html
[159]: https://www.sphinx-doc.org/en/master/tutorial/end.html
[160]: https://www.sphinx-doc.org/en/master/development/tutorials/extending_build.html
[161]: https://www.sphinx-doc.org/en/master/usage/extensions/todo.html
[162]: https://www.sphinx-doc.org/en/master/development/tutorials/extending_syntax.html
[163]: https://www.sphinx-doc.org/en/master/extdev/markupapi.html
[164]: https://www.sphinx-doc.org/en/master/development/tutorials/index.html
[165]: https://www.sphinx-doc.org/en/master/development/howtos/index.html
[166]: https://www.sphinx-doc.org/en/master/examples.html
[167]: https://www.sphinx-doc.org/en/master/index.html
[168]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[169]: https://www.sphinx-doc.org/en/master/extdev/index.html
[170]: https://www.sphinx-doc.org/en/master/extdev/builderapi.html
[171]: https://www.sphinx-doc.org/en/master/usage/builders/index.html
[172]: https://www.sphinx-doc.org/en/master/extdev/collectorapi.html
[173]: https://www.sphinx-doc.org/en/master/extdev/envapi.html
[174]: https://www.sphinx-doc.org/en/master/extdev/deprecated.html
[175]: https://www.sphinx-doc.org/en/master/changes/1.8.html
[176]: https://www.sphinx-doc.org/en/master/extdev/domainapi.html
[177]: https://www.sphinx-doc.org/en/master/usage/domains/index.html
[178]: https://www.sphinx-doc.org/en/master/extdev/envapi.html
[179]: https://www.sphinx-doc.org/en/master/extdev/collectorapi.html
[180]: https://www.sphinx-doc.org/en/master/extdev/event_callbacks.html
[181]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[182]: https://www.sphinx-doc.org/en/master/extdev/i18n.html
[183]: https://www.sphinx-doc.org/en/master/usage/advanced/intl.html
[184]: https://www.sphinx-doc.org/en/master/extdev/index.html
[185]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[186]: https://www.sphinx-doc.org/en/master/extdev/logging.html
[187]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[188]: https://www.sphinx-doc.org/en/master/extdev/markupapi.html
[189]: https://www.sphinx-doc.org/en/master/development/tutorials/extending_syntax.html
[190]: https://www.sphinx-doc.org/en/master/extdev/nodes.html
[191]: https://www.sphinx-doc.org/en/master/extdev/domainapi.html
[192]: https://www.sphinx-doc.org/en/master/extdev/parserapi.html
[193]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[194]: https://www.sphinx-doc.org/en/master/extdev/projectapi.html
[195]: https://www.sphinx-doc.org/en/master/extdev/envapi.html
[196]: https://www.sphinx-doc.org/en/master/extdev/testing.html
[197]: https://www.sphinx-doc.org/en/master/internals/contributing.html
[198]: https://www.sphinx-doc.org/en/master/extdev/utils.html
[199]: https://www.sphinx-doc.org/en/master/extdev/appapi.html
[200]: https://www.sphinx-doc.org/en/master/faq.html
[201]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[202]: https://www.sphinx-doc.org/en/master/glossary.html
[203]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[204]: https://www.sphinx-doc.org/en/master/index.html
[205]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[206]: https://www.sphinx-doc.org/en/master/internals/code-of-conduct.html
[207]: https://www.sphinx-doc.org/en/master/internals/index.html
[208]: https://www.sphinx-doc.org/en/master/internals/contributing.html
[209]: https://www.sphinx-doc.org/en/master/usage/advanced/intl.html
[210]: https://www.sphinx-doc.org/en/master/internals/index.html
[211]: https://www.sphinx-doc.org/en/master/usage/index.html
[212]: https://www.sphinx-doc.org/en/master/internals/organization.html
[213]: https://www.sphinx-doc.org/en/master/internals/contributing.html
[214]: https://www.sphinx-doc.org/en/master/internals/release-process.html
[215]: https://www.sphinx-doc.org/en/master/extdev/deprecated.html
[216]: https://www.sphinx-doc.org/en/master/latex.html
[217]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[218]: https://www.sphinx-doc.org/en/master/man/index.html
[219]: https://www.sphinx-doc.org/en/master/usage/index.html
[220]: https://www.sphinx-doc.org/en/master/man/sphinx-apidoc.html
[221]: https://www.sphinx-doc.org/en/master/man/sphinx-autogen.html
[222]: https://www.sphinx-doc.org/en/master/man/sphinx-autogen.html
[223]: https://www.sphinx-doc.org/en/master/usage/extensions/autosummary.html
[224]: https://www.sphinx-doc.org/en/master/man/sphinx-build.html
[225]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[226]: https://www.sphinx-doc.org/en/master/man/sphinx-quickstart.html
[227]: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
[228]: https://www.sphinx-doc.org/en/master/support.html
[229]: https://www.sphinx-doc.org/en/master/tutorial/end.html
[230]: https://www.sphinx-doc.org/en/master/tutorial/automatic-doc-generation.html
[231]: https://www.sphinx-doc.org/en/master/usage/extensions/autosummary.html
[232]: https://www.sphinx-doc.org/en/master/tutorial/deploying.html
[233]: https://www.sphinx-doc.org/en/master/tutorial/first-steps.html
[234]: https://www.sphinx-doc.org/en/master/tutorial/describing-code.html
[235]: https://www.sphinx-doc.org/en/master/usage/domains/index.html
[236]: https://www.sphinx-doc.org/en/master/tutorial/end.html
[237]: https://www.sphinx-doc.org/en/master/usage/index.html
[238]: https://www.sphinx-doc.org/en/master/tutorial/first-steps.html
[239]: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
[240]: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
[241]: https://www.sphinx-doc.org/en/master/tutorial/index.html
[242]: https://www.sphinx-doc.org/en/master/tutorial/index.html
[243]: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
[244]: https://www.sphinx-doc.org/en/master/tutorial/more-sphinx-customization.html
[245]: https://www.sphinx-doc.org/en/master/usage/theming.html
[246]: https://www.sphinx-doc.org/en/master/tutorial/narrative-documentation.html
[247]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[248]: https://www.sphinx-doc.org/en/master/usage/advanced/intl.html
[249]: https://www.sphinx-doc.org/en/master/internals/contributing.html
[250]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/api.html
[251]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/quickstart.html
[252]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/index.html
[253]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/quickstart.html
[254]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/quickstart.html
[255]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/api.html
[256]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/searchadapters.html
[257]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/api.html
[258]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/storagebackends.html
[259]: https://www.sphinx-doc.org/en/master/usage/advanced/websupport/api.html
[260]: https://www.sphinx-doc.org/en/master/usage/builders/index.html
[261]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[262]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[263]: https://www.sphinx-doc.org/en/master/changes/1.2.html
[264]: https://www.sphinx-doc.org/en/master/usage/domains/c.html
[265]: https://www.sphinx-doc.org/en/master/usage/domains/cpp.html
[266]: https://www.sphinx-doc.org/en/master/usage/domains/cpp.html
[267]: https://www.sphinx-doc.org/en/master/usage/domains/c.html
[268]: https://www.sphinx-doc.org/en/master/usage/domains/index.html
[269]: https://www.sphinx-doc.org/en/master/extdev/domainapi.html
[270]: https://www.sphinx-doc.org/en/master/usage/domains/javascript.html
[271]: https://www.sphinx-doc.org/en/master/usage/domains/python.html
[272]: https://www.sphinx-doc.org/en/master/usage/domains/mathematics.html
[273]: https://www.sphinx-doc.org/en/master/usage/referencing.html
[274]: https://www.sphinx-doc.org/en/master/usage/domains/python.html
[275]: https://www.sphinx-doc.org/en/master/extdev/domainapi.html
[276]: https://www.sphinx-doc.org/en/master/usage/domains/restructuredtext.html
[277]: https://www.sphinx-doc.org/en/master/extdev/markupapi.html
[278]: https://www.sphinx-doc.org/en/master/usage/domains/standard.html
[279]: https://www.sphinx-doc.org/en/master/usage/domains/index.html
[280]: https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
[281]: https://www.sphinx-doc.org/en/master/tutorial/automatic-doc-generation.html
[282]: https://www.sphinx-doc.org/en/master/usage/extensions/autosectionlabel.html
[283]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[284]: https://www.sphinx-doc.org/en/master/usage/extensions/autosummary.html
[285]: https://www.sphinx-doc.org/en/master/tutorial/automatic-doc-generation.html
[286]: https://www.sphinx-doc.org/en/master/usage/extensions/coverage.html
[287]: https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
[288]: https://www.sphinx-doc.org/en/master/usage/extensions/doctest.html
[289]: https://www.sphinx-doc.org/en/master/tutorial/describing-code.html
[290]: https://www.sphinx-doc.org/en/master/usage/extensions/duration.html
[291]: https://www.sphinx-doc.org/en/master/tutorial/more-sphinx-customization.html
[292]: https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html
[293]: https://www.sphinx-doc.org/en/master/usage/extensions/example_numpy.html
[294]: https://www.sphinx-doc.org/en/master/usage/extensions/example_numpy.html
[295]: https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html
[296]: https://www.sphinx-doc.org/en/master/usage/extensions/extlinks.html
[297]: https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html
[298]: https://www.sphinx-doc.org/en/master/usage/extensions/githubpages.html
[299]: https://www.sphinx-doc.org/en/master/tutorial/deploying.html
[300]: https://www.sphinx-doc.org/en/master/usage/extensions/graphviz.html
[301]: https://www.sphinx-doc.org/en/master/usage/extensions/math.html
[302]: https://www.sphinx-doc.org/en/master/usage/extensions/ifconfig.html
[303]: https://www.sphinx-doc.org/en/master/usage/extensions/doctest.html
[304]: https://www.sphinx-doc.org/en/master/usage/extensions/imgconverter.html
[305]: https://www.sphinx-doc.org/en/master/usage/extensions/math.html
[306]: https://www.sphinx-doc.org/en/master/usage/extensions/index.html
[307]: https://www.sphinx-doc.org/en/master/development/index.html
[308]: https://www.sphinx-doc.org/en/master/usage/extensions/inheritance.html
[309]: https://www.sphinx-doc.org/en/master/usage/extensions/graphviz.html
[310]: https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html
[311]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[312]: https://www.sphinx-doc.org/en/master/usage/extensions/linkcode.html
[313]: https://www.sphinx-doc.org/en/master/usage/extensions/viewcode.html
[314]: https://www.sphinx-doc.org/en/master/usage/extensions/math.html
[315]: https://www.sphinx-doc.org/en/master/usage/configuration.html
[316]: https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html
[317]: https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html
[318]: https://www.sphinx-doc.org/en/master/usage/extensions/todo.html
[319]: https://www.sphinx-doc.org/en/master/development/tutorials/extending_build.html
[320]: https://www.sphinx-doc.org/en/master/usage/extensions/viewcode.html
[321]: https://www.sphinx-doc.org/en/master/usage/extensions/linkcode.html
[322]: https://www.sphinx-doc.org/en/master/usage/index.html
[323]: https://www.sphinx-doc.org/en/master/tutorial/end.html
[324]: https://www.sphinx-doc.org/en/master/usage/installation.html
[325]: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
[326]: https://www.sphinx-doc.org/en/master/usage/markdown.html
[327]: https://www.sphinx-doc.org/en/master/extdev/parserapi.html
[328]: https://www.sphinx-doc.org/en/master/usage/quickstart.html
[329]: https://www.sphinx-doc.org/en/master/index.html
[330]: https://www.sphinx-doc.org/en/master/usage/referencing.html
[331]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html
[332]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html
[333]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html
[334]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html
[335]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html
[336]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/domains.html
[337]: https://www.sphinx-doc.org/en/master/usage/domains/index.html
[338]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/field-lists.html
[339]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html
[340]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html
[341]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html
[342]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html
[343]: https://www.sphinx-doc.org/en/master/usage/referencing.html
[344]: https://www.sphinx-doc.org/en/master/usage/theming.html
[345]: https://www.sphinx-doc.org/en/master/development/html_themes/index.html