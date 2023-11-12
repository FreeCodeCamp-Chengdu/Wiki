---
title: Build an AI Tool to Summarize Books Instantly
authorURL: ""
originalURL: https://levelup.gitconnected.com/build-an-ai-tool-to-summarize-books-instantly-828680c1ceb4
translator: ""
reviewer: ""
---

# Build an AI Tool to Summarize Books Instantly

## Get the gist of any book without reading it cover to cover.

[

![Alessandro Amenta](https://miro.medium.com/v2/resize:fill:88:88/1*eJmL2XsmvUfWxRbTJrfHvQ.png)









](https://medium.com/@alessandroamenta1?source=post_page-----828680c1ceb4--------------------------------)[

![Level Up Coding](https://miro.medium.com/v2/resize:fill:48:48/1*5D9oYBd58pyjMkV_5-zXXQ.jpeg)











](https://levelup.gitconnected.com/?source=post_page-----828680c1ceb4--------------------------------)

[Alessandro Amenta](https://medium.com/@alessandroamenta1?source=post_page-----828680c1ceb4--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Ff39ff33c76d2&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&user=Alessandro+Amenta&userId=f39ff33c76d2&source=post_page-f39ff33c76d2----828680c1ceb4---------------------post_header-----------)

Published in

[

Level Up Coding

](https://levelup.gitconnected.com/?source=post_page-----828680c1ceb4--------------------------------)

·

4 min read

·

4 days ago

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fgitconnected%2F828680c1ceb4&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&user=Alessandro+Amenta&userId=f39ff33c76d2&source=-----828680c1ceb4---------------------clap_footer-----------)

\--

1

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F828680c1ceb4&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&source=-----828680c1ceb4---------------------bookmark_footer-----------)

Listen

Share

In this article, we’ll build a simple yet powerful book summarizer using Python, Langchain and OpenAI embeddings.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*DFqc1P6PnOZ8S95puZqSEg.png)

Generated with DALL·E 3.

# The Challenge

AI models like GPT-3 and GPT-4 are incredibly powerful, but they have their limitations. One significant limitation is the context window, which restricts the amount of text the model can consider at any one time. This means you can’t just feed an entire book into the model and expect a coherent summary. Moreover, processing large texts can be costly.

# The Solution

To overcome these challenges, we’ve devised a method that is both cost-effective and efficient. Here’s the process:

# The Process Simplified

Here’s how we transform a full-length book into a concise summary:

1.  **Splitting & Embeddings**: We break down the book into smaller chunks and convert them into embeddings. This step is surprisingly affordable.
2.  **Clustering:** Next, we cluster these embeddings to find the most representative sections of the book.
3.  **Summarization:** We then summarize these key sections using the more cost-effective GPT-3.5 model.
4.  **Combining Summaries:** Finally, we use GPT-4 to stitch these summaries into one fluid narrative.

By using GPT-4 only in the final step, we manage to keep our costs low.

Now, let’s break down the code and the rationale behind each step.Building the Summarizer

Let’s dive into the code and build our summarizer step by step.

# Step 1: Load the Book

First, we need to read the book’s content. We’ll support PDF and EPUB formats.

import os  
import tempfile  
from langchain.document\_loaders import PyPDFLoader, UnstructuredEPubLoader  
  
def load\_book(file\_obj, file\_extension):  
    """Load the content of a book based on its file type."""  
    text = ""  
    with tempfile.NamedTemporaryFile(delete=False, suffix=file\_extension) as temp\_file:  
        temp\_file.write(file\_obj.read())  
        if file\_extension == ".pdf":  
            loader = PyPDFLoader(temp\_file.name)  
            pages = loader.load()  
            text = "".join(page.page\_content for page in pages)  
        elif file\_extension == ".epub":  
            loader = UnstructuredEPubLoader(temp\_file.name)  
            data = loader.load()  
            text = "\\n".join(element.page\_content for element in data)  
        else:  
            raise ValueError(f"Unsupported file extension: {file\_extension}")  
        os.remove(temp\_file.name)  
    text = text.replace('\\t', ' ')  
    return text

# Step 2: Split and Embed the Text

AI models have a token limit, which means they can’t process a whole book at once. By splitting the text into chunks, we ensure that each section is digestible for the AI.

We’ll split the text into chunks and convert them into embeddings. Embeddings convert text into a compact numerical form quickly and with minimal computation, making the process both fast and cost-effective.

from langchain.text\_splitter import RecursiveCharacterTextSplitter  
from langchain.embeddings import OpenAIEmbeddings  
  
def split\_and\_embed(text, openai\_api\_key):  
    text\_splitter = RecursiveCharacterTextSplitter(separators=\["\\n\\n", "\\n", "\\t"\], chunk\_size=10000, chunk\_overlap=3000)  
    docs = text\_splitter.create\_documents(\[text\])  
    embeddings = OpenAIEmbeddings(openai\_api\_key=openai\_api\_key)  
    vectors = embeddings.embed\_documents(\[x.page\_content for x in docs\])  
    return docs, vectors

# Step 3: Cluster the Embeddings

We use KMeans clustering to group similar chunks. In my version, as you can see below, I found 11 as the number of clusters to work quite well for most books. But you can tweak that for your use case.

Here, after we’ve turned the whole book into chunks and then into embeddings. These embeddings are grouped based on their similarity. For each group, we pick the most representative embedding and map it back to its corresponding text chunk.

from sklearn.cluster import KMeans  
import numpy as np  
  
def cluster\_embeddings(vectors, num\_clusters):  
    kmeans = KMeans(n\_clusters=num\_clusters, random\_state=42).fit(vectors)  
    closest\_indices = \[np.argmin(np.linalg.norm(vectors - center, axis=1)) for center in kmeans.cluster\_centers\_\]  
    return sorted(closest\_indices)

# Step 4: Summarize the Representative Chunks

We’ll summarize only the selected chunks using GPT-3.5.

from langchain.chains.summarize import load\_summarize\_chain  
from langchain.prompts import PromptTemplate  
  
def summarize\_chunks(docs, selected\_indices, openai\_api\_key):  
    llm3\_turbo = ChatOpenAI(temperature=0, openai\_api\_key=openai\_api\_key, max\_tokens=1000, model='gpt-3.5-turbo-16k')  
    map\_prompt = """  
    You are provided with a passage from a book. Your task is to produce a comprehensive summary of this passage. Ensure accuracy and avoid adding any interpretations or extra details not present in the original text. The summary should be at least three paragraphs long and fully capture the essence of the passage.  
    \`\`\`{text}\`\`\`  
    SUMMARY:  
    """  
    map\_prompt\_template = PromptTemplate(template=map\_prompt, input\_variables=\["text"\])  
    selected\_docs = \[docs\[i\] for i in selected\_indices\]  
    summary\_list = \[\]  
  
    for doc in selected\_docs:  
        chunk\_summary = load\_summarize\_chain(llm=llm3\_turbo, chain\_type="stuff", prompt=map\_prompt\_template).run(\[doc\])  
        summary\_list.append(chunk\_summary)  
      
    return "\\n".join(summary\_list)

# Step 5: Create the Final Summary

We combine the individual summaries into one final, cohesive summary using GPT-4.

from langchain.schema import Document  
from langchain.chat\_models import ChatOpenAI  
  
def create\_final\_summary(summaries, openai\_api\_key):  
    llm4 = ChatOpenAI(temperature=0, openai\_api\_key=openai\_api\_key, max\_tokens=3000, model='gpt-4', request\_timeout=120)  
    combine\_prompt = """  
    You are given a series of summarized sections from a book. Your task is to weave these summaries into a single, cohesive, and verbose summary. The reader should be able to understand the main events or points of the book from your summary. Ensure you retain the accuracy of the content and present it in a clear and engaging manner.  
    \`\`\`{text}\`\`\`  
    COHESIVE SUMMARY:  
    """  
    combine\_prompt\_template = PromptTemplate(template=combine\_prompt, input\_variables=\["text"\])  
    reduce\_chain = load\_summarize\_chain(llm=llm4, chain\_type="stuff", prompt=combine\_prompt\_template)  
    final\_summary = reduce\_chain.run(\[Document(page\_content=summaries)\])  
    return final\_summary

# Bringing It All Together

Now, we combine all the steps into a single function that takes an uploaded file and generates a summary.

\# ... (previous code for imports and functions)  
  
def generate\_summary(uploaded\_file, openai\_api\_key, num\_clusters=11, verbose=False):  
    file\_extension = os.path.splitext(uploaded\_file.name)\[1\].lower()  
    text = load\_book(uploaded\_file, file\_extension)  
    docs, vectors = split\_and\_embed(text, openai\_api\_key)  
    selected\_indices = cluster\_embeddings(vectors, num\_clusters)  
    summaries = summarize\_chunks(docs, selected\_indices, openai\_api\_key)  
    final\_summary = create\_final\_summary(summaries, openai\_api\_key)  
    return final\_summary

# Testing the Summarizer

Finally, we can test our summarizer with a book file.

\# Testing the summarizer  
if \_\_name\_\_ == '\_\_main\_\_':  
    load\_dotenv()  
    openai\_api\_key = os.getenv('OPENAI\_API\_KEY')  
    book\_path = "path\_to\_your\_book.epub"  
    with open(book\_path, 'rb') as uploaded\_file:  
        summary = generate\_summary(uploaded\_file, openai\_api\_key, verbose=True)  
        print(summary)

# Wrapping up

This tool can help you quickly understand the key points of any book quickly. The approach we’ve taken is not only cheap but also works for books of any length.

Remember, the quality of the summary depends on the clustering and the summarization prompts, so feel free to tweak those to your needs.

You can try out the Streamlit app and see the summarizer here: [GPT Summarizer App](https://gptsummarizer.streamlit.app/).

I hope this article has been helpful. If you have any questions or feedback, please leave a comment or reach out.

Happy coding! :)