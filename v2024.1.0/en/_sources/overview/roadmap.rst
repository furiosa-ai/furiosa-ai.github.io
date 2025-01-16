.. _Roadmap:

==================================================
Roadmap
==================================================

FurisaAI strives to deliver the releases for each month, while offering patch releases.
This page shows the forward-looking roadmap of ongoing & upcoming projects
and when they are expected to land, broken down by areas on :ref:`our software stack <SoftwareStack>`.

**************************************
Latest Recent Release
**************************************

The latest release is 2024.1.0 (alpha) on October 11, 2024.
You can find the release notes :ref:`here <WhatsNew>`.

**************************************
Future Releases
**************************************

.. note::

   The roadmap is subject to change and may not reflect the final product.


2024.2.0 (beta 0) - November, 2024
==========================================
* Model support:
   * Language Models: CodeLLaaMA2, Vicuna, Solar, EXAONE-3.0
   * Vision Models: MobileNetV1, MobileNetV2, ResNet152, ResNet50, EfficientNet, YOLOv8m, ..
* Tensor Parallelism support Phase 1: Intra-chip
* Torch 2.4.1 support
* CPU memory swapping of KV cache in Furiosa LLM
* Speculating with a draft model


2024.3.0 (beta 1) - December, 2024
==========================================
* Model support: TBD
* Tensor Parallelism support Phase 2: Inter-chip
* ``torch.compile()`` backend
* Huggingface Optimum integration

