(furiosa-llm-model-qwen3-embedding)=

# Qwen3-Embedding

The [Qwen3-Embedding](https://huggingface.co/Qwen) series is a family of
text-embedding models built on the Qwen3 dense transformer backbone. They map
text into dense vector representations for semantic search, retrieval, and
similarity matching, with strong multilingual coverage.

FuriosaAI publishes pre-compiled builds of the Qwen3-Embedding models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM.

For the related reranking model see {doc}`qwen3-reranker`; for the dense Qwen3
chat models see {doc}`qwen3`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-Embedding-8B`](https://huggingface.co/furiosa-ai/Qwen3-Embedding-8B) | None (16-bit) | 1 | 8B text embedding |

- **Architecture:** Qwen3 (dense), `Qwen3Model`
- **Task:** Embedding
- **Input / Output:** Text / Embeddings (vector)
- **Quantization:** No quantization — the model runs in its native 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use the model either offline through the Furiosa-LLM Python API or online
through the OpenAI-compatible server.

### Python API

Load the artifact and call `embed` to obtain dense vectors:

```python
from furiosa_llm import LLM

llm = LLM.from_artifacts("furiosa-ai/Qwen3-Embedding-8B")
embeddings = llm.embed(["Hello, world!", "How are you?"])
```

### Online server

The server exposes an OpenAI-compatible `/v1/embeddings` endpoint. Launch it the
same way as any other model:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Embedding-8B
```

Once it is ready, request embeddings with `curl`:

```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Embedding-8B",
    "input": ["Hello, world!", "How are you?"]
    }' \
    | python -m json.tool
```

Because the endpoint is OpenAI-compatible, you can also use the OpenAI Python
client:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.embeddings.create(
    model="furiosa-ai/Qwen3-Embedding-8B",
    input=["Hello, world!", "How are you?"],
)

for data in response.data:
    print(f"Index {data.index}: {len(data.embedding)} dimensions")
```

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Embeddings API
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* Upstream model card: [Qwen/Qwen3-Embedding-8B](https://huggingface.co/Qwen/Qwen3-Embedding-8B)
