(furiosa-llm-model-exaone-4-0)=

# EXAONE 4.0

[EXAONE 4.0](https://huggingface.co/LGAI-EXAONE) is LG AI Research's bilingual
(English / Korean) series of auto-regressive dense transformers that unify a
non-reasoning mode for general instruction following with a reasoning mode for
harder problems, and carry native support for tool calling and agentic use.

FuriosaAI publishes pre-compiled builds of the EXAONE 4.0 models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/EXAONE-4.0-32B-FP8`](https://huggingface.co/furiosa-ai/EXAONE-4.0-32B-FP8) | FP8 | 4 | 32B dense; reasoning / non-reasoning |

- **Architecture:** EXAONE 4.0 (dense), `Exaone4ForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** Weights are quantized to **FP8** (static), following the upstream FP8 release, and activations use **dynamic FP8 quantization** at runtime (per-token / per-block). The KV cache stays in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8
```

To parse the model's reasoning into a separate field, start the server with the
`exaone4` reasoning parser (see [Reasoning](#reasoning) below):

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 \
  --reasoning-parser exaone4
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
parser (the parser used for the EXAONE-4.0 format):

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 \
  --reasoning-parser exaone4 \
  --enable-auto-tool-choice \
  --tool-call-parser hermes
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a request with `curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/EXAONE-4.0-32B-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Reasoning

When launched with `--reasoning-parser exaone4`, the model returns its reasoning
separately from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/EXAONE-4.0-32B-FP8",
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
* Upstream model card: [LGAI-EXAONE/EXAONE-4.0-32B-FP8](https://huggingface.co/LGAI-EXAONE/EXAONE-4.0-32B-FP8)
