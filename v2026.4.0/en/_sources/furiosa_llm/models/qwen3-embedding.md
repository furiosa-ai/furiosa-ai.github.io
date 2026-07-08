(furiosa-llm-model-qwen3-embedding)=

# Qwen3-Embedding

The [Qwen3-Embedding](https://huggingface.co/Qwen) series is a family of
text-embedding models built on the Qwen3 dense transformer backbone. They map
text into dense vector representations for semantic search, retrieval, and
similarity matching, with strong multilingual coverage.

FuriosaAI publishes the Qwen3-Embedding models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as Sentence Transformers, vLLM, and
Transformers); for usage with those, see the upstream model cards linked below.

For the related reranking model see {doc}`qwen3-reranker`; for the dense Qwen3
chat models see {doc}`qwen3`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-Embedding-0.6B`](https://huggingface.co/furiosa-ai/Qwen3-Embedding-0.6B) | None (BF16) | 1 | 0.6B text embedding |
| [`furiosa-ai/Qwen3-Embedding-4B`](https://huggingface.co/furiosa-ai/Qwen3-Embedding-4B) | None (BF16) | 1 | 4B text embedding |
| [`furiosa-ai/Qwen3-Embedding-8B`](https://huggingface.co/furiosa-ai/Qwen3-Embedding-8B) | None (BF16) | 1 | 8B text embedding |

- **Architecture:** Qwen3 (dense), `Qwen3Model`
- **Task:** Embedding
- **Input / Output:** Text / Embeddings (vector)
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
furiosa-llm serve furiosa-ai/Qwen3-Embedding-0.6B
```
:::
:::{tab-item} 4B
:sync: v4
```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Embedding-4B
```
:::
:::{tab-item} 8B
:sync: v8
```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-Embedding-8B
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

### Query the server

The server exposes an OpenAI-compatible `/v1/embeddings` endpoint. Request
embeddings with `curl`:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Embedding-0.6B",
    "input": ["Hello, world!", "How are you?"]
    }' \
    | python -m json.tool
```
:::
:::{tab-item} 4B
:sync: v4
```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Embedding-4B",
    "input": ["Hello, world!", "How are you?"]
    }' \
    | python -m json.tool
```
:::
:::{tab-item} 8B
:sync: v8
```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-Embedding-8B",
    "input": ["Hello, world!", "How are you?"]
    }' \
    | python -m json.tool
```
:::
::::

Because the endpoint is OpenAI-compatible, you can also use the OpenAI Python
client:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.embeddings.create(
    model="furiosa-ai/Qwen3-Embedding-0.6B",
    input=["Hello, world!", "How are you?"],
)

for data in response.data:
    print(f"Index {data.index}: {len(data.embedding)} dimensions")
```
:::
:::{tab-item} 4B
:sync: v4
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.embeddings.create(
    model="furiosa-ai/Qwen3-Embedding-4B",
    input=["Hello, world!", "How are you?"],
)

for data in response.data:
    print(f"Index {data.index}: {len(data.embedding)} dimensions")
```
:::
:::{tab-item} 8B
:sync: v8
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
:::
::::

### Python API

For offline use, load the model with the `LLM` constructor (the FXB shipped in
the repo is discovered automatically) and call `embed` to obtain dense vectors:

::::{tab-set}
:::{tab-item} 0.6B
:sync: v06
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Embedding-0.6B") as llm:
    outputs = llm.embed(["Hello, world!", "How are you?"])
    for output in outputs:
        print(f"{len(output.outputs.embedding)} dimensions")
```
:::
:::{tab-item} 4B
:sync: v4
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Embedding-4B") as llm:
    outputs = llm.embed(["Hello, world!", "How are you?"])
    for output in outputs:
        print(f"{len(output.outputs.embedding)} dimensions")
```
:::
:::{tab-item} 8B
:sync: v8
```python
from furiosa_llm import LLM

with LLM("furiosa-ai/Qwen3-Embedding-8B") as llm:
    outputs = llm.embed(["Hello, world!", "How are you?"])
    for output in outputs:
        print(f"{len(output.outputs.embedding)} dimensions")
```
:::
::::

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Embeddings API
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* Upstream model cards: [Qwen/Qwen3-Embedding-0.6B](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B), [Qwen/Qwen3-Embedding-4B](https://huggingface.co/Qwen/Qwen3-Embedding-4B), [Qwen/Qwen3-Embedding-8B](https://huggingface.co/Qwen/Qwen3-Embedding-8B)
