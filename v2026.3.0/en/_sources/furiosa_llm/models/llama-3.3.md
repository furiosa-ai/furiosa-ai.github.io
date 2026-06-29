(furiosa-llm-model-llama-3-3)=

# Llama 3.3

[Llama 3.3](https://huggingface.co/meta-llama) is Meta's instruction-tuned,
auto-regressive dense transformer optimized for multilingual dialogue,
instruction following, and tool usage, with quality competitive with much larger
models.

FuriosaAI publishes pre-compiled builds of the Llama 3.3 models under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
each shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same upstream weights
also run on other frameworks (such as vLLM, SGLang, and Transformers); for usage
with those, see the upstream model card linked below.

For the smaller 8B model see {doc}`llama-3.1`.

## Variants

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Llama-3.3-70B-Instruct`](https://huggingface.co/furiosa-ai/Llama-3.3-70B-Instruct) | None (16-bit) | 4 | 70B instruction-tuned |

- **Architecture:** Llama 3.3 (dense), `LlamaForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** No quantization — the model runs in its native 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Llama-3.3-70B-Instruct
```

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Launch the server with tool calling

To enable tool (function) calling, start the server with the `llama3_json`
tool-call parser (the parser used by the Llama 3 series):

```sh
furiosa-llm serve furiosa-ai/Llama-3.3-70B-Instruct \
  --enable-auto-tool-choice \
  --tool-call-parser llama3_json
```

### Query the server

The server exposes an OpenAI-compatible API. You can send a request with `curl`:

```sh
curl http://localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
    "model": "furiosa-ai/Llama-3.3-70B-Instruct",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Tool calling

With the server launched using `--enable-auto-tool-choice --tool-call-parser llama3_json`,
you can pass `tools` and let the model decide when to call them. See the
[Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for a complete client example and details on tool-choice options.

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and more examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [meta-llama/Llama-3.3-70B-Instruct](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct)
