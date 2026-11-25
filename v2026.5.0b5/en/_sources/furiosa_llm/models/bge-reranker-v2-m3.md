(furiosa-llm-model-bge-reranker-v2-m3)=

# BGE Reranker v2 M3

[BGE Reranker v2 M3](https://huggingface.co/BAAI/bge-reranker-v2-m3) is a
multilingual cross-encoder reranking model developed by BAAI on the
XLM-RoBERTa encoder architecture. Given a query and a set of candidate
documents, it produces relevance scores used to reorder retrieval results — a
common second stage in retrieval-augmented generation (RAG) and search
pipelines.

FuriosaAI publishes the upstream model under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as FlagEmbedding, Sentence Transformers, and
Transformers); for usage with those, see the upstream model card linked below.

For the related BGE embedding model see {doc}`bge-m3`; for another reranker
family see {doc}`qwen3-reranker`.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/bge-reranker-v2-m3`](https://huggingface.co/furiosa-ai/bge-reranker-v2-m3) | None (FP32 weights; BF16 FXB) | 1 | Multilingual cross-encoder reranker |

- **Architecture:** XLM-RoBERTa (dense encoder), `XLMRobertaForSequenceClassification`
- **Task:** Reranking
- **Input / Output:** Text (query-document pairs) / Relevance score
- **Quantization:** No lower-bit quantization is applied. The upstream checkpoint stores FP32 weights; Furiosa-LLM downcasts them to BF16 for W16A16 execution by the FXB.

## Usage

To run this model with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use the model either online through the OpenAI-compatible server or
offline through the Furiosa-LLM Python API.

### Launch the server

Serve the model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/bge-reranker-v2-m3
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

The server exposes a `/v1/rerank` endpoint compatible with the Cohere and Jina
rerank APIs. Send a query and candidate documents with `curl`; the server
returns the documents ordered by their sigmoid-normalized `relevance_score`:

```sh
curl http://localhost:8000/v1/rerank \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/bge-reranker-v2-m3",
    "query": "What is deep learning?",
    "documents": [
        "Deep learning is a subset of machine learning using neural networks.",
        "Python is a popular programming language for data science.",
        "Neural networks are inspired by biological neural networks."
    ]
    }' \
    | python -m json.tool
```

You can pass `top_n` to keep only the most relevant documents. To score
query-document pairs directly instead of reranking, the server also exposes a
`/v1/score` endpoint.

### Advanced Usage

For offline use, load the model with the `LLM` constructor (the FXB shipped in
the repo is discovered automatically) and call `score` with a query and the
candidate documents to obtain relevance scores:

```python
from furiosa_llm import LLM

query = "What is deep learning?"
documents = [
    "Deep learning is a subset of machine learning using neural networks.",
    "Python is a popular programming language for data science.",
]

with LLM("furiosa-ai/bge-reranker-v2-m3") as llm:
    outputs = llm.score(query, documents)
    for document, output in zip(documents, outputs, strict=True):
        print(f"score={output.outputs.score:.4f}  {document}")
```

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Rerank and Score APIs
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* [`BAAI/bge-reranker-v2-m3`](https://huggingface.co/BAAI/bge-reranker-v2-m3) — upstream model card
