.. _Release2026_2_0:

********************************************
Furiosa SDK Release 2026.2
********************************************

We are happy to announce the release of **Furiosa SDK 2026.2**.

Building on the foundation laid in 2026.1, this release focuses on making production
deployments faster, easier to configure, and easier to scale out. We have improved
serving performance for **Qwen3** and **Exaone4**, removed most of the bucket-tuning
ceremony from artifact builds via **preset-based configuration**, introduced an
independent **Data Parallel (DP) Router** with a **prefix-aware** variant, enabled
**prefix caching by default**, and shipped the first phase of the **Response API
(/v1/responses)**.

If you are upgrading from 2026.1, please also read the
:ref:`Release2026_2_0_BreakingChanges` section and the :ref:`UpgradeGuide`.


Highlights
----------

Qwen3 and Exaone4 Performance Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Two of the most frequently deployed model families on RNGD see meaningful throughput
improvements in this release, with per-request latencies held at 2026.1 levels.
**Qwen3** artifacts ship with expanded prefill, decode, and append buckets (including
a 64k append bucket for Qwen3-32B-FP8) so more of the real-world request distribution
hits well-tuned kernels, and **Exaone4** gains more high-batch and append buckets
along with :ref:`Hybrid KV Cache Management <HybridKVCache>`, which splits KV memory
into separate global-attention and sliding-window pools and reclaims sliding-window
blocks as soon as they fall outside the active window — reducing memory waste and
lifting effective concurrency on long-context workloads.

Across a sweep of input length, output length, and concurrency, tokens/second
throughput improved by **74.9% on average over 2026.1**, rising to **84.8%** in the
low-concurrency regime (concurrency below 64), while TTFT and TPOT remain in line
with the 2026.1 baseline.

Per-Model Bucket Presets
^^^^^^^^^^^^^^^^^^^^^^^^

Good bucket configuration — the set of prefill, decode, and append bucket sizes
an artifact is compiled for — is one of the highest-leverage knobs for serving
performance, and also one of the hardest to tune by hand. In 2026.2,
``ArtifactBuilder`` absorbs that complexity: every supported model ships with
**per-model bucket presets**, tuned by the Furiosa team to match each model's
architecture and cover its full maximum context, and applied automatically at
build time.

The design goal is simple — **the default build should produce the best artifact
for typical serving workloads**, without requiring the user to reason about
bucketization at all:

.. code-block:: bash

   # Build an artifact for Qwen3-32B-FP8 and write it to ./Qwen3-32B-FP8
   furiosa-llm build Qwen/Qwen3-32B-FP8 -tp 8 ./Qwen3-32B-FP8

Explicit bucket arguments are still supported and take precedence, so workload-
specific tuning remains available for users who need it.


Data Parallel Router with Prefix-Aware Routing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2026.2 introduces a first-class **Data Parallel (DP) Router** for horizontally scaled
deployments. The router sits in front of the frontend and dispatches each request to
a DP replica before any engine-local scheduling happens, so clients see a single
entry point while each replica runs its own scheduler independently.

Two routing policies are available. ``round_robin`` distributes requests evenly
across replicas; ``prefix_aware`` inspects the tokenized prefix and prefers the
replica that already holds matching prefix cache entries — important for workloads
with shared leading tokens, such as chatbots with long system prompts or RAG systems,
where naive round-robin scatters shared prefixes across replicas and destroys cache
locality. The policy is selected via ``--data-parallel-routing-policy``, and defaults
to ``prefix_aware`` when prefix caching is enabled and ``round_robin`` otherwise:

.. code-block:: bash

   # Explicitly use prefix-aware routing
   furiosa-llm serve <model> --data-parallel-size <N> \
       --data-parallel-routing-policy <policy>

Prefix Caching Is Now On by Default
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prefix caching was introduced in 2026.1 behind an opt-in flag. After a release of
production use and several correctness and performance improvements, it is now
**enabled by default** in 2026.2. You no longer need any flag to benefit from it.

Alongside the default flip, we landed several refinements:

* **Prefix cache hit deferral** maximizes the cache hit rate by briefly deferring
  requests that are about to match an in-flight prefix, instead of kicking them off to
  a fresh prefill.
* **Prefix caching is applied to decoded outputs** as well, so multi-turn
  conversations benefit from the cache even on the continuation side.

If you need to disable prefix caching for a specific deployment — for example, for
workloads that rarely share prefixes across requests, or for benchmarking runs where
cache-hit variance would distort the measurements — pass ``--no-enable-prefix-caching``:

.. code-block:: bash

   furiosa-llm serve <model> --no-enable-prefix-caching

Response API (Phase 1)
^^^^^^^^^^^^^^^^^^^^^^

2026.2 adds initial support for the `Response API
<https://developers.openai.com/api/reference/responses/overview>`_ at
``/v1/responses``. The Response API is OpenAI's newer, more expressive alternative to
Chat Completions, and this release covers enough of the surface area to be useful for
straightforward request/response flows.

This is labeled **Phase 1** intentionally: the endpoint accepts requests and produces
well-formed responses, but a number of advanced behaviors are still being rolled out.

A minimal example using the OpenAI Python client:

.. code-block:: python

   from openai import OpenAI

   client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

   response = client.responses.create(
       model="Qwen/Qwen3-32B-FP8",
       input="Tell me a three-sentence bedtime story about a unicorn.",
   )
   print(response.output_text)

For the full endpoint list, supported parameters, multi-turn conversations via
``previous_response_id``, streaming, tool calling, and structured output, see
:ref:`ResponsesAPI`.

