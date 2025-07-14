Compiling nn.Module
===================

`torch.compile <https://docs.pytorch.org/docs/stable/torch.compiler.html>`_ is a powerful API introduced to enable ahead-of-time (AOT) compilation of
PyTorch programs. It captures the full Python-level computation graph and lowers it to an optimized backend for improved performance.
This allows users to seamlessly accelerate PyTorch models without modifying model code.

The ``furiosa.torch.backend`` provides a custom backend implementation compatible with `torch.compile <https://docs.pytorch.org/docs/stable/torch.compiler.html>`_,
targeting FuriosaAI NPUs. It optimizes the captured computation graph, and lowers PyTorch programs to executable NPU programs optimized for the target device.

The following example demonstrates how to use torch.compile to compile a simple nn.Module using the Furiosa backend and run it on an NPU device:

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/compile/torch_compile.py
   :language: Python
