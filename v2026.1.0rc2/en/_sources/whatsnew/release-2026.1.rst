.. _Release2026_1_0:

********************************************
Announcing Furiosa SDK Release 2026.1
********************************************

This release brings major architectural improvements focusing on throughput, efficiency, and scalability.
Key introductions include Hybrid Batching to balance high throughput with low latency, Prefix Caching
for reduced latency in multi-turn applications, and comprehensive support for Pooling Models (Embeddings, Scoring, Reranking).
Additionally, we have enhanced our production readiness with distributed inference capabilities using `LLM-D <https://llm-d.ai/docs/architecture>`_,
NPU operator, `Dynamic Resource Allocation (DRA) <https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/>`_ for K8s, and advanced observability features.

Please refer to the :ref:`UpgradeGuide` section for instructions on
obtaining this update.


.. _Release2026_1_0_Highlights:

🚀 Highlights
-------------

Major Features & Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Hybrid Batching**

  * Introduced hybrid batching capabilities that intelligently combine multiple prefill and decode requests within a single batch.
  * This feature boosts throughput while maintaining low tail latency, achieving up to X times higher requests per second compared to traditional batching methods.

* **Prefix Caching**

  * Automatically detects and reuses common prompt prefixes across requests using a branch-compressed radix tree, eliminating redundant computation.
  * Ideal for applications with shared context such as chatbots with system prompts and RAG systems.
  * Significantly reduces Time-To-First-Token (TTFT) with SIMD-optimized prefix matching and cache eviction.
  * Smart eviction strategies leverage idle time to minimize memory overhead while maintaining high cache hit rates.

* **Pooling Model Support**

  * Added comprehensive support for pooling models to enable critical NLP tasks:

    * **Embeddings**: Generate vector representations for semantic search using ``encode()`` and ``embed()`` APIs.
    * **Scoring**: Evaluate relevance between query-document pairs with the ``score()`` API.
    * **Reranking**: Improve search results by reordering candidates using the ``rerank()`` API.

  * Includes ``PoolingParams.normalize`` support for normalized embedding outputs.
  * Currently supports **Qwen3-8B embedding and reranking models**, with plans to expand support.

* **Structured Output with Multiple Backends**

  * Production-ready structured output generation supporting JSON schema validation, Regular expression constraints, and Grammar-based generation.
  * Includes both ``outlines`` and ``xgrammar`` backends, providing flexibility and performance for various use cases.
  * Optimized guided decoding with bitmask prefetching for reduced latency during NPU task execution.

* **Distributed and Scaling Inference**

  * **LLM-D (LLM Distributed)**: Enabled seamless deployment of large language models across multiple nodes, intelligently handling request routing.

    * Supports KV-cache usage aware routing, prefix-aware routing, and LLM-aware routing.
    * See `LLM-D documentation <https://llm-d.ai/docs/architecture>`_ for more details.

  * **Dynamic Resource Allocation (DRA)**: Intelligent resource management to allocate/deallocate NPU resources based on PCIe topology-aware strategies.
  * **NPU Operator Support**: Added comprehensive support for Kubernetes environments with native NPU operator integration, handling device discovery, driver/fw rolling upgrade, and lifecycle management.

* **Enhanced Observability**

  * Migrated metrics collection to Rust native implementations for improved performance and reduced overhead.
  * Added comprehensive instrumentation including per-device metrics, KV cache utilization, request pool statistics, and detailed scheduler logs for hybrid batching.
  * Integrated **OpenTelemetry** support with quiet span filtering for production environments.
  * Improved liveness check endpoints and health monitoring capabilities.
  * Metrics enabled by default with configurable Prometheus endpoint export.

Performance & Efficiency
^^^^^^^^^^^^^^^^^^^^^^^^

* **Optimization**

  * **Ahead-of-Time (AOT) Wired Pipeline**: Pre-wired execution graphs for low-latency task launches and reduced runtime overhead.
  * **SIMD Optimizations**: Applied throughout the host runtime for decoding strategies, prefix caching, radix tree's branch operations, and other performance-critical paths.

    * Softmax, log-softmax, and normalization operations accelerated with AVX-512 16-lane vectorization (``f32x16``, ``u16x16``).
    * ``libmvec`` integration for SIMD-optimized exponential function, loaded via dynamic linking.
    * Native bf16 (bfloat16) support with automatic f32 widening for numerical precision.
    * Runtime CPU feature detection with automatic scalar fallback for non-AVX-512 systems.
  * **Memory Management**: Improved with expandable buffer pools, smart KV cache allocation strategies, and efficient memory usage for sliding window attention in long context models.
  * **Pipeline Loading**: Parallelized pipeline loading with Ray-based parallelization for next-gen artifact builds.
  * **Warm Up**: Warm-up of tokenizers and templates before serving for reduced cold-start latency.

Expanded Model & Quantization Support
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **New Model Support**:

  * **Qwen3 Family**: Support for 32B variants and 8B embedding/reranking models.
  * **Exaone4**: Comprehensive support including up to 128k context length, sparse attention, and sliding window attention.

* **Quantization**:

  * `Fine-grained FP8 quantization <https://huggingface.co/docs/transformers/en/quantization/finegrained_fp8>`_ support, enabling DeepSeek-style 2D-block weight quantization and per-token group activation quantization.

API & Usability Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Tool Calling Enhancements**:

  * Support for ``tool_choice="required"``, ``tool_choice="auto"``, and named tool selection in Tool Calling API.

* **Sampling Parameters**:

  * Added ``repetition_penalty`` for controlling repetitive outputs.
  * Added ``return_token_ids`` for returning token IDs alongside text.
  * Added ``skip_special_tokens`` for controlling special token inclusion in outputs.
  * Added ``ignore_eos`` for bypassing end-of-sequence token handling.
  * Added ``stop_token_ids`` for custom stopping criteria.

* **Prompt Logprobs Support**:

  * Added ``prompt_logprobs`` parameter to return log probabilities for prompt tokens.
  * Useful for analyzing model confidence and token-level predictions.

* **Security & Authentication**:

  * Implemented API key authentication via ``--api-key`` argument for secure server access.

* **Content Format Improvements**:

  * Support for **Harmony response format** in Chat Completions API with proper reasoning token counting.
  * Support for **content parts format** for multi-part messages in chat completions.


🚨 Breaking Changes & Deprecations
----------------------------------

* **Legacy Artifact Format Deprecated**:

  * Blockwise artifact format is no longer supported. **Migration**: All artifacts must be rebuilt using the new ArtifactBuilder.

* **Python Version Requirement**:

  * Minimum Python version increased to 3.10+. **Migration**: Upgrade to Python 3.10 or later before installing this release.

* **Transformer and PyTorch Library Update**:

  * Bumped transformers to 4.57.1.
  * Bumped PyTorch to 2.7.1. **Migration**: Ensure your environment supports PyTorch 2.7.x.

* **Removed Beam Search Support**:

  * Beam search decoding has been removed from the LLM engine. **Migration**: Use sampling-based decoding methods such as top-k, top-p, or temperature sampling.
