---
title: 打造一款人工智能工具，即时总结书籍内容
authorURL: ""
originalURL: https://levelup.gitconnected.com/build-an-ai-tool-to-summarize-books-instantly-828680c1ceb4
translator: ""
reviewer: ""
---

## 无需从头到尾阅读一本书，就能掌握其要点。
[![Alessandro Amenta](https://miro.medium.com/v2/resize:fill:88:88/1*eJmL2XsmvUfWxRbTJrfHvQ.png)](https://medium.com/@alessandroamenta1?source=post_page-----828680c1ceb4--------------------------------)


[关注我](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Ff39ff33c76d2&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Fbuild-an-ai-tool-to-summarize-books-instantly-828680c1ceb4&user=Alessandro+Amenta&userId=f39ff33c76d2&source=post_page-f39ff33c76d2----828680c1ceb4---------------------post_header-----------)


[提升编码水平](https://levelup.gitconnected.com/?source=post_page-----828680c1ceb4--------------------------------)


在本文中，我们将使用 Python、Langchain 和 OpenAI embeddings构建一个简单但功能强大的书籍摘要器。

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*DFqc1P6PnOZ8S95puZqSEg.png)

用 DALL-E 3 生成上图。

# 面临的挑战

GPT-3 和 GPT-4 等人工智能模型功能强大，但也有其局限性。其中一个重要的限制就是上下文窗口，它限制了模型在同一时间可以考虑的文本数量。这意味着你不能把整本书输入模型，然后期待得到一个连贯的摘要。此外，处理大量文本的成本也很高。

# 解决方案

为了克服这些挑战，我们设计了一种既经济又高效的方法。流程如下:

# 简化流程

以下是我们如何将长篇大论的书籍转化为简明扼要的摘要：

1.  **拆分（Splitting）与嵌入（Embeddings）**： 我们将图书分解成小块，并将其转换为嵌入式。这一步的成本出奇地低。
2.  **聚类（Clustering）:** 接下来，我们对这些嵌入式内容进行聚类，找出书中最具代表性的部分。
3.  **摘要（Summarization）:** 然后，我们使用更具成本效益的 GPT-3.5 模型对这些关键部分进行摘要。
4.  **合并摘要（Combining Summaries）:** 最后，我们使用 GPT-4 将这些摘要拼接成一个流畅的叙述。

通过仅在最后一步使用 GPT-4，我们成功地降低了成本。

现在，让我们来分析一下代码和每个步骤背后的原理。

让我们深入代码，一步步构建我们的摘要。

# 步骤 1: 加载图书

首先，我们需要阅读图书内容。我们将支持 PDF 和 EPUB 格式。

```python
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
            text = "\n".join(element.page_content for element in data)
        else:
            raise ValueError(f"Unsupported file extension: {file_extension}")
        os.remove(temp_file.name)
    text = text.replace('\t', ' ')
    return text
```
# 步骤 2: 分割和嵌入文本

人工智能模型有代币限制，这意味着它们无法一次性处理一整本书。通过将文本分割成块，我们可以确保人工智能可以消化每一部分。

我们将文本分割成块，然后将它们转换成嵌入式。嵌入式将文本快速转换为紧凑的数字形式，计算量极小，因此整个过程既快速又经济。

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings

def split_and_embed(text, openai_api_key):
    text_splitter = RecursiveCharacterTextSplitter(separators=["\n\n", "\n", "\t"], chunk_size=10000, chunk_overlap=3000)
    docs = text_splitter.create_documents([text])
    embeddings = OpenAIEmbeddings(openai_api_key=openai_api_key)
    vectors = embeddings.embed_documents([x.page_content for x in docs])
    return docs, vectors
```
# 步骤 3: 聚类嵌入

我们使用 KMeans 聚类对相似的内容块进行分组。在我的版本中，正如你在下面看到的，我发现 11 个聚类（clusters）对于大多数书籍来说都很有效。但你也可以根据自己的实际情况进行调整。

在这里，我们将整本书转化为块（chunks），然后再转化为嵌入（embeddings）。这些嵌入根据相似度进行分组（grouped）。对于每一组，我们都会选出最具代表性的嵌入，并将其映射回相应的文本块（text chunk）。

```python
from sklearn.cluster import KMeans
import numpy as np

def cluster_embeddings(vectors, num_clusters):
    kmeans = KMeans(n_clusters=num_clusters, random_state=42).fit(vectors)
    closest_indices = [np.argmin(np.linalg.norm(vectors - center, axis=1)) for center in kmeans.cluster_centers_]
    return sorted(closest_indices)
```
# 步骤 4: 总结具有代表性的大块内容

我们将使用 GPT-3.5 只汇总选定的数据块。

```python
from langchain.chains.summarize import load_summarize_chain
from langchain.prompts import PromptTemplate

def summarize_chunks(docs, selected_indices, openai_api_key):
    llm3_turbo = ChatOpenAI(temperature=0, openai_api_key=openai_api_key, max_tokens=1000, model='gpt-3.5-turbo-16k')
    map_prompt = """
    You are provided with a passage from a book. Your task is to produce a comprehensive summary of this passage. Ensure accuracy and avoid adding any interpretations or extra details not present in the original text. The summary should be at least three paragraphs long and fully capture the essence of the passage.
    ```{text}```
    SUMMARY:
    """
    map_prompt_template = PromptTemplate(template=map_prompt, input_variables=["text"])
    selected_docs = [docs[i] for i in selected_indices]
    summary_list = []

    for doc in selected_docs:
        chunk_summary = load_summarize_chain(llm=llm3_turbo, chain_type="stuff", prompt=map_prompt_template).run([doc])
        summary_list.append(chunk_summary)
    
    return "\n".join(summary_list)
```

# 步骤 5: 创建最终摘要

我们使用 GPT-4 将各个摘要合并成一个最终的、有内涵的摘要。

```python
from langchain.schema import Document
from langchain.chat_models import ChatOpenAI

def create_final_summary(summaries, openai_api_key):
    llm4 = ChatOpenAI(temperature=0, openai_api_key=openai_api_key, max_tokens=3000, model='gpt-4', request_timeout=120)
    combine_prompt = """
    You are given a series of summarized sections from a book. Your task is to weave these summaries into a single, cohesive, and verbose summary. The reader should be able to understand the main events or points of the book from your summary. Ensure you retain the accuracy of the content and present it in a clear and engaging manner.
    ```{text}```
    COHESIVE SUMMARY:
    """
    combine_prompt_template = PromptTemplate(template=combine_prompt, input_variables=["text"])
    reduce_chain = load_summarize_chain(llm=llm4, chain_type="stuff", prompt=combine_prompt_template)
    final_summary = reduce_chain.run([Document(page_content=summaries)])
    return final_summary
```

# 将一切汇集在一起

现在，我们将所有步骤合并到一个函数中，该函数接收上传的文件并生成摘要。

```python
# ... (previous code for imports and functions)

def generate_summary(uploaded_file, openai_api_key, num_clusters=11, verbose=False):
    file_extension = os.path.splitext(uploaded_file.name)[1].lower()
    text = load_book(uploaded_file, file_extension)
    docs, vectors = split_and_embed(text, openai_api_key)
    selected_indices = cluster_embeddings(vectors, num_clusters)
    summaries = summarize_chunks(docs, selected_indices, openai_api_key)
    final_summary = create_final_summary(summaries, openai_api_key)
    return final_summary
```

# 测试摘要

最后，我们可以用一个图书文件来测试我们的摘要。

```python
# Testing the summarizer
if __name__ == '__main__':
    load_dotenv()
    openai_api_key = os.getenv('OPENAI_API_KEY')
    book_path = "path_to_your_book.epub"
    with open(book_path, 'rb') as uploaded_file:
        summary = generate_summary(uploaded_file, openai_api_key, verbose=True)
        print(summary)
```

# 总结

该工具可帮助您快速了解任何书籍的要点。我们采用的方法不仅便宜，而且适用于任何篇幅的书籍。

请记住，摘要的质量取决于聚类(clustering )和摘要提示(summarization prompts)，因此可以根据自己的需要随意调整。

你可以在这里试用 Streamlit 应用程序并查看摘要器： [GPT 摘要应用程序](https://gptsummarizer.streamlit.app/)。

希望本文对你有所帮助。如果您有任何问题或反馈，请留言或联系我们。

祝您编码愉快！:)