.. _torchTensorInRNGD:

torch.Tensor in RNGD
====================

Furiosa-Torch provides seamless integration with PyTorch to execute tensor operations on RNGD,
FuriosaAI’s 2nd-generation NPU. This section introduces how to move tensors to the device,
execute operations, and utilize both DRAM and SRAM memory spaces within the NPU.

You can move tensors to the RNGD device by using the standard `torch.Tensor.to() <https://docs.pytorch.org/docs/stable/generated/torch.Tensor.to.html>`_
method with a torch.device specifying ``rngd:{index}``.

For more advanced use cases, you can copy tensors into SRAM,
which provides faster access for certain operations. However, due to architectural constraints,
you must first move the tensor to DRAM before copying to SRAM.

To match the internal hardware layout (64 slices), the high-rank shape of the tensor must match the number of slices.
For example: ``Tensor.reshape([64, 4, 256])`` or ``Tensor.reshape([32, 2, 4, 256])``, where 32 * 2 = 64 slices.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/torch/tensor.py
   :language: Python
