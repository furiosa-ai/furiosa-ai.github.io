.. _Roadmap:

==================================================
Roadmap
==================================================

FuriosaAI regularly publishes its software with new features, performance improvements, and expanded hardware support.
This page shows the forward-looking roadmap of ongoing & upcoming projects and when they are expected to land, broken down by areas on
:ref:`our software stack <SoftwareStack>`.

.. note::
    The latest release is |release|. You can find the release notes :ref:`here <WhatsNew>`.

Upcoming Releases 2026 Q3
==========================================
* 🔨 EXAONE 4.5, Qwen 3.6, Gemma 4 model support
* 🔨 Multi-modal (vision-language) serving optimization
* 🔨 Hierarchical KV caching support, including KV cache offloading
* 🔨 KVCacheConnector support
* 🔨 Prefill/decode (PD) disaggregation
* 🔨 Speculative decoding support
* 🔨 PyTorch eager mode support

------------

2026 Q1 - Q2
==========================================

Furiosa-LLM
------------------------------------------
* ✅ Qwen3 MoE, gpt-oss, K-EXAONE, Solar-Open model support
* ✅ Qwen3-VL and multi-modal (vision-language) serving
* ✅ Qwen3 dense and Qwen3 MoE (FP8), EXAONE 4.0 model support
* ✅ TCL (Tensor Contraction Language) kernel framework and furiosa-kernels
* ✅ FXB (Furiosa Executable Bundle) shareable compiled-artifact format
* ✅ Overlap scheduler for zero-overhead batching
* ✅ Per-model bucket presets for best default artifacts
* ✅ Responses API support
* ✅ Data Parallel router with prefix-aware and scoring-based routing
* ✅ Enhanced observability with OpenTelemetry and per-device metrics

Platform & Packages
------------------------------------------
* ✅ Python 3.14 support (supported versions now 3.10–3.14)
* ✅ Broader arm64 (aarch64) support across Python wheels and cloud-native images
* ✅ Rocky Linux 10 / RHEL support via ``.el10`` RPM packages

------------

2025 Q3 - Q4
==========================================

Furiosa-LLM
------------------------------------------
* ✅ Hybrid batching support (i.e., chunked prefill or inflight-batching)
* ✅ Exaone4, Qwen3 support
* ✅ Guided-decoding support (libguidance, xgrammar backends)
* ✅ Tool-calling support
* ✅ Prefix-caching support
* ✅ Pooling Model support (embedding, score, and rank)
* ✅ Fine-tuned model support
* ✅ Tensor Parallelism support Phase 2: Inter-chip
* ✅ Hugging Face Hub support
* ✅ Pre-compiled artifacts on Hugging Face Hub
* ✅ Qwen2 and Qwen2.5 model support
* ✅ EXAONE3 model support
* ✅ API Key based authentication support
* ✅ Harmony response format support

Quantization
------------------------------------------
* ✅ Fine-grained FP8 Quantization (dynamic quantization, mixed quantization)

Distributed & Scalable Inference
------------------------------------------
* ✅ llm-d integration
* ✅ NPU operator support for Kubernetes
* ✅ DRA (Dynamic Resource Allocation) support for Kubernetes


2025 Q1 - Q2
==========================================
* ✅ Tool-calling support in Furiosa-LLM
* ✅ Device remapping support (e.g., /dev/rngd/npu2pe0-3 -> /dev/rngd/npu0pe0-3) for container
* ✅ Automatic configuration for the maximum KV-cache memory allocation
* ✅ Min-p sampling support
* ✅ Chunked Prefill support in Furiosa-LLM
* ✅ Chat API support in Furiosa-LLM
* ✅ Reasoning parser support
* ✅ Torch 2.5.1 support
* ✅ Python 3.11 and 3.12 support
* ✅ Support for building bfloat16, float16, and float32 models to model artifact without quantization
* ✅ Metrics endpoint (``/metrics/``) support in Furiosa-LLM
* ✅ Model artifact support in Huggingface Hub
* ✅ Sampling parameter "logprobs" support
* ✅ Container Runtime and Container Interface Device (CDI) support

2024 Q4
==========================================
* ✅ Language Model Support: CodeLLaMA2, Vicuna, Solar, EXAONE-3.0
* ✅ Vision Model Support: MobileNetV1, MobileNetV2, ResNet152, ResNet50, EfficientNet, YOLOv8m, etc
* ✅ Tensor Parallelism support Phase 1: Intra-chip
* ✅ Torch 2.4.1 support
* ✅ Huggingface Optimum integration
