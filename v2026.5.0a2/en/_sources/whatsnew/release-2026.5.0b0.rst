.. _Release2026_5_0b0:

********************************************
Furiosa SDK Release 2026.5.0b0
********************************************

**Furiosa SDK 2026.5.0b0** is the first beta release of 2026.5.0.
The changes since :doc:`2026.4.0 <release-2026.4.0>` are summarized below.


.. _Release2026_5_0b0_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Model Support & Multimodal Serving
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Added support for the ``Gemma3ForConditionalGeneration`` architecture
  (MedGemma 4B), including image-and-text requests.
* Added full support for the ``Qwen3_5ForConditionalGeneration`` and
  ``Qwen3_5MoeForConditionalGeneration`` architectures, including vision input,
  tool-call parsing, and reasoning.
* Added support for the ``SolarOpen2ForCausalLM`` architecture, including
  tool-call parsing and reasoning.


Engine Core & Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

* Added serving, prefix caching, and configurable state snapshots for
  linear-attention models.
* Enabled multi-replica data parallelism and cache-aware request routing for
  SSM models.
* Enabled overlap scheduling for vision-language models.
* Reduced gaps between batches for text-only models using overlap scheduling
  and NPU sampling.
* Added CPU pinning for NPU I/O threads through the new ``--npu-io-cpus`` CLI
  argument and the ``npu_io_cpus`` argument in the Python ``LLM`` API.
* Enabled L3 KV-cache backup and prefetch with overlap scheduling, and
  refactored L2/L3 cache-transfer management.
* Improved profile-guided decode cost estimation for different batch sizes and
  context lengths, including full and sliding-window attention.
* Refactored input preparation for eager and overlap scheduling and reduced
  host submission overhead.
* Refactored speculative-decoding scheduling to unify target prefill, draft
  generation, and verification.
* Added profiling for KV-cache offload, reload, and promotion, with optional
  detailed DMA traces through ``FURIOSA_PROFILER_DMA_DETAIL=true``.


API & Usability Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Refactored multimodal preprocessing, improved code quality, and enhanced caching.
* Added ``StructuredOutputsParams.terminate_on_accepting`` for xgrammar and
  llguidance. Set it to ``True`` to stop generation as soon as the output
  satisfies the grammar; the default is ``False``.
* Enabled JSON strict mode in xgrammar. Generated objects exclude undeclared
  fields unless allowed by ``additionalProperties``; ``json_object`` responses
  continue to allow arbitrary properties.
* Refactored structured-output handling to unify reasoning and final-content
  configuration.
* Improved Qwen3 Thinking chat-template detection for OpenAI-style text and
  image content lists.
* Expanded time-to-first-token, inter-token latency, and end-to-end latency
  histogram ranges to match vLLM. Added a time-per-output-token metric for
  average token-generation latency after the first token.


Bug Fixes
~~~~~~~~~

* Fixed image-embedding cache reuse across different preprocessing sizes,
  preventing token-count mismatches when reusing the same image.
* Fixed prefix-cache fallback for sliding-window models after host-cache reload
  failures and during prefill/decode disaggregation.
* Fixed the Hermes tool parser's handling of nested JSON in tool arguments.
* Fixed Solar Open structured-output requests with low reasoning effort in
  streaming Chat Completions and the Responses API, including control markers
  leaking into streamed content.
* Added support for Mistral NeMo checkpoints using ModelOpt static FP8
  quantization.


FXB & Compiler Tooling
^^^^^^^^^^^^^^^^^^^^^^

* Added slow-kernel reporting to diagnose FXB compilation bottlenecks, with a
  default threshold of 15 minutes. Configure it with ``--slow-kernel-threshold``.
* Added ``furiosa-tcc --evaluate-tcl`` to estimate the cycle cost of TCL IR
  from a file or standard input without generating an EDF.
* Added FXB Manifest v3, composed-bucket metadata, and initial megakernel build
  infrastructure. Composed megakernel implementations remain in development.


Device Runtime
^^^^^^^^^^^^^^

* Improved weight-loading memory management with huge-page support and
  system-memory fallback, and reduced unnecessary upload copies.
* Improved multi-chip synchronization and execution ordering.
* Reduced the default IPC timeout from 60 to 50 seconds so firmware errors can
  be reported before the driver timeout. Improved timeout and tensor-parallel
  error diagnostics, and reduced IPC overhead on supported drivers.


Compatibility & Rust API Changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Renamed linear-attention runtime settings from ``linear_`` to ``ssm_``.
  Replace ``--linear-full-memory-ratio``, ``--linear-state-cache-size``, and
  ``--linear-state-interval`` with ``--ssm-full-memory-ratio``,
  ``--ssm-state-cache-size``, and ``--ssm-state-interval``, respectively.
  Update the corresponding Python ``LLM`` keyword arguments as well.
* Changed the default SSM state-cache allocation from 1,024 slots to a
  state-to-KV memory ratio of ``1.0``. Use ``--ssm-state-cache-size`` for a fixed
  slot count or ``--ssm-full-memory-ratio`` to adjust the ratio; the options
  are mutually exclusive.
