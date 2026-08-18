(furiosa-llm-model-exaone-4-0)=

# EXAONE 4.0

[EXAONE 4.0](https://huggingface.co/LGAI-EXAONE) is LG AI Research's multilingual
(English / Korean / Spanish) series of auto-regressive dense transformers that
unify a
non-reasoning mode for general instruction following with a reasoning mode for
harder problems, plus native support for tool calling and agentic use.

Furiosa-LLM runs EXAONE 4.0 in **FP8** (static FP8 weights with dynamic FP8
activation quantization; the KV cache stays in 16-bit precision). FuriosaAI
publishes pre-compiled FP8 builds under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

## Available Models

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

EXAONE 4.0 is a hybrid model that runs in **non-reasoning mode by default**. To
have the chain of thought returned in a separate field once reasoning is enabled,
start the server with the `exaone4` reasoning parser (see
[Advanced Usage](#advanced-usage) for how to turn reasoning on):

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 \
  --reasoning-parser exaone4
```

To also enable tool (function) calling, add the `hermes` tool-call parser (the
parser used for the EXAONE-4.0 format); keep `--reasoning-parser exaone4` so
reasoning, when enabled, is parsed into its own field:

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 \
  --reasoning-parser exaone4 \
  --enable-auto-tool-choice \
  --tool-call-parser hermes
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

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

EXAONE 4.0 runs in non-reasoning mode by default. Once reasoning is enabled (see
[Advanced Usage](#advanced-usage)) and the server is started with
`--reasoning-parser exaone4`, the chain of thought is returned separately from the
final answer:

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

By default the response carries no reasoning content, so read only
`message.content`; `message.reasoning` is populated once you enable reasoning.

> **Note:** The `reasoning` field is not part of the OpenAI API specification but
> is a widely followed convention (the OpenAI Agents SDK, vLLM, and others). It
> appears only in responses that contain reasoning content; accessing it otherwise
> raises an `AttributeError`.

### Advanced Usage

**Enabling reasoning.** EXAONE 4.0 runs in non-reasoning mode by default. Enable
reasoning for a single request by passing `enable_thinking` through
`chat_template_kwargs`; with `--reasoning-parser exaone4` the chain of thought is
then returned separately from the final answer:

```python
# Enable reasoning for a single request
response = client.chat.completions.create(
    model="furiosa-ai/EXAONE-4.0-32B-FP8",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": True}},
)

print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

To make every request reason, launch the server with `--default-chat-template-kwargs`
(a request can still override with its own `chat_template_kwargs`):

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 \
  --reasoning-parser exaone4 \
  --default-chat-template-kwargs '{"enable_thinking": true}'
```

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser hermes` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [LGAI-EXAONE/EXAONE-4.0-32B-FP8](https://huggingface.co/LGAI-EXAONE/EXAONE-4.0-32B-FP8)
