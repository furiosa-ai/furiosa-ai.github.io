.. _Release2026_5_0b2:

********************************************
Furiosa SDK Release 2026.5.0b2
********************************************

**Furiosa SDK 2026.5.0b2** is the third beta release of 2026.5.0.
The changes since :doc:`2026.5.0b1 <release-2026.5.0b1>` are summarized below.


.. _Release2026_5_0b2_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Model Support & Multimodal Serving
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Added image inputs for speculative decoding, including EXAONE-4.5 MTP.
  Multimodal draft-model decoding requires the same FXB for target and draft;
  MTP remains unavailable for Qwen vision-language models.
* Added guided reasoning and tool-call output for Qwen3.5, Qwen3.6, and Qwen3.8
  in Chat Completions and the Responses API.


Engine Core & Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

* Reduced host-to-NPU data transfers for Qwen3-VL by moving text-embedding
  lookup to the NPU, enabling input prefetch with overlap scheduling and NPU
  sampling.
* Improved profile-guided decode cost estimation by reducing measurement noise
  and timing spikes.
* Refactored KV-cache memory planning.


API & Compatibility
~~~~~~~~~~~~~~~~~~~

* Enabled NPU sampling by default for generative models. Use
  ``--no-enable-npu-sampling`` or Python
  ``SchedulerConfig(enable_npu_sampling=False)`` for host sampling, including
  when using an FXB without sampling kernels. Speculative decoding disables
  NPU sampling automatically; the previous
  ``--experimental-enable-npu-sampling`` flag remains accepted.
* Changed multimodal processor options to use modality-keyed objects. For
  example, replace ``{"min_pixels": 1024}`` with
  ``{"image": {"min_pixels": 1024}}`` in the ``--mm-processor-kwargs`` CLI
  argument, Python ``LLM(mm_processor_kwargs=...)``, and request-level
  ``mm_processor_kwargs``. Request options override defaults within each
  modality.
* Added optional vLLM-style requantization for Mistral NeMo static per-tensor
  FP8 checkpoints through ``--experimental-enable-fp8-max-scale-requant`` and
  Python ``LLM(fp8_max_scale_requant=True)``. This matches vLLM's weight
  handling at a small precision cost and is disabled by default.


Bug Fixes
~~~~~~~~~

* Fixed unnecessary multimodal cache misses when equivalent processor options
  are supplied in a different order.
* Fixed trailing replacement characters (``U+FFFD``) when generation ends
  with an incomplete character.
