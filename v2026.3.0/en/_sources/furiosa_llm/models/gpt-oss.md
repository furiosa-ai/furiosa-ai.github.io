(furiosa-llm-model-gpt-oss)=

# GPT-OSS

[GPT-OSS](https://huggingface.co/openai) is OpenAI's family of open-weight
reasoning models. They are auto-regressive **Mixture-of-Experts (MoE)**
transformers that emit responses in the
[harmony response format](https://github.com/openai/harmony), separating the
model's chain-of-thought reasoning from its final answer and carrying native
support for tool calling and configurable reasoning effort.

FuriosaAI publishes pre-compiled builds of the GPT-OSS models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model cards linked below.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/gpt-oss-20b`](https://huggingface.co/furiosa-ai/gpt-oss-20b) | MXFP4 | 1 | Lower latency, local / specialized use |
| [`furiosa-ai/gpt-oss-120b`](https://huggingface.co/furiosa-ai/gpt-oss-120b) | MXFP4 | 4 | Production, general-purpose, high reasoning |

- **Architecture:** GPT-OSS (Mixture-of-Experts), `GptOssForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** The MoE expert weights are quantized to **MXFP4**, the format gpt-oss ships in upstream. The remaining components (attention, router, and embeddings) stay in higher precision.

## Usage

To run these models with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Pass the model's `furiosa-ai/<repo>` identifier. Reasoning works out of the box
(the harmony format is auto-detected), so no `--reasoning-parser` flag is needed;
the reasoning content is returned in a separate field (see
[Reasoning](#reasoning) below):

```sh
# gpt-oss-20b — single RNGD card
furiosa-llm serve furiosa-ai/gpt-oss-20b
```

```sh
# gpt-oss-120b — four RNGD cards
furiosa-llm serve furiosa-ai/gpt-oss-120b
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Launch the server with tool calling

To enable tool (function) calling, start the server with the `openai` tool-call
parser:

```sh
furiosa-llm serve furiosa-ai/gpt-oss-120b \
  --enable-auto-tool-choice \
  --tool-call-parser openai
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a request with `curl`
(replace the model id with the variant you launched):

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/gpt-oss-120b",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Reasoning

GPT-OSS models return their reasoning separately from the final answer:

* `response.choices[].message.reasoning` (non-streaming)
* `response.choices[].delta.reasoning` (streaming)

You can control how much effort the model spends reasoning with the
`reasoning_effort` parameter (`"low"`, `"medium"`, or `"high"`):

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="furiosa-ai/gpt-oss-120b",
    messages=[{"role": "user", "content": "How many r's are in 'strawberry'?"}],
    extra_body={"reasoning_effort": "high"},
)

print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

> **Note:** The `reasoning` field is not part of the OpenAI API specification, but
> it is the convention OpenAI [recommends](https://developers.openai.com/cookbook/articles/gpt-oss/verifying-implementations#chat-completions)
> for returning the chain-of-thought (CoT) in Chat Completions-compatible APIs. The
> OpenAI Agents SDK uses `reasoning` as its primary property for the CoT, and many
> LLM serving frameworks (such as vLLM) follow the same convention. It appears only
> in responses that contain reasoning content; accessing it on a response without
> reasoning content raises an `AttributeError`.

### Tool calling

With the server launched using `--enable-auto-tool-choice --tool-call-parser openai`,
you can pass `tools` and let the model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model cards: [openai/gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b), [openai/gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b)
