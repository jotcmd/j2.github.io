---
title: 用写入qdrant向量数据库，来快速对比llama_index langchain
date: 2024-04-25 21:45:50
tags:
---

## 背景

挺喜欢qdrant向量数据库，每天都自动的会写入一些东西，之前用的llama_index，但api升级之后，完全不兼容了，终于完成了重写。记录一下。

用新版llama_index重写的同时，也用最新的langchain实现了一遍，刚好对比一下。

先说结论，这时候我更喜欢langchain的api设计了。

## 先看llama_index

设置openai的env变量

```python
import os,openai
os.environ["OPENAI_API_BASE"] = "https://xxx.com/v1"
os.environ["OPENAI_API_KEY"] = "sk-xxx"
openai.api_key = os.environ["OPENAI_API_KEY"]
openai.api_base = os.environ["OPENAI_API_BASE"] 
```

设置日志级别debug，这样可以看到openai的具体入参、知道什么字段参与了embeddings向量化。

```python
import logging,sys
logging.basicConfig(stream=sys.stdout, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.DEBUG)
```

还是需先初始化原生client

```python
from qdrant_client import QdrantClient
client = QdrantClient(url="http://xxx:6333", api_key="xxx", timeout=30)
```

构造为llama_index体系的对象

```python
from llama_index.vector_stores.qdrant import QdrantVectorStore
vector_store=QdrantVectorStore(client=client, collection_name="xxx")

from llama_index.core.storage.storage_context import StorageContext
storage_context = StorageContext.from_defaults(vector_store=vector_store)
```

开始构造用于embeddings的对象，text_template是为了让extra_info不参与embeddings

```python
from llama_index.core.schema import Document
text_for_embedding='hello1-text-nice1'
json_for_metadata={"a1":"v1","a2":"v2"}
documents = [Document(text=text_for_embedding,extra_info={"content":text_for_embedding,"metadata":json_for_metadata},text_template='{content}')]
```

开始构造索引

```python
from llama_index.core.indices.vector_store import VectorStoreIndex
index = VectorStoreIndex.from_documents(documents, storage_context=storage_context)
```

实际写入到qdrant中的对象json

```json
  {
    "id": "d2766b26-1358-48e3-a069-661412429424",
    "payload": {
      "_node_content": "{\"id_\": \"d2766b26-1358-48e3-a069-661412429424\", \"embedding\": null, \"metadata\": {\"content\": \"hello1-text-nice1\", \"metadata\": {\"a1\": \"v1\", \"a2\": \"v2\"}}, \"excluded_embed_metadata_keys\": [], \"excluded_llm_metadata_keys\": [], \"relationships\": {\"1\": {\"node_id\": \"813ad2df-9c3b-4417-8375-6ca755f10adf\", \"node_type\": \"4\", \"metadata\": {\"content\": \"hello1-text-nice1\", \"metadata\": {\"a1\": \"v1\", \"a2\": \"v2\"}}, \"hash\": \"d4817e367ecf40e9cc146f9cff6217e369fcbc61a9a1b20286419598f6db13ba\", \"class_name\": \"RelatedNodeInfo\"}}, \"text\": \"hello1-text-nice1\", \"start_char_idx\": 0, \"end_char_idx\": 17, \"text_template\": \"{content}\", \"metadata_template\": \"{key}: {value}\", \"metadata_seperator\": \"\\n\", \"class_name\": \"TextNode\"}",
      "_node_type": "TextNode",
      "content": "hello1-text-nice1",
      "doc_id": "813ad2df-9c3b-4417-8375-6ca755f10adf",
      "document_id": "813ad2df-9c3b-4417-8375-6ca755f10adf",
      "metadata": {
        "a1": "v1",
        "a2": "v2"
      },
      "ref_doc_id": "813ad2df-9c3b-4417-8375-6ca755f10adf"
    },
    "vector": null
  },
```

一大坨`_node_content`可能是为了后面能反序列化回来，但真的有点多。。


## 再来看langchain

跳过一样的openai和logging的设置部分。

langchain这里api涉及不大一样，偏好的是直接loading到向量数据库中

```python
from langchain.docstore.document import Document
text = "这是一个示例文档的内容。它包含了一些文本。"
metadata = {"source": "example", "author": "John Doe"}
document = Document(page_content=text, metadata=metadata)


from langchain_community.vectorstores import Qdrant
from langchain_openai import OpenAIEmbeddings
qdrant = Qdrant.from_documents(
    [document],
    OpenAIEmbeddings(),
    url="http://xxx.com:6333",
    prefer_grpc=False,
    api_key="xxx",
    collection_name="xxx",
    timeout=30
)
```

嗯这样就结束了。。这个是结果。

```json
  {
    "id": "12be48a9-d8ce-45ad-9ce5-55b7c6b814ee",
    "payload": {
      "metadata": {
        "author": "John Doe",
        "source": "example"
      },
      "page_content": "这是一个示例文档的内容。它包含了一些文本。"
    },
    "vector": null
  },
```

确实干净很多，但我还是要修正一下page_content的位置，希望和前面llama_index一样

```python
from langchain_community.vectorstores import Qdrant
from langchain_openai import OpenAIEmbeddings
qdrant = Qdrant.from_documents(
    [document],
    OpenAIEmbeddings(),
    url="http://xxx.com:6333",
    prefer_grpc=False,
    api_key="xxx",
    collection_name="xxx",
    timeout=30,
    content_payload_key="content",
    metadata_payload_key="metadata",
)
```

搞定！感觉脡不错的，高效、直接，没那么绕了。
llama_index的绕，可能是某种优雅的设计模式吧，但对应理解成本也高。


## 换一种langchain写法

为啥要换一种，因为发现上面有点一波流，但我还要继续不断往后insert，不想每次重新初始化qdrant对象。。

然而找了一会、也没找到合适的办法。

看了源码，刚好有个类似llama_index的、基于原生client的方法。

```python
from qdrant_client import QdrantClient
client = QdrantClient(url="http://xxx.com:6333", api_key="xxx", timeout=30)

from langchain_community.vectorstores import Qdrant
from langchain_openai import OpenAIEmbeddings
qdrant = Qdrant(client, "xxx", OpenAIEmbeddings(),content_payload_key="content")
```

然后、查阅源码、试了几试成功了，是这么写入文档的。

```
qdrant.add_documents(
    [Document(page_content="hello3", metadata={"a1":"v1"})],
)
```

很简洁，点赞langchain。

最后向量库中实际存储的内容、也是干净如预期的。

```json
  {
    "id": "9d678096-8bb9-4f9e-b651-4a8553622fc4",
    "payload": {
      "content": "hello3",
      "metadata": {
        "a1": "v1"
      }
    },
    "vector": null
  },
```

回过头来，可能前面第一种的写法，用`qdrant.add_documents`也能同样解决诉求问题。

马上试了试，确实如此。

但理解上，还是第二种方式合适我。

以上。