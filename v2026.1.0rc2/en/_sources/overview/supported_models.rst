.. _SupportedModels:

**********************************
Supported Models
**********************************

FuriosaAI's software stack supports a wide range of Transformer-based models available on the Hugging Face Hub.
Below is a list of model architectures currently supported by the Furiosa SDK.
If your model is based on any of these architectures, you can use the
Furiosa SDK to compile, quantize, and run the model efficiently on
Furiosa's NPUs.

.. tip::

  Pre-compiled models are available in `Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_.
  You can download and run them quickly with Furiosa-LLM.


Decoder-only Models (Text Generation)
======================================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - DeepSeek R1
     - ``LlamaForCausalLM``
     - ``deepseek-ai/DeepSeek-R1-Distill-Llama-8B``, ``deepseek-ai/DeepSeek-R1-Distill-Llama-70B``
   * - EXAONE 3.0, 3.5, 4.0
     - ``Exaone4ForCausalLM``
     - ``LGAI-EXAONE/EXAONE-4.0-32B``, ``LGAI-EXAONE/EXAONE-4.0-32B-FP8``
   * - Llama 3.1, Llama 3.3
     - ``LlamaForCausalLM``
     - ``meta-llama/Llama-3.1-8B-Instruct``, ``meta-llama/Llama-3.1-70B-Instruct``, ``meta-llama/Llama-3.3-70B-Instruct``
   * - Solar 1.0
     - ``LlamaForCausalLM``
     - ``upstage/SOLAR-10.7B-v1.0``, ``upstage/SOLAR-10.7B-Instruct-v1.0``
   * - Qwen 2, Qwen 2.5
     - ``Qwen2ForCausalLM``
     - ``Qwen/Qwen2.5-Coder-32B-Instruct``, ``Qwen/Qwen2-32B``
   * - Qwen 3
     - ``Qwen3ForCausalLM``
     - ``Qwen/Qwen3-0.5B-Instruct``, ``Qwen/Qwen3-32B-Instruct``

Embedding and Reranking Models
================================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - Qwen 3 Embedding
     - ``Qwen3Model``
     - ``Qwen/Qwen3-Embedding-4B``, ``Qwen/Qwen3-Embedding-8B``
   * - Qwen 3 Reranker
     - ``Qwen3ForSequenceClassification``
     - ``Qwen/Qwen3-Reranker-4B``, ``Qwen/Qwen3-Reranker-8B``

Key Features
============

* **Context Length**: Up to 128k tokens (EXAONE 4.0 with sparse attention)
* **Quantization**: FP8, FP16, BF16 support
* **Parallelism**: Tensor Parallelism, Pipeline Parallelism, Data Parallelism
* **Optimization**: PagedAttention, continuous batching, prefix caching
* **APIs**: OpenAI-compatible endpoints for chat, completions, embeddings, score, and rerank


Planned Models for Future Releases
===============================================
(The order of the models listed below indicate the priority of support.)

* gemma-3
* Llama 4
* Mistral-7B-v0.1, Mistral-7B-Instruct-v0.1
* Mixtral-8x7B-v0.1, Mixtral-8x7B-Instruct-v0.1
* Phi-3, Phi-3.5,
* Phi-3.5-MoE
* DeepSeek-Coder-V2-Instruct
* Stable Diffusion XL

