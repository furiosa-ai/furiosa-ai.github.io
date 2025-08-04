.. _WhatsNew:

***************
What's New
***************

.. contents:: Table of Contents
  :depth: 1
  :local:
  :backlinks: none


.. _Release2025_3_0:

Furiosa SDK 2025.3.0 Beta3 (2025-08-04)
=======================================
The 2025.3.0 release primarily focuses on inter-chip tensor parallelism and performance optimizations.
The efforts have resulted in dramatic performance improvements, including up to 3x throughput increase
for Llama 3.1 70B and up to 55% reduction in first-token latency for Llama 3.1 8B.
This release also introduces support for the Qwen 2 and 2.5 models, as well as W8A16 quantization.


Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.

.. _Release2025_3_0_Highlights:

🚀 Highlights
-------------

Major Features & Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Inter-chip Tensor Parallelism

  * Tensor parallelism across multiple NPU cards is now officially supported, enabling efficient
    scaling of large models and leading to significantly improved throughput.
  * To maximize performance, this feature is backed by key optimizations including:
    optimized PCIe paths for peer-to-peer (P2P) communication, advanced communication scheduling,
    and compiler tactics that overlap inter-chip DMA with computation.

* Compiler and Runtime Optimizations

  * Enhanced Global Compiler Optimization: Furiosa compiler's global optimization
    capabilities were enhanced to maximize SRAM reuse between transformer blocks.
    This reduces memory access latency and boosts overall throughput.
  * Runtime Optimization: Further optimized the runtime by reducing interference
    between the host and NPU, improving synchronization across devices, and
    minimizing overhead between consecutive decoding steps.

Performance Highlights
^^^^^^^^^^^^^^^^^^^^^^

The optimizations in this release yield the following performance improvements compared
to the previous release 2025.2:

* Llama 3.1 8B: Up to 4.5% average throughput improvement and up to a 55% average reduction in
  Time-to-First-Token (TTFT).
* Llama 3.1 70B: Up to 3x average throughput improvement and up to a 35% average reduction in
  Time-to-First-Token (TTFT).
* The experiment configuration is as follows:

  * input lengths: 1k ~ 12k
  * output lengths: 128 ~ 10k
  * batch size: 1 ~ 128

Expanded Model Support
^^^^^^^^^^^^^^^^^^^^^^

* New Model Families: Added support for Qwen 2 and 2.5 models
  (e.g., `Qwen2.5-Coder-32B-Instruct <https://huggingface.co/furiosa-ai/Qwen2.5-Coder-32B-Instruct>`_).
* New Quantization Support: Added support for W8A16 quantization
  (e.g., `Llama-3.3-70B-Instruct-INT8 <https://huggingface.co/furiosa-ai/Llama-3.3-70B-Instruct-INT8>`_).
* New pre-compiled artifacts are available on the `Hugging Face Hub <https://huggingface.co/furiosa-ai>`_:

  * `EXAONE-3.5-32B-Instruct <https://huggingface.co/furiosa-ai/EXAONE-3.5-32B-Instruct>`_
  * `DeepSeek-R1-Distill-Llama-70B <https://huggingface.co/furiosa-ai/DeepSeek-R1-Distill-Llama-70B>`_
  * `Llama-3.3-70B-Instruct <https://huggingface.co/furiosa-ai/Llama-3.3-70B-Instruct>`_
  * `Llama-3.3-70B-Instruct-INT8 <https://huggingface.co/furiosa-ai/Llama-3.3-70B-Instruct-INT8>`_
  * `Qwen2.5-Coder-32B-Instruct <https://huggingface.co/furiosa-ai/Qwen2.5-Coder-32B-Instruct>`_

* Longer Context Lengths: all pre-compiled artifacts on `Hugging Face Hub <https://huggingface.co/furiosa-ai>`_ now support
  context lengths of up to 32k tokens.


🚨 Breaking Changes
--------------------
* The SDK 2025.2.0 cannot load artifacts built with 2025.3.x.
  Please use the artifact built with 2025.3.x, or rebuild the model again with the new SDK.
* furiosa-mlperf is deprecated and is removed from this release.
  Please use other benchmark tools, such as `vLLM benchmark <https://github.com/vllm-project/vllm/tree/main/benchmarks>`_
  or `LLMPerf <https://github.com/ray-project/llmperf>`_.

.. _Release2025_2_0:

Furiosa SDK 2025.2.0 Beta2 (2025-04-25)
=======================================

RNGD SDK 2025.2.0 is the fourth major release, bringing a wide range of new features and significant improvements,
including support for reasoning models, the metrics endpoint, the chat API, the Hugging Face Hub, the abort() API,
and the chunked prefill feature. This release also enables direct building of bfloat16, float16, and float32 models from
the Hugging Face Hub without a quantization step. Additionally, pre-compiled model artifacts are now available
on the Hugging Face Hub, so you can use them immediately without having to build them yourself.

Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.

