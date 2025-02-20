.. _Roadmap:

==================================================
Roadmap
==================================================

FurisaAI aims to deliver monthly releases, along with patch updates.
This page presents the roadmap for ongoing and upcoming projects, along with
their expected release dates.

**************************************
Latest Recent Release
**************************************

The latest release is |release| (beta 1) on Feb 24, 2025.
You can find the release notes :ref:`here <WhatsNew>`.

**************************************
Future Releases
**************************************


2025 Q1
==========================================
* 🔲 Tensor Parallelism support Phase 2: Inter-chip (planned for 2025.3.0 release)
* 🔲 Speculating with a draft model (planned for 2025.2.0 release)
* 🔲 Embedding API support in Furiosa LLM (planned for 2025.2.0 release)
* 🔲 Chat API support in Furiosa LLM (planned for 2025.2.0 release)
* ✅ Tool-calling support in Furiosa LLM (included in 2025.1.0 release)
* 🔲 CPU memory swapping of KV cache in Furiosa LLM (planned for 2025.3.0 release)
* 🔲 ``torch.compile()`` backend (planned for 2025.3.0 release)
* 🔲 Chunked Prefill support in Furiosa LLM (planned for 2025.2.0 release)
* ✅ Device remapping support (e.g., /dev/rngd/npu2pe0-3 -> /dev/rngd/npu0pe0-3) for container (included in 2025.1.0 release)
* ✅ Automatic configuration for the maximum KV-cache memory allocation (included in 2025.1.0 release)
* ✅ Min-p sampling support (included in 2025.1.0 release)


2024 Q4
==========================================
* ✅ Language Model Support: CodeLLaMA2, Vicuna, Solar, EXAONE-3.0 (2024.2.0 release)
* ✅ Vision Model Support: MobileNetV1, MobileNetV2, ResNet152, ResNet50, EfficientNet, YOLOv8m, etc (2024.2.0 release)
* ✅ Tensor Parallelism support Phase 1: Intra-chip (2024.2.0 release)
* ✅ Torch 2.4.1 support (2024.2.0)
* ✅ Huggingface Optimum integration (2024.2.0 release)
