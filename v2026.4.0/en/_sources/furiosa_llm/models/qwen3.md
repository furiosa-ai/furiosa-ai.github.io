(furiosa-llm-model-qwen3)=

# Qwen3 (dense)

[Qwen3](https://huggingface.co/Qwen) is Alibaba's latest generation of dense,
auto-regressive transformer language models with grouped-query attention. Their
hallmark is seamless switching between a **thinking mode** — emitting a chain of
thought before the final answer for complex reasoning, math, and coding — and a
**non-thinking mode** for efficient general dialogue, within a single model.
They also offer strong tool-calling and agent capabilities and multilingual
support.

Furiosa-LLM runs the Qwen3 dense models in **FP8** (static FP8 weights with
dynamic FP8 activation quantization; the KV cache stays in 16-bit precision).
FuriosaAI publishes pre-compiled FP8 builds under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model cards linked below.

This page covers the dense Qwen3 chat models. For the Mixture-of-Experts
variants see {doc}`qwen3-moe`, and for the embedding, reranking, and
vision-language members of the family see {doc}`qwen3-embedding`,
{doc}`qwen3-reranker`, and {doc}`qwen3-vl`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-4B-FP8`](https://huggingface.co/furiosa-ai/Qwen3-4B-FP8) | FP8 | 1 | 4B params; hybrid thinking / non-thinking |
| [`furiosa-ai/Qwen3-8B-FP8`](https://huggingface.co/furiosa-ai/Qwen3-8B-FP8) | FP8 | 1 | 8.2B params; hybrid thinking / non-thinking |
| [`furiosa-ai/Qwen3-32B-FP8`](https://huggingface.co/furiosa-ai/Qwen3-32B-FP8) | FP8 | 4 | 32B params; hybrid thinking / non-thinking |

- **Architecture:** Qwen3 (dense), `Qwen3ForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** Weights are quantized to **FP8** (static, fine-grained, block size 128), and activations use **dynamic FP8 quantization** at runtime (per-token / per-block). The KV cache stays in 16-bit precision.

## Usage

To run these models with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve a model is to pass its `furiosa-ai/<repo>` identifier.
Qwen3 dense models are hybrid reasoning models, so add `--reasoning-parser qwen3`
to have the thinking content parsed into a separate field (see
[Reasoning](#reasoning) below):

```sh
# Qwen3-8B-FP8 — single RNGD card
furiosa-llm serve furiosa-ai/Qwen3-8B-FP8 --reasoning-parser qwen3
```

```sh
# Qwen3-32B-FP8 — four RNGD cards
furiosa-llm serve furiosa-ai/Qwen3-32B-FP8 --reasoning-parser qwen3
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Launch the server with tool calling

To enable tool (function) calling, start the server with the `hermes` tool-call
parser (the parser used by the Qwen3 series):

```sh
furiosa-llm serve furiosa-ai/Qwen3-8B-FP8 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser hermes
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a request with `curl`
(replace the model id with the variant you launched):

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-8B-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Reasoning

With `--reasoning-parser qwen3`, the thinking content is returned separately from
the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

You can switch off thinking for a request by adding `/no_think` to the prompt
(and back on with `/think`).

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/Qwen3-8B-FP8",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
)

print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

> **Note:** The `reasoning` field is not part of the OpenAI API specification, but
> it is the convention OpenAI recommends for returning the chain-of-thought (CoT)
> in Chat Completions-compatible APIs. The OpenAI Agents SDK uses `reasoning` as
> its primary property for the CoT, and many LLM serving frameworks (such as vLLM)
> follow the same convention. It appears only in responses that contain reasoning
> content; accessing it on a response without reasoning content raises an
> `AttributeError`.

### Tool calling

With the server launched using `--enable-auto-tool-choice --tool-call-parser hermes`,
you can pass `tools` and let the model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model cards: [Qwen/Qwen3-8B-FP8](https://huggingface.co/Qwen/Qwen3-8B-FP8), [Qwen/Qwen3-32B-FP8](https://huggingface.co/Qwen/Qwen3-32B-FP8)