.. _Release2025_2_0_Highlights:

🚀 Highlights
-------------
* Add support for `stream_options.include_usage <https://community.openai.com/t/usage-stats-now-available-when-using-streaming-with-the-chat-completions-api-or-completions-api/738156>`_ in :ref:`OpenAIServer`.
* Introduce ``LLM.chat()`` API to support chat-based models (see :ref:`FuriosaLLMExamplesChat`, :ref:`FuriosaLLMExamplesChatWithTools`).
* Mitigate out-of-memory issue by setting the default value of ``spare_block_ratio=0`` in :ref:`OpenAIServer`.
* Fix a bug caused by duplicate buckets in ``furiosa-llm``.
* Add support for ``/v1/models`` and ``/v1/models/{model_id}`` endpoints in ``furiosa-llm`` (see :ref:`ModelsEndpoint`).
* Add support for ``/version`` endpoint in :ref:`OpenAIServer` (see :ref:`VersionEndpoint`).
* Fix a bug that prevented interruption of a running :ref:`OpenAIServer`. by ``Ctrl+C``.
* Add support the chunked prefill feature in ``furiosa-llm`` (see :ref:`ChunkedPrefill`).
* Enable direct building of bfloat16/float16/float32 models without quantization step (see :ref:`AutoBfloat16Cast`).
* Add support for the reasoning model parser in :ref:`OpenAIServer` (see :ref:`Reasoning`).
* ``LLM`` API, ``furiosa-mlperf``, ``furiosa-llm serve`` now support loading artifacts from Hugging Face Hub.
* Add support for ``npu_queue_limit`` option in ``furiosa-llm serve`` command to configure the NPU queue limit.
* ``furiosa-llm`` now supports Python 3.11 and 3.12.
* Optimize the NPU DRAM stack usage for the ``furiosa-llm``.
* Support Ubuntu 24.04 (Noble Numbat).
* Remove the group ``furiosa`` to access NPU devices on Linux system.
* Pre-compiled model artifacts are now available in Hugging Face Hub.
* Add support for ``abort()`` in ``LLMEngine`` and ``AsyncLLMEngine`` APIs.
* Add support for the metrics endpoint (``/metrics``) used to monitor the health of :ref:`OpenAIServer` (see :ref:`MetricsEndpoint`).
* Support sampling parameter "logprobs" in Furiosa-LLM (see :ref:`SamplingParams`).
* Add support for Container Device Interface (CDI) for container runtimes (e.g., docker, containerd, and crio) (see :ref:`Container`).

🚨 Breaking Changes
--------------------
* The SDK 2025.2.0 cannot load artifacts built with 2025.1.x. Please use the artifact built with 2025.2.x, or rebuild the model again with the new SDK.
* The `furiosa` group is no longer required to access NPU devices on Linux systems.


Versions of components:

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-compiler
     - 2025.2.0
   * - furiosa-driver-rngd
     - 2025.2.0
   * - furiosa-firmware-tools-rngd
     - 2025.2.0
   * - furiosa-firmware-image-rngd
     - 2025.2.0
   * - furiosa-pert-rngd
     - 2025.2.0
   * - furiosa-model-compressor
     - 2025.2.0
   * - furiosa-llm
     - 2025.2.0
   * - furiosa-llm-models
     - 2025.2.0
   * - furiosa-mlperf
     - 2025.2.0
   * - furiosa-mlperf-resources
     - 4.1.0
   * - furiosa-native-compiler
     - 2025.2.0
   * - furiosa-native-runtime
     - 2025.2.0
   * - furiosa-feature-discovery
     - 2025.2.0
   * - furiosa-device-plugin
     - 2025.2.0
   * - furiosa-smi
     - 2025.2.0
   * - furiosa-libsmi
     - 2025.2.0

.. raw:: html

   <hr>


.. _Release2025_1_0:

Furiosa SDK 2025.1.0 Beta1 (2025-02-24)
==============================================

2025.1.0 is the third major SDK release for RNGD. This release includes a lot of new features and significant
improvements, including significant LLM latency optimization, tool-calling support in Furiosa-LLM,
the device remapping support for container environment, command line tools improvements, and bug fixes.

Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.

.. _Release2025_1_0_Highlights:

