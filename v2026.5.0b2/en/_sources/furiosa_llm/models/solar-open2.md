(furiosa-llm-model-solar-open2)=

# Solar Open 2

Solar Open 2 is Upstage's **Mixture-of-Experts (MoE)** language model family.
Its hybrid attention architecture combines grouped-query softmax attention
with Kimi Delta Attention (KDA) linear attention. It supports reasoning,
non-reasoning chat, and tool (function) calling.

The checkpoint below has approximately **34.3B parameters**, according to its
weight index metadata. It has 40 layers, 128 routed experts and one shared
expert per layer, and activates four routed experts per token. For the family
architecture, see the public
[Solar-Open2-250B model card](https://huggingface.co/upstage/Solar-Open2-250B).

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| solar-mini4 | FP8 | 4; 2 with the alternate FXB | Approximately 34.3B MoE; reasoning / non-reasoning; performance experimental |

- **Architecture:** Solar Open 2 (Mixture-of-Experts), `SolarOpen2ForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** Quantized linear layers use **static per-channel FP8 weights** and **dynamic per-token FP8 activation quantization** at runtime, without offline activation calibration. Attention output projections, shared experts, and the LM head retain higher precision. The KV cache uses **static per-tensor FP8 quantization**.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).
These examples require a build with support for a checkpoint and the `solar_open2` reasoning and tool-call parsers.

### Launch the server

The primary FXB uses four RNGD cards (tensor-parallel size 32 PEs). A separate
two-card FXB uses tensor-parallel size 16 PEs; see
[Advanced Usage](#advanced-usage) to select it explicitly.

Serve the model with the `solar_open2` reasoning parser so reasoning is returned
in a separate field:

```sh
furiosa-llm serve <repo id or path> \
  --reasoning-parser solar_open2
```

To also enable tool (function) calling, add the `solar_open2` tool-call parser:

```sh
furiosa-llm serve <repo id or path> \
  --reasoning-parser solar_open2 \
  --enable-auto-tool-choice \
  --tool-call-parser solar_open2
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
    "model": "EMPTY",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

Reasoning is enabled by default. With `--reasoning-parser solar_open2`, it is
returned separately from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="EMPTY",
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

**Controlling reasoning.** The chat template defaults to
`reasoning_effort="high"`. To skip reasoning for a single request, pass
`"low"` through `chat_template_kwargs` and read only `message.content`:

```python
response = client.chat.completions.create(
    model="EMPTY",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
    extra_body={"chat_template_kwargs": {"reasoning_effort": "low"}},
)
print(response.choices[0].message.content)
```

To default every request to non-reasoning, set
`--default-chat-template-kwargs`. A request can re-enable reasoning by setting
its own `chat_template_kwargs` to `{"reasoning_effort": "high"}`:

```sh
furiosa-llm serve <repo id or path> \
  --reasoning-parser solar_open2 \
  --default-chat-template-kwargs '{"reasoning_effort": "low"}'
```

The template enables reasoning for `"medium"`, `"high"`, `"xhigh"`, and `"max"`.
These settings share the same server reasoning-token budget. To change that
budget, set `SOLAR_REASONING_BUDGET` before starting the server; the default is
131,072 tokens, and `0` removes this reasoning-specific cap. The request's
`max_tokens` still limits the complete generated response, including reasoning
and the final answer.

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser solar_open2` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. Include parameter types in each tool's JSON
schema so the parser can convert argument values to the intended types. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* [upstage/Solar-Open2-250B](https://huggingface.co/upstage/Solar-Open2-250B) — architecture reference and setup for other frameworks
