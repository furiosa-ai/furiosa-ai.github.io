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
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The current FXB supports
text-to-text inference; image input is not yet supported. The same upstream
weights also run on other frameworks (such as vLLM, SGLang, and Transformers);
for usage with those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/gemma-4-31B-it-FP8-block`](https://huggingface.co/furiosa-ai/gemma-4-31B-it-FP8-block) | FP8 block | 4 | 31B dense; current FXB is text-only; thinking is off by default |

- **Architecture:** Gemma 4 (dense), `Gemma4ForConditionalGeneration`
- **Input / Output:** Text / Text with the current FXB (the upstream model also supports image + text input)
- **Quantization:** Linear-layer weights use static FP8 quantization with 128×128 blocks, and their input activations use dynamic FP8 quantization at runtime with groups of 128 values. The vision tower, embeddings, output head, and KV cache stay in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

Serve the model by passing its `furiosa-ai/<repo>` identifier:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/gemma-4-31B-it-FP8-block
```

Gemma 4's chat template can generate reasoning and tool calls, but Furiosa-LLM
does not currently register parsers for the Gemma 4 output format. Keep the
default non-thinking chat mode and do not enable automatic tool choice with this
FXB.

When the server is ready, you will see:

```sh
INFO:     Started server process [27507]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Basic Usage

The server exposes an OpenAI-compatible API. You can send a text request with
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

### Advanced Usage

**Current feature scope.** The upstream model accepts image input and its chat
template supports optional reasoning through `enable_thinking` as well as native
tool calls. The current Furiosa FXB supports text input only, and Furiosa-LLM
does not yet provide the Gemma 4 reasoning and tool-call parsers required to
return those outputs through the corresponding OpenAI-compatible response
fields. Multimodal serving flags and automatic tool choice therefore do not
enable these upstream capabilities for this FXB.

See the [Vision-Language Models guide](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html)
and [Tool Calling guide](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html)
for the APIs that will apply when those capabilities are supported.

## Learn more

* [Vision-Language Models](https://developer.furiosa.ai/latest/en/furiosa_llm/vision-language.html) — image input formats and multimodal server options
* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — parsers, tool-choice options, and examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* [RedHatAI/gemma-4-31B-it-FP8-block](https://huggingface.co/RedHatAI/gemma-4-31B-it-FP8-block) — upstream quantized model card
* [google/gemma-4-31B-it](https://huggingface.co/google/gemma-4-31B-it) — base model card
