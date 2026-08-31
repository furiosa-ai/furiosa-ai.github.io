.. _Release2026_4_0:

********************************************
Furiosa SDK Release 2026.4.0
********************************************

We are happy to announce the release of **Furiosa SDK 2026.4**.

This release turns several foundations introduced in 2026.3 into end-to-end serving
capabilities. **Hierarchical KV caching** can now retain prefixes beyond NPU memory in host
memory and, optionally, in a cluster-wide Mooncake store. Multimodal serving is now
integrated into the regular FCFS scheduler, with batched vision-encoder execution and reuse
of encoded images across turns. 2026.4 also introduces experimental **speculative decoding**.
Model coverage expands to Qwen3-VL 2B/4B, Mistral NeMo, and two state-of-the-art model
families: **Gemma 4** and **EXAONE 4.5**. The embedding and reranking lineup also grows
with the Qwen3 families, BGE-M3, E5-Mistral, BGE Reranker v2 M3, and Harrier OSS v1.
The runtime also moves completely to the **FXB-based model
path**, with faster model loading, warmup, and sampling.

If you are upgrading from 2026.3, please also read the
:ref:`Release2026_4_0_BreakingChanges` section and the :ref:`UpgradeGuide`.


Highlights
----------

Hierarchical KV Cache Offloading
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prefix caching avoids repeated prefill for long, shared contexts, but a conventional cache is
bounded by NPU DRAM. Once a cached prefix is evicted, a later request must recompute it even
if the same prefix was used only moments earlier.

2026.4 introduces an experimental **hierarchical KV cache** that expands prefix reuse beyond
NPU memory. This release also consolidates the prefix-cache foundation in the new
``UnifiedRadixCache`` implementation, replacing separate global-only and hybrid cache paths
with one radix-tree topology. Global-attention KV and the sliding-window auxiliary KV remain
component-local within that tree, so they share the same prefix-matching and scheduling
infrastructure while retaining their own LRU, reference, and eviction state.

The hierarchical cache carries this component-based model across three tiers:

* **L1 — NPU DRAM:** the active, lowest-latency KV blocks used directly by attention.
* **L2 — host memory:** evicted or proactively backed-up blocks are copied over DMA and can
  be reloaded instead of recomputed.
* **L3 — external storage:** an optional backend retains prefixes beyond local process
  memory; a shared backend makes them reusable by other Furiosa-LLM replicas in the cluster.

L3 is storage-independent by design. A backend-agnostic **KV cache connector** interface
(``RemoteKvStore``) separates the radix tree and cache controller from backend-specific
existence checks and data transfers. Additional storage systems can be integrated behind this
interface without changing the cache hierarchy. **Mooncake is the first supported backend**,
providing content-addressed, cluster-wide prefix storage and asynchronous transfer outside the
scheduler's critical path.

The scheduler can proactively write hot prefixes to L2, preserve host copies when their L1
blocks are reclaimed, and reload only the missing suffix when part of a requested prefix is
already resident locally. Offloading also works with hybrid global/sliding-window attention
and intra-node pipeline parallelism.

Enable the L2 tier by assigning a process-wide host-memory budget:

.. code-block:: bash

   furiosa-llm serve <model> --experimental-kv-offload-host-memory-gb 64

Alternatively, ``--experimental-kv-offload-host-memory-ratio`` sizes L2 relative to each
model lane's L1 KV capacity. Hot prefixes are backed up using a write-through policy by
default; ``--experimental-kv-offload-write-back`` additionally preserves NPU-only blocks at
eviction time, but can add synchronous DMA latency.

This feature is **experimental and off by default**. Its options are deliberately hidden
from ``furiosa-llm serve --help`` while the policy and defaults are being tuned. With the
backend included in 2026.4, L3 requires a compatible Mooncake deployment and shared-library
installation; see :ref:`K8sMooncakeDeployment` for setup and namespace configuration.


Faster, Broader Multimodal Serving
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The vision-language support introduced in 2026.3 now runs end to end on the regular FCFS
scheduler. This unifies text-only and image-and-text requests under the same scheduling,
prefix-cache, and data-parallel infrastructure instead of a separate proof-of-concept loop.

The new path reduces NPU idle time in several ways:

* Vision-encoder inputs from multiple requests can be **batched**.
* The vision encoder and text decoder use **separate NPU task queues**, allowing host-side
  preparation and independent NPU work to overlap more effectively.
* A two-phase acquire-and-encode/generate flow prepares image embeddings before admitting
  the text generation phase.
