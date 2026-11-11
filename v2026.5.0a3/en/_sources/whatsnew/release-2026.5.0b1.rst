.. _Release2026_5_0b1:

********************************************
Furiosa SDK Release 2026.5.0b1
********************************************

**Furiosa SDK 2026.5.0b1** is the second beta release of 2026.5.0.
The changes since :doc:`2026.5.0b0 <release-2026.5.0b0>` are summarized below.


.. _Release2026_5_0b1_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Model Support & Multimodal Serving
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Added BF16 support for Qwen3.5-0.8B
  (``Qwen3_5ForConditionalGeneration``) on four processing elements (PEs) of a
  single RNGD NPU. Its FXB can be compiled with
  ``--tensor-parallel-size=4``.
* Refactored multimodal preprocessing and caching, including processor reuse
  across requests.


Engine Core & Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

* Added token log-probability reporting with NPU sampling.
* Improved L3 KV-cache prefetch to reduce memory pressure and redundant
  transfers, and preserve cached prefixes during remote fetches.


Bug Fixes
~~~~~~~~~

* Fixed experimental NPU sampling for Solar Open2 models.
