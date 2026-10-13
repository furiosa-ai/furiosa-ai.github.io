(furiosa-llm-model-qwen3-vl)=

# Qwen3-VL

The upstream [Qwen3-VL](https://huggingface.co/Qwen) models are dense **vision-language
models** that pair a vision encoder with a dense transformer decoder, using
Interleaved-MRoPE positional embeddings and DeepStack multi-level feature fusion
to handle images and videos alongside text. They cover visual understanding
tasks such as OCR, document and chart analysis, spatial reasoning, and video
comprehension, and natively support tool (function) calling.

FuriosaAI publishes the Qwen3-VL models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. Both the Instruct
(non-thinking) and Thinking editions are published; the Thinking editions emit an
explicit chain of thought before the final answer (see [Basic Usage](#basic-usage)
below). The same upstream weights also run on other frameworks (such as vLLM,
SGLang, and Transformers); for usage with those, see the upstream model cards
linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-VL-2B-Instruct`](https://huggingface.co/furiosa-ai/Qwen3-VL-2B-Instruct) | None (BF16) | 1 | 2B dense; Instruct (non-thinking) edition |
| [`furiosa-ai/Qwen3-VL-4B-Instruct`](https://huggingface.co/furiosa-ai/Qwen3-VL-4B-Instruct) | None (BF16) | 1 | 4B dense; Instruct (non-thinking) edition |
| [`furiosa-ai/Qwen3-VL-32B-Instruct`](https://huggingface.co/furiosa-ai/Qwen3-VL-32B-Instruct) | None (BF16) | 4 | 32B dense; Instruct (non-thinking) edition |
| [`furiosa-ai/Qwen3-VL-2B-Thinking`](https://huggingface.co/furiosa-ai/Qwen3-VL-2B-Thinking) | None (BF16) | 1 | 2B dense; Thinking edition (always reasons) |
| [`furiosa-ai/Qwen3-VL-4B-Thinking`](https://huggingface.co/furiosa-ai/Qwen3-VL-4B-Thinking) | None (BF16) | 1 | 4B dense; Thinking edition (always reasons) |
| [`furiosa-ai/Qwen3-VL-32B-Thinking`](https://huggingface.co/furiosa-ai/Qwen3-VL-32B-Thinking) | None (BF16) | 4 | 32B dense; Thinking edition (always reasons) |

- **Architecture:** Qwen3-VL (dense), `Qwen3VLForConditionalGeneration`
- **Input / Output:** Image + Text / Text
- **Quantization:** No quantization is applied — the models run in the same precision as the upstream weights.

## Usage

To run these models with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
The examples use the smallest 2B models; the larger sizes run the same way (see
[Available Models](#available-models) for the RNGD card counts).

### Launch the server

Serve a model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Qwen3-VL-2B-Instruct — single RNGD card
furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Instruct
```

The **Thinking** editions always produce a chain of thought before the final
answer; serve them with `--reasoning-parser qwen3` so the reasoning is returned
in a separate field (see [Basic Usage](#basic-usage) below):

```sh
# Qwen3-VL-2B-Thinking — single RNGD card
furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Thinking --reasoning-parser qwen3
```

To also enable tool (function) calling, add the `hermes` tool-call parser (the
parser used by the Qwen3 series):

```sh
furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Instruct \
  --enable-auto-tool-choice \
  --tool-call-parser hermes
```

The **Thinking** editions additionally keep `--reasoning-parser qwen3` so
thinking is still parsed into its own field.

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
    "model": "furiosa-ai/Qwen3-VL-2B-Instruct",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

To ask about an image, pass an `image_url` content part in the message:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-VL-2B-Instruct",
    "messages": [{
        "role": "user",
        "content": [
            {"type": "image_url", "image_url": {"url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"}},
            {"type": "text", "text": "Describe this image."}
        ]
    }]
    }' \
    | python -m json.tool
```

The `image_url.url` field accepts a remote `http://`/`https://` URL, an inline
base64 `data:` URL, or a local `file://` path (the last requires the
`--allowed-local-media-path` flag described under [Advanced Usage](#advanced-usage)).

The **Thinking** editions (such as `furiosa-ai/Qwen3-VL-2B-Thinking`) return
their reasoning separately from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/Qwen3-VL-2B-Thinking",
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

**Multimodal serving options.** This example allows local images under
`/srv/media`, restricts remote image fetches to `cdn.example.com`, and limits
each request to four images:

```sh
furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Instruct \
  --allowed-local-media-path /srv/media \
  --allowed-media-domains cdn.example.com \
  --image-limit-per-prompt 4
```

See [Server Options](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html#server-options)
in the Vision-Language Models guide for the full option reference and defaults.

**Image processing options.** This example sets a 10-second HTTP image-fetch
timeout and uses `min_pixels` and `max_pixels` to bound image resizing:

```sh
furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Instruct \
  --media-io-kwargs '{"image": {"timeout": 10}}' \
  --mm-processor-kwargs '{"min_pixels": 4096, "max_pixels": 200704}'
```

For supported values, sizing behavior, and per-request overrides, see
[Multimodal Options](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html#multimodal-options)
in the Vision-Language Models guide.

**Reasoning.** The **Thinking** editions always reason; there is no
`enable_thinking` switch (the Instruct editions are the non-thinking
counterparts).

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser hermes` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats, multimodal server options, and content-based caching
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model cards: [Qwen/Qwen3-VL-2B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct), [Qwen/Qwen3-VL-4B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct), [Qwen/Qwen3-VL-32B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-32B-Instruct), [Qwen/Qwen3-VL-2B-Thinking](https://huggingface.co/Qwen/Qwen3-VL-2B-Thinking), [Qwen/Qwen3-VL-4B-Thinking](https://huggingface.co/Qwen/Qwen3-VL-4B-Thinking), [Qwen/Qwen3-VL-32B-Thinking](https://huggingface.co/Qwen/Qwen3-VL-32B-Thinking)
