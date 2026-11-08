(furiosa-llm-model-solar-open)=

# Solar Open

[Solar Open](https://huggingface.co/upstage) is a large-scale open-weight
language model developed by Upstage. It is an auto-regressive
**Mixture-of-Experts (MoE)** transformer that supports English and Korean, and
it handles both reasoning and non-reasoning chat as well as tool (function)
calling.

Furiosa-LLM runs Solar Open in **NVFP4A16** (NVFP4 weights with 16-bit activations
and KV cache). FuriosaAI publishes pre-compiled NVFP4A16 builds under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The upstream weights also
run on other frameworks (such as vLLM, SGLang, and Transformers); for usage with
those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Solar-Open-100B-NVFP4A16`](https://huggingface.co/furiosa-ai/Solar-Open-100B-NVFP4A16) | NVFP4A16 | 4 | 100B MoE; reasoning / non-reasoning |

- **Architecture:** SolarOpen (Mixture-of-Experts), `SolarOpenForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** The weights are quantized to **NVFP4** (4-bit floating point), while activations and the KV cache remain in 16-bit precision (NVFP4A16).

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Serve the model with the `solar_open` reasoning parser so the chain of thought is
returned in a separate field:

```sh
furiosa-llm serve furiosa-ai/Solar-Open-100B-NVFP4A16 \
  --reasoning-parser solar_open
```

To also enable tool (function) calling, add the `solar_open` tool-call parser;
keep `--reasoning-parser solar_open` so thinking is still parsed into its own
field:

```sh
furiosa-llm serve furiosa-ai/Solar-Open-100B-NVFP4A16 \
  --reasoning-parser solar_open \
  --enable-auto-tool-choice \
  --tool-call-parser solar_open
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
    "model": "furiosa-ai/Solar-Open-100B-NVFP4A16",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

With `--reasoning-parser solar_open`, Solar Open returns its reasoning separately
from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/Solar-Open-100B-NVFP4A16",
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

**Controlling reasoning effort.** You can control how much the model reasons per
request with the `reasoning_effort` parameter (`"low"`, `"medium"`, or `"high"`;
default `"high"`). Solar Open's chat template maps this to its thinking behavior:
`"high"` and `"medium"` produce a chain of thought — with `"high"` allowed a larger
reasoning-token budget than `"medium"` — while `"low"` (and `"minimal"`) prefill an
empty thinking block so the model skips reasoning and answers directly:

```python
response = client.chat.completions.create(
    model="furiosa-ai/Solar-Open-100B-NVFP4A16",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
    extra_body={"reasoning_effort": "high"},
)

print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser solar_open` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [upstage/Solar-Open-100B](https://huggingface.co/upstage/Solar-Open-100B)
