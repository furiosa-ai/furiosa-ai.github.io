.. _Release2026_5_0b3:

********************************************
Furiosa SDK Release 2026.5.0b3
********************************************

**Furiosa SDK 2026.5.0b3** is the fourth beta release of 2026.5.0.
The changes since :doc:`2026.5.0b2 <release-2026.5.0b2>` are summarized below.


.. _Release2026_5_0b3_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Model Support & Multimodal Serving
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Added video input for the ``Qwen3VLForConditionalGeneration``,
  ``Qwen3_5ForConditionalGeneration``, and
  ``Qwen3_5MoeForConditionalGeneration`` architectures through the server and
  Python APIs. Decoding video URLs requires ``furiosa-llm[video]`` and FFmpeg
  shared libraries.
* Added video-frame sampling controls and preprocessing caching, including
  reuse of previously submitted videos through UUID references.


Engine Core & Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

* Reduced SSM decode-state memory use with eager scheduling, allowing more
  requests to run concurrently when state capacity limits batching.
* Enabled overlap scheduling for speculative decoding with an independent
  draft model and a text-only target model. Embedded MTP still requires eager
  scheduling.
* Reduced host overhead in hierarchical KV-cache management.
* Refactored prefill/decode-disaggregated transfer management and host-cache
  staging-memory management.


API & Usability Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Added reasoning-effort control through ``reasoning_effort`` in Chat
  Completions and ``reasoning.effort`` in the Responses API. Support and
  accepted values depend on the model's chat template.
* Added support for ``parallel_tool_calls=False`` in Chat Completions and the
  Responses API to return at most one tool call. For automatically parsed tool
  calls, token usage can still include additional generated calls omitted from
  the response.
* Improved NPU error diagnostics with the failing kernel launch, bucket, and
  pipeline stage when available, and suppressed repeated details for the same
  underlying fault.


Bug Fixes
~~~~~~~~~

* Fixed sliding-window prefix-cache corruption after a disaggregated worker
  switches from decoding to prefilling, which could cause incorrect output or
  NPU-sampling failures.
* Fixed KV-cache sizing for data-parallel replicas sharing a single chip.
* Fixed hybrid-attention cache sizing with ``EXPECTED_AVERAGE_SEQ_LENGTH`` to
  account for sliding-window reservations, preventing requests from stalling
  and triggering engine shutdown under affected configurations.
* Fixed waiting-request metrics while the overlap scheduler is idle, including
  prefill workers in disaggregated serving.


Device Runtime
^^^^^^^^^^^^^^

* Fixed NPU out-of-memory failures caused by runtime I/O fragmentation during
  high-concurrency serving with overlap scheduling and NPU sampling.
* Fixed K-EXAONE startup failures with two pipeline stages across eight chips
  by reducing shared-memory use for runtime I/O.
* Reduced excess NPU memory allocation for kernel weights in multi-chip models.
* Reduced redundant synchronization when waiting for DMA transfers to complete.


Compatibility & Security
^^^^^^^^^^^^^^^^^^^^^^^^

* Added an explicit startup error for combining prefill/decode disaggregation
  with L3 caching. Unset ``L3_ENABLED`` or remove the KV-transfer configuration
  to use a supported configuration.
* Updated Rust TLS certificate-validation and trust-store dependencies in
  response to the eMudhra intermediate CA compromise.
