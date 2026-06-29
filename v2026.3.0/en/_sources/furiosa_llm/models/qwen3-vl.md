(furiosa-llm-model-qwen3-vl)=

# Qwen3-VL

The [Qwen3-VL](https://huggingface.co/Qwen) series are dense **vision-language
models** that pair a vision encoder with a dense transformer decoder, using
Interleaved-MRoPE positional embeddings and DeepStack multi-level feature fusion
to handle images and videos alongside text. They cover visual understanding
tasks such as OCR, document and chart analysis, spatial reasoning, and video
comprehension, and natively support tool (function) calling.

FuriosaAI publishes pre-compiled builds of the Qwen3-VL models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model cards linked below.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Qwen3-VL-32B-Instruct`](https://huggingface.co/furiosa-ai/Qwen3-VL-32B-Instruct) | None (16-bit) | 4 | 32B dense; Instruct (non-thinking) edition |

- **Architecture:** Qwen3-VL (dense), `Qwen3VLForConditionalGeneration`
- **Input / Output:** Image + Text / Text
- **Quantization:** No quantization is applied — the model runs in the same precision as the upstream weights.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct
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
parser:

```sh
furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct \
  --enable-auto-tool-choice \
  --tool-call-parser hermes
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a text-only request
with `curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-VL-32B-Instruct",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

To ask about an image, pass an `image_url` content part in the message:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Qwen3-VL-32B-Instruct",
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
base64 `data:` URL, or a local `file://` path (the latter requires the
`--allowed-local-media-path` flag below).

### Multimodal serving options

`furiosa-llm serve` provides flags to control multimodal behavior; requests
that violate them are rejected with HTTP 400:

- `--image-limit-per-prompt N` / `--video-limit-per-prompt N` — maximum number of images/videos allowed per request (default: unlimited).
- `--allowed-local-media-path PATH` — allow `file://` URLs whose resolved path is under `PATH`. Local file access is disabled unless this is set.
- `--allowed-media-domains D [D ...]` — whitelist of remote domains for SSRF protection. When set, only images from the listed domains are fetched.
- `--interleave-mm-strings` — keep image placeholders at their original positions when the model uses a string-format chat template (no-op for OpenAI-format templates, the common case).
- `--mm-processor-cache-gb GB` — size of the UUID-keyed multimodal processor cache (default: 4.0). Clients can tag an `image_url` part with a `uuid` field and re-reference it in follow-up requests without re-uploading the image bytes; set to 0 to disable.

For example, to serve local images under `/srv/media` and restrict remote
fetches to a single domain:

```sh
furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct \
  --allowed-local-media-path /srv/media \
  --allowed-media-domains cdn.example.com \
  --image-limit-per-prompt 4
```

See the [Vision-Language Models guide](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html)
for image input formats, the UUID cache, and Python client examples.

### Tool calling

With the server launched using `--enable-auto-tool-choice --tool-call-parser hermes`,
you can pass `tools` and let the model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats, multimodal server options, and the UUID cache
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [Qwen/Qwen3-VL-32B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-32B-Instruct)
