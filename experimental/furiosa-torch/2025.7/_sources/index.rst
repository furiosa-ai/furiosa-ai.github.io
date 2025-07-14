.. extension documentation master file, created by
   sphinx-quickstart on Mon Jun 19 15:02:05 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

FuriosaAI RNGD Extension for PyTorch
================================================================

FuriosaAI RNGD Extension for PyTorch offers a seamless integration with existing PyTorch workflows
as well as lower-level APIs for more advanced use cases, allowing you to leverage the power of
FuriosaAI's TCP (Tensor Contraction Processor) architecture.
FuriosaAI RNGD Extension for PyTorch consists of three main components: Furiosa-Torch, Furiosa-HAL, and Furiosa-LLR.

.. warning::

   FuriosaAI RNGD Extension for PyTorch is an alpha release and is not yet ready for production use.
   It is intended for testing and evaluation purposes only.
   The features and APIs described herein are subject to change in the future.


.. grid:: 2
   :padding: 0


   .. grid-item-card:: Quick Start with Furiosa-Torch
      :link: quick_start_with_furiosa_torch
      :link-type: doc
      :columns: 6

      This section describes how to install the FuriosaAI RNGD extension for PyTorch,
      and how to get started with Furiosa-Torch.



   .. grid-item-card:: Furiosa-Torch
      :link: furiosa_torch/torch/index
      :link-type: doc
      :columns: 6

      Furiosa-Torch is a PyTorch extension that enables you to run PyTorch models on FuriosaAI NPUs.


.. grid:: 2
   :padding: 0

   .. grid-item-card:: HAL (Hardware Abstraction Layer)
      :link: furiosa_torch/hal/index
      :link-type: doc
      :columns: 6

      Furiosa-HAL is an extension that provides a hardware abstraction layer for FuriosaAI’s hardware acceleration,
      enabling asynchronous execution of PyTorch models and I/O on FuriosaAI NPUs.

   .. grid-item-card:: Furiosa Low-level Runtime
      :link: furiosa_torch/lla/index
      :link-type: doc
      :columns: 6

      Furiosa-LLR is a low-level runtime that directly exposes low-level primitives
      of the Tensor-Contraction Processor (TCP) architecture.


.. toctree::
   :glob:
   :titlesonly:
   :caption: Getting Started
   :maxdepth: 1
   :hidden:

   quick_start_with_furiosa_torch


.. toctree::
   :glob:
   :titlesonly:
   :caption: Furiosa-Torch
   :maxdepth: 1
   :hidden:


   furiosa_torch/torch/index
   furiosa_torch/torch/tensor
   furiosa_torch/torch/device_configuration
   furiosa_torch/compile/index
   furiosa_torch/torch/custom_extension
   furiosa_torch/examples/index

.. toctree::
   :glob:
   :titlesonly:
   :caption: Furiosa-HAL
   :maxdepth: 3
   :hidden:

   furiosa_torch/hal/index

.. toctree::
   :glob:
   :titlesonly:
   :caption: Furiosa-LLA
   :maxdepth: 3
   :hidden:

   furiosa_torch/lla/index

.. toctree::
   :glob:
   :titlesonly:
   :caption: API References
   :maxdepth: 1
   :hidden:

   native_torch <api_references/native_torch/index>
   torch <api_references/torch/index>

