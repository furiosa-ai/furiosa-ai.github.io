(furiosa-llm-model-llama-3-1)=

# Llama 3.1

[Llama 3.1](https://huggingface.co/meta-llama) is Meta's series of
instruction-tuned, auto-regressive dense transformers optimized for multilingual
dialogue, instruction following, and tool usage.

FuriosaAI publishes pre-compiled builds of the Llama 3.1 models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

For the newer 70B revision see {doc}`llama-3.3`.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Llama-3.1-8B-Instruct`](https://huggingface.co/furiosa-ai/Llama-3.1-8B-Instruct) | None (16-bit) | 1 | 8B instruction-tuned |
| [`furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic`](https://huggingface.co/furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic) | FP8 (dynamic) | 1 | 8B instruction-tuned; RedHatAI FP8-dynamic build |

- **Architecture:** Llama 3.1 (dense), `LlamaForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** `furiosa-ai/Llama-3.1-8B-Instruct` runs in native 16-bit; `furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic` uses static FP8 weights with dynamic FP8 activation quantization (per-token).

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic
```

To also enable tool (function) calling, add the `llama3_json` tool-call parser
(the parser used by the Llama 3 series):

```sh
furiosa-llm serve furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic \
  --enable-auto-tool-choice \
  --tool-call-parser llama3_json
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
    "model": "furiosa-ai/Meta-Llama-3.1-8B-Instruct-FP8-dynamic",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Advanced Usage

**Tool calling.** With the server launched using
`--enable-auto-tool-choice --tool-call-parser llama3_json` (see
[Launch the server](#launch-the-server)), pass `tools` in the request and let the
model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [meta-llama/Llama-3.1-8B-Instruct](https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct)
