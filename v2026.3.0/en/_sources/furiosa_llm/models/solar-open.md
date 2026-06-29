(furiosa-llm-model-solar-open)=

# Solar Open

[Solar Open](https://huggingface.co/upstage) is a large-scale open-weight
language model developed by Upstage. It is an auto-regressive
**Mixture-of-Experts (MoE)** transformer that supports English and Korean, and
it handles both reasoning and non-reasoning chat as well as tool (function)
calling.

FuriosaAI publishes pre-compiled builds of Solar Open under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The base model also runs
on other frameworks (such as vLLM, SGLang, and Transformers); for usage with
those, see the upstream model card linked below.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Solar-Open-100B-NVFP4A16`](https://huggingface.co/furiosa-ai/Solar-Open-100B-NVFP4A16) | NVFP4 | 4 | 100B MoE; reasoning / non-reasoning |

- **Architecture:** SolarOpen (Mixture-of-Experts), `SolarOpenForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** The weights are quantized to **NVFP4** (4-bit floating point), while activations and the KV cache remain in 16-bit precision (NVFP4A16).

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Solar-Open-100B-NVFP4A16 \
  --reasoning-parser solar_open
```

The `--reasoning-parser solar_open` flag separates the model's chain of thought
from the final answer (see [Reasoning](#reasoning) below).

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Launch the server with tool calling

To enable tool (function) calling, start the server with the `solar_open` tool-call
parser:

```sh
furiosa-llm serve furiosa-ai/Solar-Open-100B-NVFP4A16 \
  --reasoning-parser solar_open \
  --enable-auto-tool-choice \
  --tool-call-parser solar_open
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a request with `curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Solar-Open-100B-NVFP4A16",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Reasoning

With `--reasoning-parser solar_open`, Solar-Open returns its reasoning separately
from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

> **Note:** The `reasoning` field is not part of the OpenAI API specification, but
> it is the convention OpenAI recommends for returning the chain-of-thought (CoT) in
> Chat Completions-compatible APIs. The OpenAI Agents SDK uses `reasoning` as its
> primary property for the CoT, and many LLM serving frameworks (such as vLLM) follow
> the same convention. It appears only in responses that contain reasoning content;
> accessing it on a response without reasoning content raises an `AttributeError`.

### Tool calling

With the server launched using `--enable-auto-tool-choice --tool-call-parser solar_open`,
you can pass `tools` and let the model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [upstage/Solar-Open-100B](https://huggingface.co/upstage/Solar-Open-100B)