🚀 Highlights
-------------
* LLM Latency Optimization (Up to 11.66% TTFT, 11.45% TPOT improvement for 30k inputs, 1k outputs)
* Support Tool-calling in Furiosa-LLM (`Tool Calling <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/furiosa-llm-serve.html#tool-calling>`_)
* Support Device remapping (e.g., ``/dev/rngd/npu2pe0-3`` -> ``/dev/rngd/npu0pe0-3``) for container
* Add the new command line tool ``furiosa-llm build`` to build easily an artifact from Hugging Face model (`Building a Model Artifact <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/model-preparation-workflow.html#building-a-model-artifact>`_)
* Fix continuous batch scheduling bugs which occur in certain ranges of sequence lengths and batch sizes
* Automatic configuration of the maximum KV-cache memory allocation
* Reduce fragmentation in runtime memory allocation
* Allow ``furiosa-mlperf`` command to specify ``pipeline_parallel_size`` and ``data_parallel_size``
* Add ``--allowed-origins`` argument to ``furiosa-llm serve`` (`OpenAIServer <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/furiosa-llm-serve.html>`_)
* Fix ``trust_remote_code`` support bug in furiosa-llm
* Support Min-p sampling in ``SamplingParams`` (`SamplingParams class <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/reference/sampling_params.html>`_)
* Allow ``npu:X`` in addition to ``npu:X:*`` in ``devices`` option
    * e.g., ``furiosa-llm serve ./model --devices "npu:0"``
* ``furiosa-mlperf`` command supports ``npu_queue_limit``, ``spare_blocks_ratio``, allowing to optimize the performance

⚠️ Deprecations & Upcoming Changes
----------------------------------
* ``LLM.from_artifacts()`` API will be deprecated from the 2025.2.0 release. Please use ``LLM.load_artifact()`` instead (`LLM class <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/reference/llm.html>`_).

🚨 Breaking Changes
--------------------
* ``--model`` option of ``furiosa-llm serve`` become a positional argument.
  Please use ``furiosa-llm serve <model>`` instead of ``furiosa-llm serve --model <model>``. (`OpenAIServer <https://developer.furiosa.ai/v2025.1.0/en/furiosa_llm/furiosa-llm-serve.html>`_)


Versions of components:

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-compiler
     - 2025.1.0
   * - furiosa-driver-rngd
     - 2025.1.0
   * - furiosa-firmware-tools-rngd
     - 2025.1.0
   * - furiosa-firmware-image-rngd
     - 2025.1.0
   * - furiosa-pert-rngd
     - 2025.1.0
   * - furiosa-model-compressor
     - 2025.1.0
   * - furiosa-llm
     - 2025.1.0
   * - furiosa-llm-models
     - 2025.1.0
   * - furiosa-mlperf
     - 2025.1.0
   * - furiosa-mlperf-resources
     - 4.1.0
   * - furiosa-native-compiler
     - 2025.1.0
   * - furiosa-native-runtime
     - 2025.1.0
   * - furiosa-feature-discovery
     - 2025.1.0
   * - furiosa-device-plugin
     - 2025.1.0
   * - furiosa-smi
     - 2025.1.0
   * - furiosa-libsmi
     - 2025.1.0

.. raw:: html

   <hr>

.. _Release2024_2_1:

Furiosa SDK 2024.2.1 Beta0 (2025-01-10)
=======================================

2024.2.1 is a minor release based on 2024.2.0 major release.

Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.

.. _Release2024_2_1_Highlights:

🚀 Highlights
-------------
* Support for context lengths of up to 32k in furiosa-llm for various models, including LLaMA 3.1, and EXAONE
* Artifacts with the same ``tensor_parallel_size`` are compatible even with any ``pipeline_parallel_size``

Versions of components:

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-compiler
     - 2024.2.0
   * - furiosa-driver-rngd
     - 2024.2.1
   * - furiosa-firmware-tools-rngd
     - 2024.2.1
   * - furiosa-firmware-image-rngd
     - 2024.2.0
   * - furiosa-pert-rngd
     - 2024.2.1
   * - furiosa-model-compressor
     - 2024.2.0
   * - furiosa-llm
     - 2024.2.1
   * - furiosa-llm-models
     - 2024.2.0
   * - furiosa-mlperf
     - 2024.2.1
   * - furiosa-mlperf-resources
     - 4.1.0
   * - furiosa-native-compiler
     - 2024.2.0
   * - furiosa-native-runtime
     - 2024.2.1
   * - furiosa-feature-discovery
     - 2024.2.0
   * - furiosa-device-plugin
     - 2024.2.0
   * - furiosa-smi
     - 2024.2.0
   * - furiosa-libsmi
     - 2024.2.0

.. raw:: html

   <hr>

.. _Release2024_2_0:

Furiosa SDK 2024.2.0 Beta0 (2024-12-23)
=======================================

2024.2.0 is the second major SDK release for RNGD.
This release includes a lot of new features and significant improvements,
including new model support, support for context lengths of up to 8k,
support for Tensor Parallelism, support for PyTorch 2.4, Optimum API, and
multiple performance improvements.

Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.


