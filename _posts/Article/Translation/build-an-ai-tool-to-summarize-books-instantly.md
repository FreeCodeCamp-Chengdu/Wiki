---
title: Build an AI Tool to Summarize Books Instantly
authorURL: ""
originalURL: https://levelup.gitconnected.com/build-an-ai-tool-to-summarize-books-instantly-828680c1ceb4
translator: ""
reviewer: ""
---

## Get the gist of any book without reading it cover to cover.

[![Alessandro Amenta](https://miro.medium.com/v2/resize:fill:88:88/1*eJmL2XsmvUfWxRbTJrfHvQ.png)](https://medium.com/@alessandroamenta1?source=post_page-----828680c1ceb4--------------------------------)

[![Level Up Coding](https://miro.medium.com/v2/resize:fill:48:48/1*5D9oYBd58pyjMkV_5-zXXQ.jpeg)](https://levelup.gitconnected.com/?source=post_page-----828680c1ceb4--------------------------------)

[Alessandro Amenta](https://medium.com/@alessandroamenta1?source=post_page-----828680c1ceb4--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Ff39ff33c76d2&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&user=Alessandro+Amenta&userId=f39ff33c76d2&source=post_page-f39ff33c76d2----828680c1ceb4---------------------post_header-----------)

Published in

[Level Up Coding](https://levelup.gitconnected.com/?source=post_page-----828680c1ceb4--------------------------------)
·
4 min read
·
4 days ago
[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fgitconnected%2F828680c1ceb4&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&user=Alessandro+Amenta&userId=f39ff33c76d2&source=-----828680c1ceb4---------------------clap_footer-----------)

\--

1

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F828680c1ceb4&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&source=-----828680c1ceb4---------------------bookmark_footer-----------)

In this article, we’ll build a simple yet powerful book summarizer using Python, Langchain and OpenAI embeddings.

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*DFqc1P6PnOZ8S95puZqSEg.png)

Generated with DALL·E 3.

<!-- more -->

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
from langchain.document_loaders import PyPDFLoader, UnstructuredEPubLoader

def load_book(file_obj, file_extension):  
 """Load the content of a book based on its file type."""  
 text = ""  
 with tempfile.NamedTemporaryFile(delete=False, suffix=file_extension) as temp_file:  
 temp_file.write(file_obj.read())  
 if file_extension == ".pdf":  
 loader = PyPDFLoader(temp_file.name)  
 pages = loader.load()  
 text = "".join(page.page_content for page in pages)  
 elif file_extension == ".epub":  
 loader = UnstructuredEPubLoader(temp_file.name)  
 data = loader.load()  
 text = "\\n".join(element.page_content for element in data)  
 else:  
 raise ValueError(f"Unsupported file extension: {file_extension}")  
 os.remove(temp_file.name)  
 text = text.replace('\\t', ' ')  
 return text

# Step 2: Split and Embed the Text

AI models have a token limit, which means they can’t process a whole book at once. By splitting the text into chunks, we ensure that each section is digestible for the AI.

We’ll split the text into chunks and convert them into embeddings. Embeddings convert text into a compact numerical form quickly and with minimal computation, making the process both fast and cost-effective.

from langchain.text_splitter import RecursiveCharacterTextSplitter  
from langchain.embeddings import OpenAIEmbeddings

def split_and_embed(text, openai_api_key):  
 text_splitter = RecursiveCharacterTextSplitter(separators=\["\\n\\n", "\\n", "\\t"\], chunk_size=10000, chunk_overlap=3000)  
 docs = text_splitter.create_documents(\[text\])  
 embeddings = OpenAIEmbeddings(openai_api_key=openai_api_key)  
 vectors = embeddings.embed_documents(\[x.page_content for x in docs\])  
 return docs, vectors

# Step 3: Cluster the Embeddings

We use KMeans clustering to group similar chunks. In my version, as you can see below, I found 11 as the number of clusters to work quite well for most books. But you can tweak that for your use case.

Here, after we’ve turned the whole book into chunks and then into embeddings. These embeddings are grouped based on their similarity. For each group, we pick the most representative embedding and map it back to its corresponding text chunk.

from sklearn.cluster import KMeans  
import numpy as np

def cluster_embeddings(vectors, num_clusters):  
 kmeans = KMeans(n_clusters=num_clusters, random_state=42).fit(vectors)  
 closest_indices = \[np.argmin(np.linalg.norm(vectors - center, axis=1)) for center in kmeans.cluster_centers\_\]  
 return sorted(closest_indices)

# Step 4: Summarize the Representative Chunks

We’ll summarize only the selected chunks using GPT-3.5.

from langchain.chains.summarize import load_summarize_chain  
from langchain.prompts import PromptTemplate

def summarize_chunks(docs, selected_indices, openai_api_key):  
 llm3_turbo = ChatOpenAI(temperature=0, openai_api_key=openai_api_key, max_tokens=1000, model='gpt-3.5-turbo-16k')  
 map_prompt = """  
 You are provided with a passage from a book. Your task is to produce a comprehensive summary of this passage. Ensure accuracy and avoid adding any interpretations or extra details not present in the original text. The summary should be at least three paragraphs long and fully capture the essence of the passage.  
 \`\`\`{text}\`\`\`  
 SUMMARY:  
 """  
 map_prompt_template = PromptTemplate(template=map_prompt, input_variables=\["text"\])  
 selected_docs = \[docs\[i\] for i in selected_indices\]  
 summary_list = \[\]

    for doc in selected\_docs:
        chunk\_summary = load\_summarize\_chain(llm=llm3\_turbo, chain\_type="stuff", prompt=map\_prompt\_template).run(\[doc\])
        summary\_list.append(chunk\_summary)

    return "\\n".join(summary\_list)

# Step 5: Create the Final Summary

We combine the individual summaries into one final, cohesive summary using GPT-4.

from langchain.schema import Document  
from langchain.chat_models import ChatOpenAI

def create_final_summary(summaries, openai_api_key):  
 llm4 = ChatOpenAI(temperature=0, openai_api_key=openai_api_key, max_tokens=3000, model='gpt-4', request_timeout=120)  
 combine_prompt = """  
 You are given a series of summarized sections from a book. Your task is to weave these summaries into a single, cohesive, and verbose summary. The reader should be able to understand the main events or points of the book from your summary. Ensure you retain the accuracy of the content and present it in a clear and engaging manner.  
 \`\`\`{text}\`\`\`  
 COHESIVE SUMMARY:  
 """  
 combine_prompt_template = PromptTemplate(template=combine_prompt, input_variables=\["text"\])  
 reduce_chain = load_summarize_chain(llm=llm4, chain_type="stuff", prompt=combine_prompt_template)  
 final_summary = reduce_chain.run(\[Document(page_content=summaries)\])  
 return final_summary

# Bringing It All Together

Now, we combine all the steps into a single function that takes an uploaded file and generates a summary.

\# ... (previous code for imports and functions)

def generate_summary(uploaded_file, openai_api_key, num_clusters=11, verbose=False):  
 file_extension = os.path.splitext(uploaded_file.name)\[1\].lower()  
 text = load_book(uploaded_file, file_extension)  
 docs, vectors = split_and_embed(text, openai_api_key)  
 selected_indices = cluster_embeddings(vectors, num_clusters)  
 summaries = summarize_chunks(docs, selected_indices, openai_api_key)  
 final_summary = create_final_summary(summaries, openai_api_key)  
 return final_summary

# Testing the Summarizer

Finally, we can test our summarizer with a book file.

\# Testing the summarizer  
if \_\_name\_\_ == '\_\_main\_\_':  
 load_dotenv()  
 openai_api_key = os.getenv('OPENAI_API_KEY')  
 book_path = "path_to_your_book.epub"  
 with open(book_path, 'rb') as uploaded_file:  
 summary = generate_summary(uploaded_file, openai_api_key, verbose=True)  
 print(summary)

# Wrapping up

This tool can help you quickly understand the key points of any book quickly. The approach we’ve taken is not only cheap but also works for books of any length.

Remember, the quality of the summary depends on the clustering and the summarization prompts, so feel free to tweak those to your needs.

You can try out the Streamlit app and see the summarizer here: [GPT Summarizer App](https://gptsummarizer.streamlit.app/).

I hope this article has been helpful. If you have any questions or feedback, please leave a comment or reach out.

Happy coding! :)
