Exporting Compiled Models
=========================

Furiosa-Torch enables ahead-of-time (AOT) compilation of PyTorch programs
for efficient execution on FuriosaAI NPUs. After exporting a model into
a static graph using `torch.export <https://docs.pytorch.org/docs/stable/export.html>`_,
the key workflow centers around two APIs:

* ``furiosa.torch.compiler.compile_gm(gm, inputs)``: Compiles a torch.fx.GraphModule to a NPU binary program called EDF,
  which encapsulates optimized kernel code and metadata for execution on the NPU.
* ``furiosa.torch.executor.load_edf(edf, device)``: Loads a precompiled EDF into the runtime and
  returns an executable function that can be invoked with inputs.

Exporting to EDF is particularly useful for scenarios where you want to optimize the model execution
and deploy it without the overhead of dynamic tracing or compilation at runtime.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/compile/torch_export.py
   :language: Python
