(furiosa-llm-model-e5-mistral)=

# E5-Mistral

[E5-Mistral-7B-Instruct](https://huggingface.co/intfloat/e5-mistral-7b-instruct)
is an instruction-aware text-embedding model built by fine-tuning the dense
Mistral-7B-v0.1 decoder backbone. It produces 4,096-dimensional dense
embeddings through last-token pooling and L2 normalization for retrieval and
semantic similarity tasks. Although it was fine-tuned on multilingual data,
the upstream model card recommends it primarily for English.

FuriosaAI publishes E5-Mistral-7B-Instruct under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as Sentence Transformers and Transformers);
for usage with those, see the upstream model card linked below.

For other instruction-aware embedding models, see {doc}`harrier-oss-v1` and
{doc}`qwen3-embedding`.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/e5-mistral-7b-instruct`](https://huggingface.co/furiosa-ai/e5-mistral-7b-instruct) | None (FP16) | 1 | 7B instruction-aware text embedding |

- **Architecture:** Mistral (dense decoder backbone), `MistralModel`
- **Task:** Embedding
- **Input / Output:** Text / Dense embeddings (vector)
- **Quantization:** No quantization — the model uses its upstream FP16 weights.

## Usage

To run this model with Furiosa-LLM, follow the examples below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
You can use the model either online through the OpenAI-compatible server or
offline through the Furiosa-LLM Python API.

### Launch the server

Serve the model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/e5-mistral-7b-instruct
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

The server exposes an OpenAI-compatible `/v1/embeddings` endpoint. E5-Mistral
is instruction-aware: prepend a one-sentence task description to each query in
the `Instruct: ...\nQuery: ...` format, and do not add the instruction to
documents. For more details, see the
[upstream model card](https://huggingface.co/intfloat/e5-mistral-7b-instruct).
Request embeddings with `curl`:

```sh
curl http://localhost:8000/v1/embeddings \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/e5-mistral-7b-instruct",
    "input": [
      "Instruct: Given a web search query, retrieve relevant passages that answer the query\nQuery: summit define",
      "Definition of summit: the highest point of a mountain."
    ]
    }' \
    | python -m json.tool
```

Because the endpoint is OpenAI-compatible, you can also use the OpenAI Python
client:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

query = (
    "Instruct: Given a web search query, retrieve relevant passages that answer the query\n"
    "Query: summit define"
)
document = "Definition of summit: the highest point of a mountain."

response = client.embeddings.create(
    model="furiosa-ai/e5-mistral-7b-instruct",
    input=[query, document],
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

query = (
    "Instruct: Given a web search query, retrieve relevant passages that answer the query\n"
    "Query: summit define"
)
document = "Definition of summit: the highest point of a mountain."

with LLM("furiosa-ai/e5-mistral-7b-instruct") as llm:
    outputs = llm.embed([query, document])
    embeddings = [output.outputs.embedding for output in outputs]

similarity = sum(a * b for a, b in zip(*embeddings, strict=True))
print(f"Cosine similarity: {similarity:.4f}")
```

## Learn more

* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference, including the Embeddings API
* [Furiosa-LLM](https://developer.furiosa.ai/latest/en/furiosa_llm/intro.html) — Furiosa-LLM documentation and API reference
* [`intfloat/e5-mistral-7b-instruct`](https://huggingface.co/intfloat/e5-mistral-7b-instruct) — upstream model card
