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
   * - EXAONE 4.0
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

Pooling Models
================================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 120 300

   * - Model Name
     - Architecture
     - Task
     - Example Hugging Face Models
   * - Qwen 3 Embedding
     - ``Qwen3Model``
     - Embedding
     - ``Qwen/Qwen3-Embedding-4B``, ``Qwen/Qwen3-Embedding-8B``
   * - Qwen 3 Reranker
     - ``Qwen3ForSequenceClassification``
     - Reranking
     - ``Qwen/Qwen3-Reranker-4B``, ``Qwen/Qwen3-Reranker-8B``

Planned Models for Future Releases
===============================================
(The order of the models listed below indicate the priority of support.)

* GPT-OSS
* K-EXAONE
* Qwen3 MoE
* Qwen3 VL
