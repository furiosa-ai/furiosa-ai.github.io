(furiosa-llm-model-mistral-nemo)=

# Mistral NeMo

[Mistral NeMo](https://huggingface.co/mistralai/Mistral-Nemo-Instruct-2407)
is a 12B instruction-tuned, auto-regressive dense transformer developed jointly
by Mistral AI and NVIDIA for assistant-style text generation.

Furiosa-LLM runs Mistral NeMo in **FP8** using RedHatAI's quantized release
(static FP8 weights with dynamic FP8 activation quantization; the KV cache stays
in 16-bit precision). FuriosaAI publishes a pre-compiled build under the
[`furiosa-ai` organization on the Hugging Face Hub](https://huggingface.co/furiosa-ai),
shipping a Furiosa Executable Bundle (FXB) for running it on
[FuriosaAI RNGD](https://furiosa.ai) with Furiosa-LLM. The same weights also run
on other frameworks (such as vLLM, SGLang, and Transformers); for usage with
those, see the upstream model card linked below.

## Available Models

| Model | Quantization | RNGD cards | Notes |
| --- | --- | --- | --- |
| [`furiosa-ai/Mistral-Nemo-Instruct-2407-FP8`](https://huggingface.co/furiosa-ai/Mistral-Nemo-Instruct-2407-FP8) | FP8 | 2 | 12B instruction-tuned |

- **Architecture:** Mistral NeMo (dense), `MistralForCausalLM`
- **Input / Output:** Text / Text
- **Quantization:** The linear (Transformer block) weights are quantized to **FP8** (static), and activations use **dynamic FP8 quantization** at runtime (per-token, without offline calibration). The KV cache stays in 16-bit precision.

## Usage

To run this model with Furiosa-LLM, follow the example commands below after
[installing Furiosa-LLM and its prerequisites](https://developer.furiosa.ai/latest/en/get_started/furiosa_llm.html#installing-furiosa-llm).

### Launch the server

The simplest way to serve the model is:

```sh
# Launch the server, listening on port 8000 by default
furiosa-llm serve furiosa-ai/Mistral-Nemo-Instruct-2407-FP8
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
    "model": "furiosa-ai/Mistral-Nemo-Instruct-2407-FP8",
    "messages": [{"role": "user", "content": "What is the capital of France?"}]
    }' \
    | python -m json.tool
```

### Advanced Usage

Mistral-Nemo-Instruct-2407-FP8 is a non-reasoning chat model and does not
require a reasoning parser or model-specific chat-template flags. For general
serving options, see the [Furiosa-LLM Server guide](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html).

## Learn more

* [Tool Calling](https://developer.furiosa.ai/latest/en/furiosa_llm/toolcalling.html) — supported parsers, tool-choice options, and examples
* [Furiosa-LLM Server (`furiosa-llm serve`)](https://developer.furiosa.ai/latest/en/furiosa_llm/furiosa-llm-serve.html) — full OpenAI-compatible API reference and serving options
* Upstream model card: [RedHatAI/Mistral-Nemo-Instruct-2407-FP8](https://huggingface.co/RedHatAI/Mistral-Nemo-Instruct-2407-FP8)
