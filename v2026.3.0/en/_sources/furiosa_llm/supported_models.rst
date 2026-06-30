.. _SupportedModels:
.. _FuriosaLLMModels:

**********************************
Supported Models
**********************************

FuriosaAI's software stack supports a wide range of Transformer-based models available on the Hugging Face Hub.
Below is a list of model architectures currently supported by Furiosa-LLM.
If your model is based on any of these architectures, you can use
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

Verified Models
================

You can compile and run any of the architectures listed above on RNGD yourself. The models
below go a step further: each has been validated by FuriosaAI to run correctly **out of the
box** for accuracy and functionality, with performance characterized per the support level
shown below.

Each row lists the Hugging Face repository you pass to ``furiosa-llm serve`` (or ``fxb build``),
the tensor parallel (TP) size it is verified at, the precision it runs in, and its current support
level. ``TP`` is the number of PEs per tensor-parallel group (8 PEs per chip).

Support level uses the following scale:

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 80 420

   * - Level
     - Meaning
   * - 🚀 Optimized
     - Verified for correctness and tuned for performance; recommended for production use.
   * - ✅ Functional
     - Verified to run correctly; performance optimization is ongoing.
   * - 🧪 Experimental
     - Runs, but functionality and performance are still being validated.

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 320 110 60 110 140

   * - Model
     - Type
     - TP
     - Precision
     - Support Level
   * - `EXAONE-4.0-32B-FP8 <https://huggingface.co/furiosa-ai/EXAONE-4.0-32B-FP8>`_
     - Text
     - 32
     - FP8
     - 🚀 Optimized
   * - `K-EXAONE-236B-A23B-NVFP4A16 <https://huggingface.co/furiosa-ai/K-EXAONE-236B-A23B-NVFP4A16>`_
     - Text (MoE)
     - 32
     - NVFP4A16
     - ✅ Functional
   * - `Llama-3.1-8B-Instruct <https://huggingface.co/furiosa-ai/Llama-3.1-8B-Instruct>`_
     - Text
     - 8
     - BF16
     - 🚀 Optimized
   * - `Llama-3.3-70B-Instruct <https://huggingface.co/furiosa-ai/Llama-3.3-70B-Instruct>`_
     - Text
     - 32
     - BF16
     - 🚀 Optimized
   * - `Qwen3-30B-A3B-FP8 <https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-FP8>`_
     - Text (MoE)
     - 32
     - FP8
     - ✅ Functional
   * - `Qwen3-30B-A3B-Instruct-2507-FP8 <https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-Instruct-2507-FP8>`_
     - Text (MoE)
     - 32
     - FP8
     - ✅ Functional
   * - `Qwen3-30B-A3B-Thinking-2507-FP8 <https://huggingface.co/furiosa-ai/Qwen3-30B-A3B-Thinking-2507-FP8>`_
     - Text (MoE)
     - 32
     - FP8
     - ✅ Functional
   * - `Qwen3-32B-FP8 <https://huggingface.co/furiosa-ai/Qwen3-32B-FP8>`_
     - Text
     - 32
     - FP8
     - 🚀 Optimized
   * - `Qwen3-4B-FP8 <https://huggingface.co/furiosa-ai/Qwen3-4B-FP8>`_
     - Text
     - 8
     - FP8
     - ✅ Functional
   * - `Qwen3-8B-FP8 <https://huggingface.co/furiosa-ai/Qwen3-8B-FP8>`_
     - Text
     - 8
     - FP8
     - ✅ Functional
   * - `Qwen3-Coder-30B-A3B-Instruct-FP8 <https://huggingface.co/furiosa-ai/Qwen3-Coder-30B-A3B-Instruct-FP8>`_
     - Text (MoE)
     - 32
     - FP8
     - ✅ Functional
   * - `Qwen3-Embedding-8B <https://huggingface.co/furiosa-ai/Qwen3-Embedding-8B>`_
     - Embedding
     - 8
     - BF16
     - ✅ Functional
   * - `Qwen3-Reranker-8B <https://huggingface.co/furiosa-ai/Qwen3-Reranker-8B>`_
     - Reranking
     - 8
     - BF16
     - ✅ Functional
   * - `Qwen3-VL-32B-Instruct <https://huggingface.co/furiosa-ai/Qwen3-VL-32B-Instruct>`_
     - Multimodal
     - 32
     - BF16
     - ✅ Functional
   * - `Solar-Open-100B-NVFP4A16 <https://huggingface.co/furiosa-ai/Solar-Open-100B-NVFP4A16>`_
     - Text (MoE)
     - 32
     - NVFP4A16
     - ✅ Functional
   * - `gpt-oss-120b <https://huggingface.co/furiosa-ai/gpt-oss-120b>`_
     - Text (MoE)
     - 32
     - MXFP4
     - ✅ Functional
   * - `gpt-oss-20b <https://huggingface.co/furiosa-ai/gpt-oss-20b>`_
     - Text (MoE)
     - 8
     - MXFP4
     - ✅ Functional

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