.. _Release2024_2_0_Highlights:

🚀 Highlights
-------------
* New Model support: Solar, EXAONE-3.0, CodeLLaMA2, Vicuna
* Up to 8k context length support in models, such as LLaMA 3.1
* Tensor Parallelism support (``tensor_parallel_size <= 8``)
* PyTorch 2.4.1 support
* Transformers 4.44.2 support
* Furiosa-LLM
    * ArtifactBuilder API and CLI tools (refer to `ArtifactBuilder <https://developer.furiosa.ai/v2024.2.0/en/furiosa_llm/furiosa-llm-build.html#artifactbuilder>`_)
        * Users can build artifacts from Huggingface Hub models with Huggingface Transformers compatible API
    * Huggingface Transformers compatible API support (`furiosa_llm.optimum`)
        * AutoModel, AutoModelForCausalLM, AutoModelForQuestionAnswering API
        * QuantizerForCausalLM API support for calibration and quantization
    * LLMEngine, AsyncLLMEngine API support compatible with vLLM
* About 20% performance improvements in models based on LlamaForCausalLM
    * e.g., 3580 tokens/sec in LLaMA 3.1 8B model with a single RNGD card

🚨 Breaking Changes
-------------------
* LLM.from_artifacts() API has been deprecated. Please use LLM.load_artifacts() instead.
* The artifacts built from 2024.1.x is not compatible with 2024.2.x. Please use the artifact built from 2024.2.x.

.. list-table:: Component version
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-compiler
     - 2024.2.0
   * - furiosa-driver-rngd
     - 2024.2.0
   * - furiosa-firmware-tools-rngd
     - 2024.2.0
   * - furiosa-firmware-image-rngd
     - 2024.2.0
   * - furiosa-pert-rngd
     - 2024.2.0
   * - furiosa-llm
     - 2024.2.0
   * - furiosa-llm-models
     - 2024.2.0
   * - furiosa-mlperf
     - 2024.2.0
   * - furiosa-mlperf-resources
     - 4.1.0
   * - furiosa-model-compressor
     - 2024.2.0
   * - furiosa-native-compiler
     - 2024.2.0
   * - furiosa-native-runtime
     - 2024.2.0
   * - furiosa-smi
     - 2024.2.0
   * - furiosa-libsmi
     - 2024.2.0
   * - furiosa-device-plugin
     - 2024.2.0
   * - furiosa-feature-discovery
     - 2024.2.0

.. _Release2024_1_0:

.. raw:: html

   <hr>

Furiosa SDK 2024.1.0 Alpha (2024-10-11)
=======================================

2024.1.0 is the first SDK release for RNGD. This release is alpha release,
and the features and APIs described in this document may change in the future.

.. _Release2024_1_0_Highlights:

🚀 Highlights
------------------------
* Model Support: LLaMA 3.1 8B/70B, BERT Large, GPT-J 6B
* Furiosa Quantizer supports the following quantization methods:
    * BF16 (W16A16)
    * INT8 Weight-Only (W8A16)
    * FP8 (W8A8)
    * INT8 SmoothQuant (W8A8)
* Furiosa-LLM
    * Efficient KV cache management with PagedAttention
    * Continuous batching support in serving
    * OpenAI-compatible API server
    * Greedy search and beam search
    * Pipeline Parallelism and Data Parallelism across multiple NPUs
* ``furiosa-mlperf`` command
    * Server and Offline scenarios
    * BERT, GPT-J, LLaMA 3.1 benchmarks
* System Management Interface
    * System Management Interface Library and CLI for Furiosa NPU family
* Cloud Native Toolkit
    * Kubernetes integration for managing and monitoring the Furiosa NPU family


.. list-table:: Component version
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-compiler
     - 2024.2.0
   * - furiosa-device-plugin
     - 2024.2.0
   * - furiosa-driver-rngd
     - 2024.2.0
   * - furiosa-feature-discovery
     - 2024.1.0
   * - furiosa-firmware-tools-rngd
     - 2024.1.0
   * - furiosa-firmware-image-rngd
     - 2024.1.0
   * - furiosa-libsmi
     - 2024.2.0
   * - furiosa-llm
     - 2024.2.0
   * - furiosa-llm-models
     - 2024.2.0
   * - furiosa-mlperf
     - 2024.2.0
   * - furiosa-mlperf-resources
     - 4.1.0
   * - furiosa-model-compressor
     - 2024.1.0
   * - furiosa-native-compiler
     - 2024.2.0
   * - furiosa-native-runtime
     - 2024.2.0
   * - furiosa-smi
     - 2024.1.0
