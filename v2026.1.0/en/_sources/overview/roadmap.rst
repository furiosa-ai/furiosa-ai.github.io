.. _Roadmap:

==================================================
Roadmap
==================================================

FuriosaAI regularly publishes its software with new features, performance improvements, and expanded hardware support.
This page shows the forward-looking roadmap of ongoing & upcoming projects and when they are expected to land, broken down by areas on
:ref:`our software stack <SoftwareStack>`.

.. note::
    The latest release is |release|. You can find the release notes :ref:`here <WhatsNew>`.

Upcoming Releases 2026 Q1
==========================================
* 🔨 Qwen3 MoE, GPT-OSS, K-EXAONE model support
* 🔨 Qwen3 VL and multi-modal model support
* 🔨 KV cache offloading support
* 🔨 OpenAI Response format support
* 🔨 Speculative decoding support
* 🔨 PyTorch eager mode support

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
* ✅ Tool-calling support in Furiosa-LLM (2025.1.0 release)
* ✅ Device remapping support (e.g., /dev/rngd/npu2pe0-3 -> /dev/rngd/npu0pe0-3) for container (2025.1.0 release)
* ✅ Automatic configuration for the maximum KV-cache memory allocation (2025.1.0 release)
* ✅ Min-p sampling support (2025.1.0 release)
* ✅ Chunked Prefill support in Furiosa-LLM (planned for 2025.2.0 release)
* ✅ Chat API support in Furiosa-LLM (planned for 2025.2.0 release)
* ✅ Reasoning parser support (2025.2.0 release)
* ✅ Torch 2.5.1 support (2025.2.0 release)
* ✅ Python 3.11 and 3.12 support (2025.2.0 release)
* ✅ Support for building bfloat16, float16, and float32 models to model artifact without quantization (2025.2.0 release)
* ✅ Metrics endpoint (``/metrics/``) support in Furiosa-LLM (2025.2.0 release)
* ✅ Model artifact support in Huggingface Hub (2025.2.0 release)
* ✅ Sampling parameter "logprobs" support (2025.2.0 release)
* ✅ Container Runtime and Container Interface Device (CDI) support (2025.2.0 release)

2024 Q4
==========================================
* ✅ Language Model Support: CodeLLaMA2, Vicuna, Solar, EXAONE-3.0 (2024.2.0 release)
* ✅ Vision Model Support: MobileNetV1, MobileNetV2, ResNet152, ResNet50, EfficientNet, YOLOv8m, etc (2024.2.0 release)
* ✅ Tensor Parallelism support Phase 1: Intra-chip (2024.2.0 release)
* ✅ Torch 2.4.1 support (2024.2.0)
* ✅ Huggingface Optimum integration (2024.2.0 release)
