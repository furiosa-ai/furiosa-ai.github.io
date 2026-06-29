.. _SupportedModels:
.. _FuriosaLLMModels:

**********************************
Supported Models
**********************************

FuriosaAI's software stack supports a wide range of Transformer-based models available on the Hugging Face Hub.
Below is a list of model architectures currently supported by the Furiosa-LLM.
If your model is based on any of these architectures, you can use the
Furiosa-LLM to compile and run the model efficiently on
Furiosa's NPUs.

For many of these architectures, FuriosaAI also publishes **pre-compiled
models** under the
`Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_,
each shipping a Furiosa Executable Bundle (FXB) so you can download and run it
quickly with Furiosa-LLM. The architecture names in the tables below link to
per-architecture guides covering how to launch the pre-compiled variants with
the ``furiosa-llm serve`` command — including the model-specific options each
needs — and how to use their features, such as reasoning, tool calling, and
multimodal input, with example requests. Each guide also notes the
quantization and parallelism strategy for reference. Individual
repository-level model cards live on the Hugging Face Hub.

Decoder-only Models (Text Generation)
======================================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - :doc:`EXAONE 4.0 <models/exaone-4.0>`
     - ``Exaone4ForCausalLM``
     - ``LGAI-EXAONE/EXAONE-4.0-32B-FP8``, ``LGAI-EXAONE/EXAONE-4.0-32B``
   * - :doc:`K-EXAONE <models/k-exaone>`
     - ``ExaoneMoEForCausalLM``
     - ``LGAI-EXAONE/K-EXAONE-236B-A23B``
   * - :doc:`GPT-OSS <models/gpt-oss>`
     - ``GptOssForCausalLM``
     - ``openai/gpt-oss-20b``, ``openai/gpt-oss-120b``
   * - :doc:`Llama 3.1 <models/llama-3.1>`, :doc:`Llama 3.3 <models/llama-3.3>`
     - ``LlamaForCausalLM``
     - ``meta-llama/Llama-3.1-8B-Instruct``, ``meta-llama/Llama-3.3-70B-Instruct``
   * - :doc:`Solar Open <models/solar-open>`
     - ``SolarOpenForCausalLM``
     - ``upstage/Solar-Open-100B``
   * - :doc:`Qwen 2.5 <models/qwen2.5>`
     - ``Qwen2ForCausalLM``
     - ``Qwen/Qwen2.5-0.5B-Instruct``
   * - :doc:`Qwen 3 <models/qwen3>`
     - ``Qwen3ForCausalLM``
     - ``Qwen/Qwen3-8B``, ``Qwen/Qwen3-32B``
   * - :doc:`Qwen 3 MoE <models/qwen3-moe>`
     - ``Qwen3MoeForCausalLM``
     - ``Qwen/Qwen3-30B-A3B-Instruct-2507``

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
   * - :doc:`Qwen 3 Embedding <models/qwen3-embedding>`
     - ``Qwen3Model``
     - Embedding
     - ``Qwen/Qwen3-Embedding-8B``, ``Qwen/Qwen3-Embedding-4B``
   * - :doc:`Qwen 3 Reranker <models/qwen3-reranker>`
     - ``Qwen3ForSequenceClassification``
     - Reranking
     - ``Qwen/Qwen3-Reranker-8B``, ``Qwen/Qwen3-Reranker-4B``

Vision-Language Models (Multimodal)
======================================

See :ref:`VisionLanguageModels` for a guide on launching a VL server
and sending image inputs over the OpenAI-compatible Chat Completions
API.

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 200 120 300

   * - Model Name
     - Architecture
     - Modalities
     - Example Hugging Face Models
   * - :doc:`Qwen 3 VL <models/qwen3-vl>`
     - ``Qwen3VLForConditionalGeneration``
     - Text, Image
     - ``Qwen/Qwen3-VL-32B-Instruct``, ``Qwen/Qwen3-VL-2B-Instruct-FP8``

For models planned for future releases, see the :ref:`Roadmap`.

.. toctree::
   :maxdepth: 1
   :hidden:

   models/qwen3
   models/qwen3-moe
   models/qwen3-embedding
   models/qwen3-reranker
   models/qwen3-vl
   models/qwen2.5
   models/llama-3.1
   models/llama-3.3
   models/gpt-oss
   models/exaone-4.0
   models/k-exaone
   models/solar-open