----

The complete list of changes in this release is summarized in the All Changes section
below.


.. _Release2026_2_0_AllChanges:

All Changes
-----------

Major Features & Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Data Parallel (DP) Router**

  * Introduced a dedicated DP router that schedules requests across DP replicas independently,
    enabling horizontal scale-out of inference workers with fine-grained control over request
    distribution.
  * **Prefix-aware routing** dispatches requests with shared prefixes to replicas that already
    hold matching prefix cache entries, maximizing cross-request cache hit rate in multi-tenant
    deployments.
  * The DP router is placed in front of the frontend so that routing decisions happen before any
    engine-local scheduling, reducing cross-replica contention.

* **Prefix Caching Enabled by Default**

  * Prefix caching is now enabled by default for all deployments, delivering lower
    Time-To-First-Token (TTFT) out of the box.
  * Additional refinements include **prefix cache hit deferral** to maximize cache hit rate,
    applying prefix caching to decoded outputs, and guaranteeing auxiliary block length
    equivalence for safe prefix cache insertion in hybrid attention models.

* **Response API and OpenAI-compatible API**

  * **Response API (/v1/responses)**: Added support for the Response API, enabling
    stateful interactions.
  * **Tokenize / Detokenize API**: Added ``/tokenize`` and ``/detokenize`` endpoints for
    server-side tokenization and detokenization.
  * **Embeddings ``dimensions`` parameter**: Support for requesting a specific embedding
    dimensionality via the ``dimensions`` field.

* **Broader Platform Support**

  * **ARM64 hosts (AArch64)**: The Furiosa SDK now runs on ARM64-based servers, with
    AArch64 SIMD support in the host runtime so the vectorized optimizations
    introduced in 2026.1 apply on ARM just as they do on x86-64.
  * **Red Hat Enterprise Linux (RHEL)**: RPM packages are now published, extending
    first-class OS support beyond Debian/Ubuntu to RHEL-based distributions.


Performance & Efficiency Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Model-specific optimizations**:

  * Expanded and re-tuned prefill, decode, and append buckets for **Qwen3** and
    **Exaone4** so more of the real-world request distribution hits well-tuned
    kernels, combined with compiler optimized for lower-batch buckets.

* **Optimized logit copy I/O**:

  * Reduced the memory bottleneck in loading model outputs for sampling by copying
    only the slice actually consumed by the sampler, cutting host-device transfer
    volume per decode step.

* **Scheduler optimizations**:

  * **Prefix cache hit deferral** maximizes cross-request cache hit rate by briefly
    deferring requests that are about to match an in-flight prefix.
  * Further optimization of the **prefix-aware DP router** keeps the routing
    decision fast even for very long shared prefixes.


API & Usability Improvements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Serving Options**:

  * Added ``--served-model-name`` option to customize the model ID exposed via the
    OpenAI-compatible API (useful for aliasing or drop-in replacements).
  * Added ``--chat-template-content-format`` argument for multi-part chat content rendering.

* **Sampling Defaults**:

  * Default sampling parameters can now be sourced from the model's
    ``generation_config.json``, aligning server behavior with the model's recommended defaults.

* **Usage & Observability**:

  * Usage response is augmented with **cached_tokens** and prompt/completion token details.
  * ``sampling_done`` metric now includes the device ID for per-device breakdown.

* **CLI**:

  * New ``furiosa-llm version`` subcommand reports the installed SDK version.

* **Chat Completion API**:

  * Request token length is validated against the maximum available KV blocks, preventing
    requests that cannot fit even in an empty cache.


.. _Release2026_2_0_BreakingChanges:

🚨 Breaking Changes & Deprecations
----------------------------------

* **--prefill-chunk-size Removed**:

  * The ``--prefill-chunk-size`` CLI option has been removed. Prefill chunking is now managed
    internally by the scheduler.
  * **Migration**: Remove ``--prefill-chunk-size`` from your launch scripts; no replacement is
    required.

* **PyTorch Upgrade to 2.10**:

  * PyTorch dependency bumped from 2.7 to 2.10.
  * **Migration**: Ensure your environment supports PyTorch 2.10.x.

* **reasoning_content Deprecated**:

  * The ``reasoning_content`` field in chat completions responses is being renamed to
    ``reasoning`` for consistency with upstream conventions. Both fields are populated
    in 2026.2, so existing clients continue to work unchanged; ``reasoning_content``
    will remain available for at least one more release cycle before removal.
  * **Migration**: Update client code to read the ``reasoning`` field before the
    removal lands.

* **guided_decoding Renamed to structured_output**:

  * The ``guided_decoding`` parameter group has been renamed to ``structured_output``.
  * **Migration**: Update any request payloads or SDK calls that reference
    ``guided_decoding``.


📦 Released Components
----------------------

Python packages
^^^^^^^^^^^^^^^

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-native-runtime
     - 2026.2.0
   * - furiosa-llm
     - 2026.2.0
   * - furiosa-torch
     - 2026.2.0

APT packages
^^^^^^^^^^^^

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Package name
     - Version
   * - furiosa-libsmi
     - 2026.1.1
   * - furiosa-smi
     - 2026.1.1
   * - furiosa-metrics-exporter
     - 2026.1.0
   * - furiosa-cdi
     - 2026.1.0

Docker images
^^^^^^^^^^^^^

.. list-table::
   :widths: 200 50
   :header-rows: 1

   * - Image
     - Tag
   * - furiosaai/furiosa-device-plugin
     - 2026.1.0
   * - furiosaai/furiosa-dra-driver
     - 2026.1.0
