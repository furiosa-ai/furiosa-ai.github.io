.. _Roadmap:

==================================================
Roadmap
==================================================

FuriosaAI is committed to delivering the releases for each month, while offering patch releases.
This page shows the forward-looking roadmap of ongoing & upcoming projects and when they are expected to land, broken down by areas on
:ref:`our software stack <SoftwareStack>`.

.. note::
    The latest release is |release|. You can find the release notes :ref:`here <WhatsNew>`.

Upcoming Releases 2025 Q4
==========================================
* 🔨 Hybrid batching support
* 🔨 Qwen3 MoE model support in Furiosa-LLM
* 🔨 Speculating with a draft model
* 🔨 CPU memory swapping of KV cache in Furiosa-LLM
* 🔨 ``torch.compile()`` backend

Upcoming Releases 2025 Q3
==========================================
* 🔨 Qwen3 support in Furiosa-LLM
* 🔨 Guided-decoding support in Furiosa-LLM
* 🔨 Prefix-caching support in Furiosa-LLM
* 🔨 Pooling Model support in Furiosa-LLM
* 🔨 NPU operator support for Kubernetes
* ✅ Tensor Parallelism support Phase 2: Inter-chip
* ✅ Dramatic performance improvements in Furiosa-LLM (7x higher throughput than 2025.1.0 release)
* ✅ Hugging Face Hub support in Furiosa-LLM
* ✅ Pre-compiled artifacts on Hugging Face Hub
* ✅ Qwen2 and Qwen2.5 model support in Furiosa-LLM
* ✅ EXAONE3 model support in Furiosa-LLM

2025 Q1 - 2025 Q2
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
