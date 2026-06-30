(furiosa-llm-model-qwen3-moe)=

# Qwen3-MoE

The Qwen3 Mixture-of-Experts (MoE) models are auto-regressive **MoE**
transformers with 30.5B total parameters, of which about 3.3B are activated per
token. They offer strong instruction following, reasoning over text,
multilingual coverage, and tool usage. They come in the original hybrid
thinking / non-thinking release as well as updated instruct, thinking, and
coding-specialized editions.

Furiosa-LLM runs the Qwen3-MoE models in **FP8** (static FP8 weights with dynamic
FP8 activation quantization; the KV cache stays in 16-bit precision). FuriosaAI
publishes pre-compiled FP8 builds under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model cards linked below.

For the dense Qwen3 chat models see {doc}`qwen3`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-30B-A3B-FP8`](https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-FP8) | FP8 | 4 | Original release; hybrid thinking / non-thinking |
| [`furiosa-ai/Qwen3-30B-A3B-Instruct-2507-FP8`](https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-Instruct-2507-FP8) | FP8 | 4 | Updated (2507) instruct; non-thinking only |
| [`furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8`](https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8) | FP8 | 4 | Updated (2507) thinking; always reasons |
| [`furiosa-ai/Qwen3-Coder-30B-A3B-Instruct-FP8`](https://huggingface.co/furiosa-ai/Qwen3-Coder-30B-A3B-Instruct-FP8) | FP8 | 4 | Agentic coding; non-thinking only |

- **Architecture:** Qwen3-MoE (Mixture-of-Experts), `Qwen3MoeForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** Weights are quantized to **FP8** (static), following the upstream FP8 release, and activations use **dynamic FP8 quantization** at runtime (per-token / per-block). The KV cache stays in 16-bit precision.

## Usage

To run these models with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Pass the model's `furiosa-ai/<repo>` identifier. Each variant runs on four RNGD
cards (tensor-parallel size 32 PEs).

The **Instruct** and **Coder** variants are non-thinking and need no reasoning
parser:

```sh
furiosa-llm serve furiosa-ai/Qwen3-30B-A3B-Instruct-2507-FP8
```

```sh
furiosa-llm serve furiosa-ai/Qwen3-Coder-30B-A3B-Instruct-FP8
```

The original **`Qwen3-30B-A3B-FP8`** is a hybrid model that reasons by default;
add `--reasoning-parser qwen3` to have the reasoning returned in a separate field
(see [Reasoning](#reasoning) below):

```sh
furiosa-llm serve furiosa-ai/Qwen3-30B-A3B-FP8 \
  --reasoning-parser qwen3
```

The **Thinking** variant always produces a chain of thought before its final
answer; add `--reasoning-parser qwen3` to have the reasoning returned in a
separate field (see [Reasoning](#reasoning) below):

```sh
furiosa-llm serve furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8 \
  --reasoning-parser qwen3
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
parser (the parser used by the Qwen3 series). For the Thinking variant, keep the
reasoning parser as well:

```sh
furiosa-llm serve furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8 \
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
    "model": "furiosa-ai/Qwen3-30B-A3B-Instruct-2507-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Reasoning

The **Thinking** variant returns its reasoning separately from the final answer
when launched with `--reasoning-parser qwen3`:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8",
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
* Upstream model cards: [Qwen/Qwen3-30B-A3B-FP8](https://huggingface.co/Qwen/Qwen3-30B-A3B-FP8), [Qwen/Qwen3-30B-A3B-Instruct-2507-FP8](https://huggingface.co/Qwen/Qwen3-30B-A3B-Instruct-2507-FP8), [Qwen/Qwen3-30B-A3B-Thinking-2507-FP8](https://huggingface.co/Qwen/Qwen3-30B-A3B-Thinking-2507-FP8), [Qwen/Qwen3-Coder-30B-A3B-Instruct-FP8](https://huggingface.co/Qwen/Qwen3-Coder-30B-A3B-Instruct-FP8)
