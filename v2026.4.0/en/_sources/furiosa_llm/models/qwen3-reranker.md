(furiosa-llm-model-qwen3-reranker)=

# Qwen3-Reranker

The [Qwen3-Reranker](https://huggingface.co/Qwen) series is a family of reranking
models built on the Qwen3 dense transformer backbone. Given a query and a set of
candidate documents, they produce relevance scores used to reorder retrieval
results — a common second stage in retrieval-augmented generation (RAG) and
search pipelines.

FuriosaAI publishes the Qwen3-Reranker models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as Sentence Transformers, vLLM, and
Transformers); for usage with those, see the upstream model cards linked below.

For the related embedding model see {doc}`qwen3-embedding`; for the dense Qwen3
chat models see {doc}`qwen3`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-Reranker-0.6B`](https://huggingface.co/furiosa-ai/Qwen3-Reranker-0.6B) | None (BF16) | 1 | 0.6B reranker |
| [`furiosa-ai/Qwen3-Reranker-4B`](https://huggingface.co/furiosa-ai/Qwen3-Reranker-4B) | None (BF16) | 1 | 4B reranker |
| [`furiosa-ai/Qwen3-Reranker-8B`](https://huggingface.co/furiosa-ai/Qwen3-Reranker-8B) | None (BF16) | 1 | 8B reranker |

- **Architecture:** Qwen3 (dense), `Qwen3ForSequenceClassification`
- **Task:** Reranking
- **Input / Output:** Text (query-document pairs) / Relevance score
- **Quantization:** No quantization — the models run in their native BF16 precision.

## Usage

To run these models with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use a model either online through the OpenAI-compatible server or offline
through the Furiosa-LLM Python API. Select a variant tab below; the choice is
kept in sync across all the examples on this page.

### Launch the server

Serve a model by passing its `furiosa-ai/<repo>` identifier:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Reranker-0.6B
```
:::
:::{tab-item} 4B
:sync: v4
```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Reranker-4B
```
:::
:::{tab-item} 8B
:sync: v8
```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Reranker-8B
```
:::
::::

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Rerank documents

The server exposes a `/v1/rerank` endpoint (compatible with the Cohere/Jina
rerank API, also used by vLLM). Send a query and the candidate documents with
`curl`; the server returns the documents reordered by `relevance_score`:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```sh
curl http://localhost:8000/v1/rerank \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Reranker-0.6B",
    "query": "What is deep learning?",
    "documents": [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
        "Neural networks are inspired by biological neural networks."
    ]
    }' \
    | python -m json.tool
```
:::
:::{tab-item} 4B
:sync: v4
```sh
curl http://localhost:8000/v1/rerank \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Reranker-4B",
    "query": "What is deep learning?",
    "documents": [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
        "Neural networks are inspired by biological neural networks."
    ]
    }' \
    | python -m json.tool
```
:::
:::{tab-item} 8B
:sync: v8
```sh
curl http://localhost:8000/v1/rerank \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Reranker-8B",
    "query": "What is deep learning?",
    "documents": [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
        "Neural networks are inspired by biological neural networks."
    ]
    }' \
    | python -m json.tool
```
:::
::::

You can do the same from Python with the `requests` library, and pass `top_n` to
keep only the most relevant documents:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```python
import requests

response = requests.post(
    "http://localhost:8000/v1/rerank",
    json={
        "model": "furiosa-ai/Qwen3-Reranker-0.6B",
        "query": "What is deep learning?",
        "documents": [
            "Deep learning is a subset of machine learning using neural networks.",
            "Python is a popular programming language for data science.",
            "Neural networks are inspired by biological neural networks.",
        ],
        "top_n": 2,
    },
)

for result in response.json()["results"]:
    print(f"score={result['relevance_score']:.4f}  {result['document']['text']}")
```
:::
:::{tab-item} 4B
:sync: v4
```python
import requests

response = requests.post(
    "http://localhost:8000/v1/rerank",
    json={
        "model": "furiosa-ai/Qwen3-Reranker-4B",
        "query": "What is deep learning?",
        "documents": [
            "Deep learning is a subset of machine learning using neural networks.",
            "Python is a popular programming language for data science.",
            "Neural networks are inspired by biological neural networks.",
        ],
        "top_n": 2,
    },
)

for result in response.json()["results"]:
    print(f"score={result['relevance_score']:.4f}  {result['document']['text']}")
```
:::
:::{tab-item} 8B
:sync: v8
```python
import requests

response = requests.post(
    "http://localhost:8000/v1/rerank",
    json={
        "model": "furiosa-ai/Qwen3-Reranker-8B",
        "query": "What is deep learning?",
        "documents": [
            "Deep learning is a subset of machine learning using neural networks.",
            "Python is a popular programming language for data science.",
            "Neural networks are inspired by biological neural networks.",
        ],
        "top_n": 2,
    },
)

for result in response.json()["results"]:
    print(f"score={result['relevance_score']:.4f}  {result['document']['text']}")
```
:::
::::

To score query-document pairs directly instead of reranking, the server also
exposes a `/v1/score` endpoint.

### Python API

For offline use, load the model with the `LLM` constructor (the FXB shipped in
the repo is discovered automatically) and call `score` with a query and the
candidate documents to obtain relevance scores:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Reranker-0.6B") as llm:
    query = "What is deep learning?"
    documents = [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
    ]
    outputs = llm.score(query, documents)
    for document, output in zip(documents, outputs):
        print(f"score={output.outputs.score:.4f}  {document}")
```
:::
:::{tab-item} 4B
:sync: v4
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Reranker-4B") as llm:
    query = "What is deep learning?"
    documents = [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
    ]
    outputs = llm.score(query, documents)
    for document, output in zip(documents, outputs):
        print(f"score={output.outputs.score:.4f}  {document}")
```
:::
:::{tab-item} 8B
:sync: v8
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Reranker-8B") as llm:
    query = "What is deep learning?"
    documents = [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
    ]
    outputs = llm.score(query, documents)
    for document, output in zip(documents, outputs):
        print(f"score={output.outputs.score:.4f}  {document}")
```
:::
::::

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Rerank and Score APIs
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* Upstream model cards: [Qwen/Qwen3-Reranker-0.6B](https://huggingface.co/Qwen/Qwen3-Reranker-0.6B), [Qwen/Qwen3-Reranker-4B](https://huggingface.co/Qwen/Qwen3-Reranker-4B), [Qwen/Qwen3-Reranker-8B](https://huggingface.co/Qwen/Qwen3-Reranker-8B)
