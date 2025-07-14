Intro to Furiosa-Torch
=================================

Furiosa-Torch is an extension that enables PyTorch to run on FuriosaAI’s 2nd NPU, RNGD.
It supports eager execution of various tensor operations and models directly on RNGD,
and also provides a torch.compile() backend for executing compiled models.

Furiosa-Torch is designed to be compatible with existing PyTorch code,
making it easy to integrate into your workflow and run existing PyTorch models on RNGD.

You just need three steps to get started with Furiosa-Torch:

1. import the ``furiosa.torch`` module in adaptation to ``import torch``
2. define the RNGD device like ``torch.device("rngd:0")``
3. move the tensor to the RNGD device using `torch.Tensor.to() <https://docs.pytorch.org/docs/stable/generated/torch.Tensor.to.html>`_

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/torch/index.py
   :language: Python

Documentation
-------------

* :ref:`QuickStartWithFuriosaTorch`: A quick start guide to Furiosa-Torch
* :ref:`torchTensorInRNGD`: How to use torch.Tensor in RNGD
* :ref:`DeviceConfiguration`: How to configure the device setup for FuriosaAI’s RNGD NPU
* :ref:`CompilingPyTorchModels`: How to compile PyTorch models using Furiosa-Torch
* :ref:`CustomOperatorExtension`: How to extend custom operators using Furiosa-Torch
* :ref:`PyTorchxRNGDExamples`: More examples of using PyTorch with RNGD
