(furiosa-llm-model-k-exaone)=

# K-EXAONE

[K-EXAONE](https://huggingface.co/LGAI-EXAONE) is a large-scale multilingual
language model developed by LG AI Research. It is an auto-regressive
**Mixture-of-Experts (MoE)** transformer with 236B total parameters and 23B
active per token (128 experts, 8 activated plus 1 shared), using a hybrid
attention scheme that interleaves sliding-window and global attention layers. It
covers six languages — Korean, English, Spanish, German, Japanese, and
Vietnamese — and supports both reasoning and non-reasoning chat.

Furiosa-LLM runs K-EXAONE in **NVFP4A16** (NVFP4 weights with 16-bit activations
and KV cache). FuriosaAI publishes pre-compiled NVFP4A16 builds under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The upstream weights also
run on other frameworks (such as vLLM, SGLang, and Transformers); for usage with
those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16`](https://huggingface.co/furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16) | NVFP4A16 | 4 | 236B total / 23B active; thinking by default |

- **Architecture:** ExaoneMoE (Mixture-of-Experts), `ExaoneMoEForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** The weights are quantized to **NVFP4** (4-bit floating point), while activations and the KV cache remain in 16-bit precision (NVFP4A16).

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Pass the model's `furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16` identifier; the model
runs on four RNGD cards. K-EXAONE reasons by default and can switch thinking on
and off per request (see [Advanced Usage](#advanced-usage)).

Serve it with the `deepseek_v3` reasoning parser so the chain of thought is
returned in a separate field:

```sh
furiosa-llm serve furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16 \
  --reasoning-parser deepseek_v3 \
  --default-chat-template-kwargs '{"enable_thinking": true}'
```

The `--default-chat-template-kwargs '{"enable_thinking": true}'` flag keeps the
chat template and the reasoning parser aligned: K-EXAONE's chat template enables
thinking by default, but `deepseek_v3` treats reasoning as disabled unless
`enable_thinking` is set, so without this flag a request that omits
`enable_thinking` would leak the raw `<think>...</think>` text into the response.

To also enable tool (function) calling, add the `hermes` tool-call parser (keep
both the reasoning parser and the `--default-chat-template-kwargs` flag so
thinking is still parsed into its own field):

```sh
furiosa-llm serve furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16 \
  --reasoning-parser deepseek_v3 \
  --default-chat-template-kwargs '{"enable_thinking": true}' \
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
    "model": "furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

With `--reasoning-parser deepseek_v3`, K-EXAONE returns its reasoning separately
from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

K-EXAONE thinks by default, so a normal request returns both the reasoning and
the final answer:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16",
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

**Turning thinking off.** K-EXAONE reasons by default. To turn thinking off for a
single request, pass `enable_thinking` through `chat_template_kwargs`; the
response then carries no reasoning content, so read only `message.content`:

```python
# Disable thinking for a single request
response = client.chat.completions.create(
    model="furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16",
    messages=[{"role": "user", "content": "What is the capital of France?"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": False}},
)
print(response.choices[0].message.content)
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
* Upstream model card: [LGAI-EXAONE/K-EXAONE-236B-A23B](https://huggingface.co/LGAI-EXAONE/K-EXAONE-236B-A23B)
