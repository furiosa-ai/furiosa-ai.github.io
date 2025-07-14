Profiling Compiled Models
=========================

Profiling is a critical part of optimizing deep learning models for performance.
PyTorch provides a flexible `torch.profiler <https://docs.pytorch.org/docs/stable/profiler.html>`_ API
for measuring execution time and operator-level statistics.
The example below demonstrates how to profile an eager PyTorch model and
EDF executable using the `torch.profiler <https://docs.pytorch.org/docs/stable/profiler.html>`_ API.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/compile/profile.py
   :language: Python