* The multimodal token registry is shared by DP workers. Encoded vision embeddings receive
  stable media identities and can be reused across turns without rerunning the vision
  encoder; their lifetime is tied to prefix-cache insertion and eviction.
* Host-side image preparation and mRoPE offset handling have been moved out of latency-
  sensitive execution points.

These optimizations are implemented in the shared multimodal serving path rather than being
specific to **Qwen3-VL**. They will also apply to other vision-language model families,
including **Gemma 4** and **EXAONE 4.5**, as their end-to-end serving support is added.
Requests use the OpenAI-compatible Chat Completions ``image_url`` format described in
:ref:`VisionLanguageModels`.


Expanded Model and Pooling Coverage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2026.4 substantially broadens both generation and retrieval workloads:

* :ref:`Qwen3-VL <furiosa-llm-model-qwen3-vl>` — new 2B and 4B Instruct/Thinking models,
  alongside the existing 32B family.
* :ref:`Qwen3 Embedding <furiosa-llm-model-qwen3-embedding>` — 0.6B, 4B, and 8B models,
  served through ``/v1/embeddings`` or used through the offline ``LLM.embed`` API.
* :ref:`Qwen3 Reranker <furiosa-llm-model-qwen3-reranker>` — 0.6B, 4B, and 8B models,
  served through the score and rerank endpoints or used with ``LLM.score``.
* :ref:`Harrier OSS v1 <furiosa-llm-model-harrier-oss-v1>` — the 0.6B multilingual,
  instruction-aware text embedding model.
* :ref:`Mistral NeMo <furiosa-llm-model-mistral-nemo>` — the 12B
  ``Mistral-Nemo-Instruct-2407-FP8`` text-generation model.
* :ref:`Llama 3.1 <furiosa-llm-model-llama-3-1>` and
  :ref:`Llama 3.3 <furiosa-llm-model-llama-3-3>` — dynamically quantized FP8 variants.
* :ref:`BGE-M3 <furiosa-llm-model-bge-m3>` — a multilingual dense-embedding model,
  served through ``/v1/embeddings`` or used through the offline ``LLM.embed`` API.
* :ref:`BGE Reranker v2 M3 <furiosa-llm-model-bge-reranker-v2-m3>` — a multilingual
  cross-encoder served through the score and rerank APIs.
* :ref:`E5-Mistral <furiosa-llm-model-e5-mistral>` — the instruction-aware
  ``E5-Mistral-7B-Instruct`` embedding model.
* :ref:`Gemma 4 <furiosa-llm-model-gemma-4>` — the 31B text-generation model.
* :ref:`EXAONE 4.5 <furiosa-llm-model-exaone-4-5>` — the 33B vision-language model
  supporting text and image input.
* **Qwen3.6-27B** — a hybrid linear- and full-attention architecture integrated on the
  lower-level FXB/RawModel path; end-to-end serving is planned for the next release.

Except for Qwen3.6-27B, these entries are available for end-to-end serving in 2026.4.
Consult :ref:`SupportedModels` for the models that FuriosaAI publishes and validates.


Experimental Speculative Decoding
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2026.4 introduces a new **speculative-decoding** path. A draft model
proposes several tokens while the target model verifies them together; accepted tokens let
the server make more than one token of progress per target decode step. Target and draft
lanes share one scheduler and I/O-memory pool, avoiding the duplicated cache and routing
state used by the earlier prototype.

The feature is exposed through experimental, hidden CLI options:

.. code-block:: bash

   furiosa-llm serve <target-model> \
       --experimental-spec-model <draft-model> \
       --experimental-spec-fxb <draft-model.fxb> \
       --experimental-spec-tokens 4

The target and draft must use identical tokenizer token-to-ID mappings and the same tensor
parallel degree. The draft executes with pipeline parallelism 1 on the first target PP rank.
Vision-language models currently support only self-speculation with matching multimodal
layouts.

Speculative decoding remains **experimental**. Prefix caching is disabled when it is active,
and it is currently incompatible with overlap scheduling, hierarchical KV offloading, PD
disaggregation, structured output, and log-probability requests.


TCL and FXB Model Lifecycle
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The previous model stack—models described in PyTorch and distributed in the legacy v2
artifact format—has been deprecated and replaced by the combination introduced in 2026.3:
**TCL (Tensor Contraction Language)** describes and compiles the model's NPU kernels, while
:ref:`FXB` packages those compiled kernels for loading and distribution. At serving time,
the Hugging Face model supplies the configuration, tokenizer, and weights, and the FXB
supplies the compiled NPU program. The legacy v2 artifact loader, model rewriter, and
``furiosa-llm build`` command have been removed.

