.. _CustomOperatorExtension:

Custom Operator Extension
=========================

This document introduces how to implement a custom operator extension for execution on FuriosaAI’s RNGD (2nd-generation NPU).
While Furiosa’s PyTorch integration continues to support a wider range of operators natively on RNGD,
there are cases where certain operations may not yet be supported or may require custom implementations to optimize performance or functionality.

To address this, Furiosa provides the ``furiosa.torch.extension`` module, which enables users to define and register
custom implementations of PyTorch operators. These implementations can prioritize execution on RNGD and
gracefully fall back to CPU when necessary.

The following example demonstrates how to define a custom implementation of the ``torch.mm`` operator using a function called ``custom_mm_out``.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/torch/custom_extension.py
   :language: Python
