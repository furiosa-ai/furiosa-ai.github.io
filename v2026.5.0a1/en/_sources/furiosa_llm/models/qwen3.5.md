(furiosa-llm-model-qwen3.5)=

# Qwen3.5 / Qwen3.6 / Qwen3.8 (dense)

The dense Qwen3.5 architecture pairs a vision encoder with a decoder that
combines Gated DeltaNet linear attention and full attention. Qwen3.6-27B uses
this architecture (`Qwen3_5ForConditionalGeneration`) and supports reasoning,
tool (function) calling, and visual understanding. Thinking is enabled by
default and can be turned off per request.

[Qwen3.8-27B-FP8](https://huggingface.co/Qwen/Qwen3.8-27B-FP8) uses the same
architecture, kernel shapes, and FP8 quantization configuration as
Qwen3.6-27B-FP8, so it can use the same Furiosa-LLM compilation path. The
pre-compiled model and usage examples below cover Qwen3.6-27B-FP8.

FuriosaAI packages the upstream FP8 weights together with a Furiosa Executable
Bundle (FXB) for running them on [FuriosaAI RNGD](https://furiosa.ai) with
Furiosa-LLM under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai).
The same upstream weights also run on other frameworks (such as vLLM, SGLang,
and Transformers); for usage with those, see the upstream model card linked
below.

For the Qwen3 dense chat models see {doc}`qwen3`, and for the Qwen3-VL
vision-language models see {doc}`qwen3-vl`.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3.6-27B-FP8`](https://huggingface.co/furiosa-ai/Qwen3.6-27B-FP8) | FP8 | 4 | 27B dense; hybrid thinking / non-thinking; experimental |

- **Architecture:** Qwen3.5 (dense), `Qwen3_5ForConditionalGeneration`
- **Input / Output:** Image + Text / Text
- **Quantization:** Quantized projections use **static FP8 weights** (E4M3, 128 × 128 blocks) and **dynamic FP8 activation quantization** at runtime (per-token / per-block), without offline activation calibration. Modules excluded by the upstream quantization configuration, including the vision encoder and LM head, retain higher precision. The KV cache stays in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Pass the model's `furiosa-ai/<repo>` identifier. `Qwen3.6-27B-FP8` runs on four
RNGD cards (tensor-parallel size 32 PEs). It reasons by default and can switch
thinking on and off (see [Advanced Usage](#advanced-usage)).

Serve it with the `qwen3` reasoning parser so the reasoning is returned in a
separate field:

```sh
furiosa-llm serve furiosa-ai/Qwen3.6-27B-FP8 \
  --reasoning-parser qwen3
```

To also enable tool (function) calling, add the `qwen3_coder` tool-call parser
(also available as `qwen3_xml`) for this model's XML tool-call format:

```sh
furiosa-llm serve furiosa-ai/Qwen3.6-27B-FP8 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
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
    "model": "furiosa-ai/Qwen3.6-27B-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

To ask about an image, pass an `image_url` content part in the message:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3.6-27B-FP8",
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
Furiosa-LLM currently serves text and image inputs for this model.

With thinking enabled, the model returns its reasoning separately from the
final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/Qwen3.6-27B-FP8",
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

**Toggling thinking.** The model reasons by default. To turn thinking off for a
single request, pass `enable_thinking` through `chat_template_kwargs`; the
response then carries no reasoning content, so read only `message.content`:

```python
# Disable thinking for a single request
response = client.chat.completions.create(
    model="furiosa-ai/Qwen3.6-27B-FP8",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": False}},
)
print(response.choices[0].message.content)
```

To default every request to non-thinking, launch the server with
`--default-chat-template-kwargs` (a request can still re-enable thinking with its
own `chat_template_kwargs`):

```sh
furiosa-llm serve furiosa-ai/Qwen3.6-27B-FP8 \
  --reasoning-parser qwen3 \
  --default-chat-template-kwargs '{"enable_thinking": false}'
```

**Multimodal serving options.** This example allows local images under
`/srv/media`, restricts remote image fetches to `cdn.example.com`, and limits
each request to four images:

```sh
furiosa-llm serve furiosa-ai/Qwen3.6-27B-FP8 \
  --reasoning-parser qwen3 \
  --allowed-local-media-path /srv/media \
  --allowed-media-domains cdn.example.com \
  --image-limit-per-prompt 4
```

See [Server Options](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html#server-options)
in the Vision-Language Models guide for the full option reference and defaults.

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser qwen3_coder` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats and multimodal server options
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* [Qwen/Qwen3.6-27B-FP8](https://huggingface.co/Qwen/Qwen3.6-27B-FP8) — upstream model card