This combination preserves the model's high-level intent instead of flattening it into
fine-grained Torch ATen operations. The compiler can therefore optimize fusion, hardware
mapping, padding, sharding, and multi-chip collectives with the model structure intact.
Reusable TCL kernels also make adding a model architecture primarily a matter of composing
existing building blocks and implementing only what is new.

FXB turns the compiled result into a portable, shareable bundle separate from the model
weights. A model can start without compiling on the serving host, and a bundle can be copied,
cached, or published with a model. Its architecture fingerprint also lets compatible
fine-tuned or weight-updated variants reuse the same FXB instead of recompiling.

Build a bundle with the dedicated CLI and serve it explicitly:

.. code-block:: bash

   fxb build <model> model.fxb
   furiosa-llm serve <model> --fxb model.fxb

For published ``furiosa-ai`` models, serving remains a one-command operation because the
model repository already contains a compatible FXB:

.. code-block:: bash

   furiosa-llm serve furiosa-ai/Qwen3-VL-2B-Instruct

The FXB runtime now supports intra-node pipeline parallelism, applies the runtime's automatic
I/O-memory estimate consistently, and uses bucket presets generated from compiler-validated
results. See :ref:`BuildingFXB` for build options and :ref:`ServingFromCachedFXB` for bundle
resolution and caching.


Faster Top-k and Top-p Sampling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Furiosa-LLM now accelerates CPU-side random sampling for ``top_k``, ``top_p``, and their
combination. Previously, sampling used a heap or a full sort over the vocabulary at every
decode step. The new implementation uses distribution-aware candidate pruning inspired by
Qrita: SIMD instructions compute statistics over the live BF16 logits, and a threshold derived
from a Gaussian approximation retains a small superset of the tokens needed for top-k or
top-p selection. The threshold is relaxed and selection is retried when necessary, ensuring
that candidate pruning cannot exclude the true selection.

Exact selection then operates only on the reduced candidate set. For top-p, each BF16 logit
and token index is packed into a single order-preserving integer key, replacing more expensive
float-and-index comparisons. Only a small prefix of candidates is partially ordered, and the
prefix grows when more probability mass is required. Vectorized gather and masking paths also
reduce overhead for top-k and for the common pipeline where top-k masks the logits before
top-p runs.

In the tested gpt-oss-20b and EXAONE 4.0 cases, the sampling operation itself was
**2.7–13.2× faster**. On the ShareGPT benchmark with ``top_k=50`` and ``top_p=0.9``,
output-token throughput improved by **6.9–16.9%** across the tested concurrency range while
mean TPOT fell by **6.5–15.4%**. Results depend on the model, logit distribution, and workload.


----

The complete list of changes in this release is summarized in the All Changes section below.


.. _Release2026_4_0_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Major Features & Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Hierarchical KV cache** (experimental)

  * Added a hierarchy-aware radix cache spanning NPU DRAM (L1) and host memory (L2), with
    asynchronous DMA offload, reload, capacity accounting, and on-demand eviction.
  * Added proactive write-through and optional eviction-time write-back policies so useful
    prefixes can survive NPU-cache reclamation.
  * Reloads can combine an existing local prefix head with a longer cached suffix, avoiding
    both redundant transfer and prefill recomputation.
  * Added auxiliary/sliding-window KV offload and reload, including alignment to sliding-
    window boundaries and all-sliding-window model support.
  * Enabled L1/L2 offloading with intra-node pipeline parallelism.
  * Added a **Mooncake-backed L3 tier** for cross-process and cluster-wide prefix reuse,
    including namespace isolation, write-through backup, full and partial-prefix prefetch,
    and asynchronous cross-process transfer.
  * Mooncake libraries are loaded at runtime with ``dlopen``. They are not linked into or
    bundled with the Furiosa-LLM wheel; a missing library degrades to L2-only operation.

* **Speculative decoding** (experimental)

  * Replaced the dual-scheduler prototype with a single FCFS scheduler that owns target and
    draft model lanes and verifies draft tokens in batches.
  * Added hidden ``--experimental-spec-model``, ``--experimental-spec-fxb``, and
    ``--experimental-spec-tokens`` serving options, plus matching ``LLM`` configuration.
  * Added tokenizer, model-interface, tensor-parallel, bucket-capacity, and I/O-memory
    validation for target/draft pairs.
  * Added initial self-speculation support for vision-language models with identical target
    and draft layouts.
  * Added optional MTP kernel compilation to EXAONE 4.5 FXB builds. ``fxb build`` enables it
    when the model preset and kernel plan support it; pass ``--no-enable-mtp`` to omit it.

