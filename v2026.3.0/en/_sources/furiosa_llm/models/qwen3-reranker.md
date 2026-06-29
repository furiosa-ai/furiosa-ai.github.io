(furiosa-llm-model-qwen3-reranker)=

# Qwen3-Reranker

The [Qwen3-Reranker](https://huggingface.co/Qwen) series are reranking models
built on the Qwen3 dense transformer backbone. Given a query and a set of
candidate documents, they produce relevance scores used to reorder retrieval
results — a common second stage in retrieval-augmented generation (RAG) and
search pipelines.

FuriosaAI publishes pre-compiled builds of the Qwen3-Reranker models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM.

For the related embedding model see {doc}`qwen3-embedding`; for the dense Qwen3
chat models see {doc}`qwen3`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-Reranker-8B`](https://huggingface.co/furiosa-ai/Qwen3-Reranker-8B) | None (16-bit) | 1 | 8B reranker |

- **Architecture:** Qwen3 (dense), `Qwen3ForSequenceClassification`
- **Task:** Reranking
- **Input / Output:** Text (query-document pairs) / Relevance score
- **Quantization:** No quantization — the model runs in its native 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use the model either offline through the Furiosa-LLM Python API or online
through the OpenAI-compatible server.

### Python API

Load the artifact and call `score` with query-document pairs to obtain relevance
scores:

```python
from furiosa_llm import LLM

llm = LLM.from_artifacts("furiosa-ai/Qwen3-Reranker-8B")
scores = llm.score([("query", "document1"), ("query", "document2")])
```

### Online server

The server exposes a `/v1/rerank` endpoint (compatible with the Cohere/Jina
rerank API, also used by vLLM). Launch it the same way as any other model:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Reranker-8B
```

Once it is ready, send a query and the candidate documents with `curl`; the
server returns the documents reordered by `relevance_score`:

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

You can do the same from Python with the `requests` library, and pass `top_n` to
keep only the most relevant documents:

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

To score query-document pairs directly instead of reranking, the server also
exposes a `/v1/score` endpoint.

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Rerank and Score APIs
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* Upstream model card: [Qwen/Qwen3-Reranker-8B](https://huggingface.co/Qwen/Qwen3-Reranker-8B)
