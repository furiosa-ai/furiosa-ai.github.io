(furiosa-llm-model-exaone-4-5)=

# EXAONE 4.5

[EXAONE 4.5](https://huggingface.co/LGAI-EXAONE/EXAONE-4.5-33B-FP8) is LG AI
Research's open-weight vision-language model. It integrates a dedicated vision
encoder with the dense EXAONE 4 language-model architecture to handle images
and text, and supports hybrid reasoning and tool calling.

Furiosa-LLM runs EXAONE 4.5 using the upstream **FP8** release. Selected linear
weights use static FP8 quantization with dynamic per-token FP8 activation
quantization, while the vision encoder and attention projections stay in higher
precision. FuriosaAI publishes a pre-compiled build under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
shipping a Furiosa Executable Bundle (FXB) for running the model on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/EXAONE-4.5-33B-FP8`](https://huggingface.co/furiosa-ai/EXAONE-4.5-33B-FP8) | FP8 | 4 | 33B dense vision-language model; thinking by default |

- **Architecture:** EXAONE 4.5 (dense vision-language), `Exaone4_5_ForConditionalGeneration`
- **Input / Output:** Image + Text / Text
- **Quantization:** Selected linear weights are quantized to **FP8** (static, per-channel), following the upstream FP8 release, and their activations use **dynamic per-token FP8 quantization** at runtime. The upstream configuration excludes the vision encoder, attention projections, LM head, and MTP attention projections from FP8 quantization; the KV cache stays in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

EXAONE 4.5 reasons by default and can switch thinking on and off (see
[Advanced Usage](#advanced-usage)). Launch the server with the `qwen3` reasoning
parser so its chain of thought is separated from the final answer:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/EXAONE-4.5-33B-FP8 \
  --reasoning-parser qwen3
```

To also enable tool (function) calling, add the `hermes` tool-call parser; keep
`--reasoning-parser qwen3` so thinking is still parsed into its own field:

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.5-33B-FP8 \
  --reasoning-parser qwen3 \
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

The server exposes an OpenAI-compatible API. You can send a text-only request
with `curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/EXAONE-4.5-33B-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

To ask about an image, pass an `image_url` content part in the message:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/EXAONE-4.5-33B-FP8",
    "messages": [{
        "role": "user",
        "content": [
            {"type": "image_url", "image_url": {"url": "https://github.com/LG-AI-EXAONE/EXAONE-4.5/blob/main/assets/exaone45_input1.jpg?raw=true"}},
            {"type": "text", "text": "What dish is the person preparing?"}
        ]
    }]
    }' \
    | python -m json.tool
```

The `image_url.url` field accepts a remote `http://`/`https://` URL, an inline
base64 `data:` URL, or a local `file://` path (the last requires the
`--allowed-local-media-path` flag described under [Advanced Usage](#advanced-usage)).

With `--reasoning-parser qwen3`, EXAONE 4.5 returns its reasoning separately
from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/EXAONE-4.5-33B-FP8",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
)

print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

> **Note:** The `reasoning` field is not part of the OpenAI API specification but
> is a widely followed convention (the OpenAI Agents SDK, vLLM, and others). It
> appears only in responses that contain reasoning content; accessing it otherwise
> raises an `AttributeError`.

### Advanced Usage

**Multimodal serving options.** `furiosa-llm serve` provides flags to control
multimodal behavior; requests that violate them are rejected with HTTP 400:

- `--image-limit-per-prompt N` — maximum number of images allowed per request (default: unlimited).
- `--allowed-local-media-path PATH` — allow `file://` URLs whose resolved path is under `PATH`. Local file access is disabled unless this is set.
- `--allowed-media-domains D [D ...]` — whitelist remote domains for SSRF protection. When set, only images from the listed domains are fetched.
- `--interleave-mm-strings` — keep image placeholders at their original positions when the model uses a string-format chat template.

For example, to serve local images under `/srv/media` and restrict remote
fetches to a single domain:

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.5-33B-FP8 \
  --reasoning-parser qwen3 \
  --allowed-local-media-path /srv/media \
  --allowed-media-domains cdn.example.com \
  --image-limit-per-prompt 4
```

See the [Vision-Language Models guide](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html)
for image input formats and Python client examples.

**Turning thinking off.** EXAONE 4.5 reasons by default. To turn thinking off
for a single request, pass `enable_thinking` through `chat_template_kwargs`; the
response then carries no reasoning content, so read only `message.content`:

```python
# Disable thinking for a single request
response = client.chat.completions.create(
    model="furiosa-ai/EXAONE-4.5-33B-FP8",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": False}},
)
print(response.choices[0].message.content)
```

To default every request to non-thinking, launch the server with
`--default-chat-template-kwargs` (a request can still re-enable thinking with its
own `chat_template_kwargs`):

```sh
furiosa-llm serve furiosa-ai/EXAONE-4.5-33B-FP8 \
  --reasoning-parser qwen3 \
  --default-chat-template-kwargs '{"enable_thinking": false}'
```

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser hermes` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats and multimodal server options
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [LGAI-EXAONE/EXAONE-4.5-33B-FP8](https://huggingface.co/LGAI-EXAONE/EXAONE-4.5-33B-FP8)