* **Multimodal serving**

  * Moved Qwen3-VL text-only and image-and-text serving to the standard FCFS scheduler and
    DP routing path.
  * Added vision-encoder batching and separate encoder/decoder NPU task queues.
  * Added a two-phase acquire-and-encode/generate flow to reduce NPU idle time.
  * Added content-derived media IDs and an encoded-embedding registry, allowing repeated
    images to reuse both preprocessing results and NPU-generated vision embeddings.
  * Integrated multimodal token pinning with prefix-cache insertion and eviction.
  * Added multimodal input support to the RawModel API.
  * Fixed mRoPE offset computation, chunked-prefill aborts, and multimodal requests using
    repetition penalty, minimum length, and prompt-logprob sampling parameters.

* **Model support**

  * Added Qwen3-VL 2B and 4B, including Instruct and Thinking variants.
  * Added Qwen3-Embedding 0.6B/4B/8B and Qwen3-Reranker 0.6B/4B/8B FXB serving.
  * Added Harrier OSS v1 0.6B embeddings.
  * Added Mistral-Nemo-Instruct-2407-FP8.
  * Added dynamically quantized FP8 Llama 3.1 8B and Llama 3.3 70B variants.
  * Added BGE-M3, BGE-reranker-v2-m3,
    E5-Mistral-7B-Instruct, EXAONE 4.5 33B, and Gemma 4 31B


Engine Core & Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

* **Device runtime**

  * Migrated generator execution to **device-runtime2**, expressing compute and DMA as
    explicit stream operations with event-based dependencies.
  * Reduced stream-operation host overhead, removed a redundant host buffer pool in favor of
    the runtime's DMA-BUF interface, and eliminated unnecessary DMA synchronization brackets
    and compute-stream launch barriers.
  * Updated hierarchical device-ID handling and fixed device-runtime memory leaks and
    vision-language NaN failures through dependency updates.

* **Loading and memory efficiency**

  * Load model weights for DP instances in parallel.
  * Deduplicate shared ModelBuffer uploads such as RoPE caches across EDFs.
  * Generate warmup metadata about five times faster for large MoE models by reducing
    redundant scheduler-estimation quantization steps.
  * Let the runtime estimate I/O-buffer memory by default in both ``LLM`` and serving paths;
    explicit ``--max-io-memory-mb`` values remain available.
  * Size peer-DRAM windows from the device mesh and preserve explicit I/O-memory settings on
    the FXB path.

* **Scheduling and distributed serving**

  * Improved measured-cost decode batching and strict DP scheduling.
  * Added prefix-cache-aware admission estimates, on-demand eviction, and additional
    long-context radix-cache optimizations.
  * Added the initial PD-disaggregation connector lifecycle, admission signal, cancellation
    handling, and dedicated correctness tests. The public serving workflow remains
    experimental.
  * Enabled intra-node pipeline parallelism on the FXB path.

* **Sampling**

  * Accelerated CPU ``top_k``/``top_p`` sampling with distribution-aware candidate pruning
    and packed integer ordering.
  * Fixed BF16 cumulative-probability precision during top-p sampling.
  * Added NPU-sampling kernel compilation for selected gpt-oss and Qwen3-MoE FXB presets;
    execution integration is planned as follow-up work.


API & Usability Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Responses API**

  * Added ``/v1/responses`` support for gpt-oss through the Harmony output path, including
    multi-turn and tool-calling fixes.
  * Added ``structured_outputs`` to Responses requests, with explicit conflict errors when
    ``tool_choice``, ``response_format``, and ``structured_outputs`` compete.
  * Improved gpt-oss output refinement using xgrammar structural tags.

* **Structured output and reasoning**

  * Reasoning and final-content constraints now use independent structured-output states and
    may select different backends.
  * Fixed cancellation races involving structured grammars and the Python frontend.

* **Sampling API**

  * Added ``stop`` and ``include_stop_str_in_output`` to ``SamplingParams`` for online and
    offline generation. Stop-string matches report ``finish_reason="stop"`` and expose the
    matched value in ``stop_reason``.
  * Disabled tokenizer whitespace cleanup in incremental decoding so generated whitespace is
    preserved faithfully.

* **Serving and CLI**

  * ``--max-model-len`` now actively caps prompt-plus-generation context length, is reflected
    by ``/v1/models``, and can improve admission concurrency for requests that omit
    ``max_tokens``. See :ref:`ServeMaxModelLen`.
  * Added embedding, score, and rerank endpoint support for the expanded pooling-model set.
  * Lazy-loaded heavy Python dependencies, making help and version commands nearly
    instantaneous.
  * Improved FXB inspection and build summaries, including MTP and vision-encoder kernels.


