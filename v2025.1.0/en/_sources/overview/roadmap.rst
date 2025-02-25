.. _Roadmap:

==================================================
Roadmap
==================================================

FuriosaAI is committed to delivering the releases for each month, while offering patch releases. 
This page shows the forward-looking roadmap of ongoing & upcoming projects and when they are expected to land, broken down by areas on 
:ref:`our software stack <SoftwareStack>`.

.. note::
    The latest release is |release| (beta 1) on Feb 24, 2025. You can find the release notes :ref:`here <WhatsNew>`.


**************************************
Current Release Progress 2024 Q4
**************************************
* ✅ Language Model Support: CodeLLaMA2, Vicuna, Solar, EXAONE-3.0 (2024.2.0 release)
* ✅ Vision Model Support: MobileNetV1, MobileNetV2, ResNet152, ResNet50, EfficientNet, YOLOv8m, etc (2024.2.0 release)
* ✅ Tensor Parallelism support Phase 1: Intra-chip (2024.2.0 release)
* ✅ Torch 2.4.1 support (2024.2.0 release)
* ✅ Huggingface Optimum integration (2024.2.0 release)
* ✅ Device remapping support (e.g., /dev/rngd/npu2pe0-3 -> /dev/rngd/npu0pe0-3) for container (2025.1.0 release)
* ✅ Tool-calling support in Furiosa LLM (2025.1.0 release)
* 🔨 CPU memory swapping of KV cache in Furiosa LLM (postponed to 2025 Q1)
* 🔨 Speculating with a draft model (postponed to 2025 Q1)
* 🔨 ``torch.compile()`` backend (postponed to 2025 Q1)


**************************************
Upcoming Releases 2025 Q1
**************************************

* Tensor Parallelism support Phase 2: Inter-chip (planned for 2025.3.0 release)
* Speculating with a draft model (planned for 2025.2.0 release)
* CPU memory swapping of KV cache in Furiosa LLM (planned for 2025.3.0 release)
* ``torch.compile()`` backend (planned for 2025.3.0 release)
* Embedding API support in Furiosa LLM (planned for 2025.2.0 release)
* Chunked Prefill support in Furiosa LLM (planned for 2025.2.0 release)
* Chat API support in Furiosa LLM (planned for 2025.2.0 release)