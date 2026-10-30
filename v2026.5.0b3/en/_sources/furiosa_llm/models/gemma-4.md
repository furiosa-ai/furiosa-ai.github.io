(furiosa-llm-model-gemma-4)=

# Gemma 4

[Gemma 4](https://huggingface.co/google/gemma-4-31B-it) is Google's dense
multimodal model family. Gemma 4 combines a vision encoder with a transformer
decoder that interleaves sliding-window and full-attention layers. Its chat
template supports optional reasoning and tool (function) calling.

FuriosaAI publishes the FP8-block
[`RedHatAI/gemma-4-31B-it-FP8-block`](https://huggingface.co/RedHatAI/gemma-4-31B-it-FP8-block)
weights under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
together with a Furiosa Executable Bundle (FXB) for running them on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/gemma-4-31B-it-FP8-block`](https://huggingface.co/furiosa-ai/gemma-4-31B-it-FP8-block) | FP8 block | 4 | 31B dense; thinking is off by default |

- **Architecture:** Gemma 4 (dense), `Gemma4ForConditionalGeneration`
- **Input / Output:** Image + Text / Text
- **Quantization:** Linear-layer weights use static FP8 quantization with 128×128 blocks, and their input activations use dynamic FP8 quantization at runtime with groups of 128 values. The vision tower, embeddings, output head, and KV cache stay in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Serve the model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4
```

Gemma 4 runs in **non-reasoning mode by default**. Keeping the `gemma4`
reasoning parser enabled lets the server return reasoning in a separate field
when thinking is enabled (see [Advanced Usage](#advanced-usage)).

To also enable tool (function) calling, add automatic tool choice and the
`gemma4` tool-call parser:

```sh
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4 \
  --enable-auto-tool-choice \
  --tool-call-parser gemma4
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

The server exposes an OpenAI-compatible API. You can send a text-only request with
`curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/gemma-4-31B-it-FP8-block",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

To ask about an image, pass an `image_url` content part in the message:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/gemma-4-31B-it-FP8-block",
    "messages": [{
        "role": "user",
        "content": [
            {"type": "image_url", "image_url": {"url": "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/p-blog/candy.JPG"}},
            {"type": "text", "text": "What animal is on the candy?"}
        ]
    }]
    }' \
    | python -m json.tool
```

The `image_url.url` field accepts a remote `http://`/`https://` URL, an inline
base64 `data:` URL, or a local `file://` path (the last requires the
`--allowed-local-media-path` flag described under [Advanced Usage](#advanced-usage)).

Once thinking is enabled (see [Advanced Usage](#advanced-usage)), Gemma 4
returns reasoning separately from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/gemma-4-31B-it-FP8-block",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": True}},
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
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4 \
  --enable-auto-tool-choice \
  --tool-call-parser gemma4 \
  --allowed-local-media-path /srv/media \
  --allowed-media-domains cdn.example.com \
  --image-limit-per-prompt 4
```

See [Server Options](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html#server-options)
in the Vision-Language Models guide for the full option reference and defaults.

**Image processing options.** This example sets a 10-second HTTP image-fetch
timeout and sets Gemma 4's image-token limit to `280` with `max_soft_tokens`:

```sh
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4 \
  --media-io-kwargs '{"image": {"timeout": 10}}' \
  --mm-processor-kwargs '{"max_soft_tokens": 280}'
```

For supported values, sizing behavior, and per-request overrides, see
[Multimodal Options](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html#multimodal-options)
in the Vision-Language Models guide.

**Enabling reasoning.** Gemma 4 runs in non-reasoning mode by default. Enable
reasoning for a single request by passing `enable_thinking` through
`chat_template_kwargs`, as shown under [Basic Usage](#basic-usage). With
`--reasoning-parser gemma4`, the chain of thought is returned separately from
the final answer.

To make every request reason, launch the server with
`--default-chat-template-kwargs` (a request can still override the default):

```sh
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4 \
  --default-chat-template-kwargs '{"enable_thinking": true}'
```

**Preserving reasoning in tool-call conversations.** The chat template also
supports `preserve_thinking`, which defaults to `false`. Set it to `true` to
retain reasoning from earlier assistant tool-call messages when rendering a
multi-turn agent conversation. It does not enable reasoning by itself:

```python
response = client.chat.completions.create(
    model="furiosa-ai/gemma-4-31B-it-FP8-block",
    messages=messages,
    tools=tools,
    extra_body={
        "chat_template_kwargs": {
            "enable_thinking": True,
            "preserve_thinking": True,
        }
    },
)
```

To apply both options by default for every request, use:

```sh
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block \
  --reasoning-parser gemma4 \
  --enable-auto-tool-choice \
  --tool-call-parser gemma4 \
  --default-chat-template-kwargs '{"enable_thinking": true, "preserve_thinking": true}'
```

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser gemma4` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats and multimodal server options
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* [RedHatAI/gemma-4-31B-it-FP8-block](https://huggingface.co/RedHatAI/gemma-4-31B-it-FP8-block) — upstream quantized model card
* [google/gemma-4-31B-it](https://huggingface.co/google/gemma-4-31B-it) — base model card