FXB, Build, and Packaging
~~~~~~~~~~~~~~~~~~~~~~~~~

* Removed the legacy v2 model-artifact build and loading path and standardized execution on
  FXB.
* Removed ``furiosa-llm build``; ``fxb build`` is now the supported bundle compiler.
* Removed FXB-managed models from the duplicate legacy release-artifact group.
* Runtime bucket presets are now generated deterministically from compiler test results in a
  database, keyed by the pinned npu-tools revision and an explicit model allowlist.
* Added bucket presets and build coverage for the new generation, multimodal, embedding, and
  reranking architectures.
* Added ``aarch64`` packaging support to the unified build workflow.


Reliability & Correctness
~~~~~~~~~~~~~~~~~~~~~~~~~

* Fixed gpt-oss parsing in multi-turn tool-calling conversations and multi-turn Responses API
  requests.
* Fixed hierarchical-cache admission deadlocks, auxiliary-cache reload alignment, partial
  prefix reuse, and prefix-router mirror consistency.
* Fixed vision-language sampling panics and multimodal position-offset handling.
* Fixed stop-string cancellation cleanup and structured-output cancellation races.
* Improved telemetry export performance and fixed telemetry and device-runtime memory leaks.
* Added broader end-to-end, accuracy, multimodal MMMU, hierarchical-cache, PD, and FXB-based
  model coverage.


.. _Release2026_4_0_BreakingChanges:

🚨 Breaking Changes & Deprecations
----------------------------------

* **Legacy model artifacts and ``furiosa-llm build`` removed**:

  * The v2 artifact path based on ``furiosa-models`` and the model rewriter has been removed.
    Furiosa-LLM now loads models through the FXB runtime only.
  * **Migration**: Replace ``furiosa-llm build <model> ...`` with
    ``fxb build <model> <output>.fxb``. Serve a local bundle with
    ``furiosa-llm serve <model> --fxb <output>.fxb``, or use a ``furiosa-ai`` Hugging Face
    repository that already publishes a compatible FXB.

* **No-op serve-time bucket flags removed**:

  * ``furiosa-llm serve`` no longer accepts ``--max-batch-size``, ``--min-batch-size``,
    ``--max-prompt-len``, ``--prefill-buckets``/``-pb``, or
    ``--decode-buckets``/``-db``. These options were parsed but silently ignored because an
    FXB's bucket set is fixed at build time.
  * **Migration**: Remove these options from server launch scripts. Select buckets when
    running ``fxb build``. For runtime scheduling limits, use ``--max-concurrency`` and
    ``--max-num-batched-tokens`` as appropriate.

* **``--max-model-len`` is now enforced at serve time**:

  * Unlike the removed bucket-shaping flags, ``--max-model-len`` has been restored with real
    runtime semantics. A value that previously had no effect now caps the total context used
    by each request, including prompt and generated tokens. Values above the model or FXB
    limit fail at startup.
  * **Migration**: Audit existing launch scripts that already pass ``--max-model-len`` and
    choose the intended serving cap. Remove it to use the smallest model, FXB, and KV-capacity
    limit automatically.

* **Obsolete beam-search parameters removed**:

  * ``use_beam_search``, ``best_of``, ``length_penalty``, and ``early_stopping`` have been
    removed from ``SamplingParams`` and the compatible request models. Beam search has not
    been supported since SDK 2026.1.
  * **Migration**: Remove these fields and use greedy or random sampling parameters such as
    ``temperature``, ``top_k``, and ``top_p``.

* **Runtime bucket synchronization now uses the compiler results database**:

  * The contributor-facing bucket synchronization tool no longer reads source preset YAML
    from npu-tools. It queries validated results for a specific compiler revision and emits
    the checked-in ``share/buckets`` files.
  * **Migration**: Contributors running bucket synchronization locally need the database
    credentials used by the release workflow. Normal FXB builds and serving do not require
    database access.


📦 Released Components
----------------------

Python packages
^^^^^^^^^^^^^^^

The Furiosa-LLM packages released from this repository are:

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Package name
     - Version
     - Supported architectures
   * - furiosa-native-llm-common
     - 2026.4.0
     - ``x86_64``, ``aarch64``
   * - furiosa-llm-native
     - 2026.4.0
     - ``x86_64``, ``aarch64``
   * - furiosa-llm
     - 2026.4.0
     - pure Python (``any``)
   * - furiosa-torch-ext
     - 2026.4.0
     - pure Python (``any``)
   * - furiosa-tcc
     - 2026.4.0
     - pure Python (``any``)
