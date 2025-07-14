.. _CompilingPyTorchModels:

Compiling PyTorch Models
========================

This document introduces how to compile PyTorch models based on `nn.Module <https://docs.pytorch.org/docs/stable/generated/torch.nn.Module.html>`_
and `torch.fx.GraphModule <https://docs.pytorch.org/docs/stable/fx.html#torch.fx.GraphModule>`_.
PyTorch’s new compilation stack enables more efficient model execution
by converting eager-mode models into graph-based representations,
which can be further optimized and executed on custom backends like Furiosa-Torch.

.. toctree::
   :glob:
   :titlesonly:
   :maxdepth: 2

   torch_compile
   graph_module
   torch_export
   profile
