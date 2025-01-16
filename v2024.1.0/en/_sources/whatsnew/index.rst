.. _WhatsNew:

==================================================
What's New
==================================================

This page describes the changes and functionality available in
in the latest releases of Furiosa SDK 2024.1.0.

.. _Release2024_1_0:
*****************************************************************
Furiosa SDK 2024.1.0 (2024-10-11)
*****************************************************************

2024.1.0 is the first SDK release for RNGD. This release is alpha release,
and the features and APIs described in this document may change in the future.

Highlights
=======================================
* Model Support: LLaMA 3.1 8B/70B, BERT Large, GPT-J 6B
* Furiosa Quantizer supports the following quantization methods:
    * BF16 (W16A16)
    * INT8 Weight-Only (W8A16)
    * FP8 (W8A8)
    * INT8 SmoothQuant (W8A8)
* Furiosa LLM
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
     - 2024.1.0
   * - furiosa-device-plugin
     - 2024.1.0
   * - furiosa-driver-rngd
     - 2024.1.0
   * - furiosa-feature-discovery
     - 2024.1.0
   * - furiosa-firmware-image-tools
     - 2024.1.0
   * - furiosa-firmware-image-rngd
     - 0.0.19
   * - furiosa-libsmi
     - 2024.1.0
   * - furiosa-llm
     - 2024.1.0
   * - furiosa-llm-models
     - 2024.1.0
   * - furiosa-mlperf
     - 2024.1.0
   * - furiosa-mlperf-resources
     - 2024.1.0
   * - furiosa-model-compressor
     - 2024.1.0
   * - furiosa-model-compressor-impl
     - 2024.1.0
   * - furiosa-native-compiler
     - 2024.1.0
   * - furiosa-native-runtime
     - 2024.1.0
   * - furiosa-smi
     - 2024.1.0
   * - furiosa-torch-ext
     - 2024.1.0
