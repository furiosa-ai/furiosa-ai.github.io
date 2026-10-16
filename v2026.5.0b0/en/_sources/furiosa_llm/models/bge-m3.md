(furiosa-llm-model-bge-m3)=

# BGE-M3

[BGE-M3](https://huggingface.co/BAAI/bge-m3) is a multilingual text-embedding
model developed by BAAI on the XLM-RoBERTa encoder architecture. The upstream
model unifies dense, sparse lexical, and multi-vector retrieval. Furiosa-LLM
exposes its 1,024-dimensional, CLS-pooled dense embeddings for semantic search,
retrieval, and similarity matching.

FuriosaAI publishes a BF16 build of BGE-M3 under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The base model also runs
on other frameworks (such as FlagEmbedding, Sentence Transformers, and
Transformers); for usage with those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/bge-m3`](https://huggingface.co/furiosa-ai/bge-m3) | None (BF16) | 1 | Multilingual dense text embedding |

- **Architecture:** XLM-RoBERTa (dense encoder), `XLMRobertaModel`
- **Task:** Embedding
- **Input / Output:** Text / Dense embeddings (vector)
- **Quantization:** The FuriosaAI build uses BF16 weights without a lower-bit quantization scheme.

## Usage

To run this model with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use the model either online through the OpenAI-compatible server or
offline through the Furiosa-LLM Python API.

### Launch the server

Serve the model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/bge-m3
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

The server exposes an OpenAI-compatible `/v1/embeddings` endpoint. BGE-M3 does
not require an instruction prefix for queries. Request dense embeddings with
`curl`:

```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/bge-m3",
    "input": [
      "What is BGE-M3?",
      "BGE-M3 is a multilingual text-embedding model."
    ]
    }' \
    | python -m json.tool
```

Because the endpoint is OpenAI-compatible, you can also use the OpenAI Python
client:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.embeddings.create(
    model="furiosa-ai/bge-m3",
    input=[
        "What is BGE-M3?",
        "BGE-M3 is a multilingual text-embedding model.",
    ],
)

for data in response.data:
    print(f"Index {data.index}: {len(data.embedding)} dimensions")
```

### Advanced Usage

For offline use, load the model with the `LLM` constructor (the FXB shipped in
the repo is discovered automatically) and call `embed` to obtain L2-normalized
dense vectors. Their dot product is therefore the cosine similarity:

```python
from furiosa_llm import LLM

texts = [
    "What is BGE-M3?",
    "BGE-M3 is a multilingual text-embedding model.",
]

with LLM("furiosa-ai/bge-m3") as llm:
    outputs = llm.embed(texts)
    embeddings = [output.outputs.embedding for output in outputs]

similarity = sum(a * b for a, b in zip(*embeddings, strict=True))
print(f"Cosine similarity: {similarity:.4f}")
```

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Embeddings API
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* [`BAAI/bge-m3`](https://huggingface.co/BAAI/bge-m3) — upstream model card
