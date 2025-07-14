Compiling fx.GraphModule
========================

`torch.fx <https://docs.pytorch.org/docs/stable/fx.html#torch-fx>`_
is a toolkit for capturing and transforming PyTorch programs as explicit graphs.
The central abstraction in torch.fx is the GraphModule, which represents a computation
as a Python-compiled dataflow graph. This enables advanced use cases such as
graph-level transformations, lowering, and ahead-of-time compilation.

Furiosa-Torch provides native support for compiling `torch.fx.GraphModule <https://docs.pytorch.org/docs/stable/fx.html#torch.fx.GraphModule>`_ instances
into executables optimized for FuriosaAI NPUs. This allows developers to bypass high-level model tracing and directly target the
execution graph level, giving them finer control over the compilation pipeline.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/compile/index.py
   :language: Python
