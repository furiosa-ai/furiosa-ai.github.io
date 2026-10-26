.. _PerformanceOptimization:

************************
Performance Optimization
************************

Furiosa-LLM provides settings to improve throughput and latency by tuning
request admission, decode batching, token sampling, and KV cache reuse.
Choose optimizations based on your workload and compare throughput, time to
first token (TTFT), and time per output token (TPOT) under representative load.

.. _ServeMaxModelLen:

Limiting the Context Length
===========================

Setting a context limit that matches your workload can increase concurrency
and throughput, especially for requests without an explicit output-token limit.

By default the server accepts requests up to the smallest of the model's
``max_position_embeddings``, the context length covered by the loaded FXB,
and what the KV cache can hold. ``--max-model-len`` lowers that limit:

.. code-block::

    furiosa-llm serve [MODEL_ID_OR_PATH] --max-model-len 8192

The value caps the total tokens a single request may use — prompt plus
generated tokens. It applies to every request, including those that do not set
``max_tokens``: the implicit generation budget shrinks with the cap as well. The
limit advertised on ``/v1/models`` is whichever of those four bounds is smallest,
so on a memory-tight deployment it can be the KV-derived one even with no cap
set. A request exceeding it is rejected with a context-length error.

The cap may only *lower* the limit. A value above either the model's
``max_position_embeddings`` or the context length covered by the loaded FXB
is rejected at startup, so a typo fails fast instead of being ignored.

By default, the scheduler decides whether to admit a request using its estimated
lifetime KV cache demand: prompt tokens plus the maximum output tokens, accounting
for prefix-cache hits. If neither the request nor the model's generation defaults
sets an output-token limit, the remaining context window becomes the output budget.
An unnecessarily large context limit can therefore reduce concurrency even when
actual responses are short.

For example, suppose the model, FXB, and KV cache support a **128K-token context**,
but your application needs at most **8K tokens per request**, including input and
output. The command above sets this 8K limit with ``--max-model-len 8192``.
For requests without an output-token limit, this reduces the scheduler's per-request
KV budget from 128K to 8K tokens before accounting for prefix-cache hits. The same
KV cache can then accommodate more concurrent requests, potentially increasing
throughput.

The benefit is limited when requests already specify an appropriate ``max_tokens``
or another constraint, such as ``--max-concurrency`` or NPU compute capacity,
limits throughput. The setting does not speed up individual kernel executions or
reduce allocated NPU memory; weights, compiled buckets, and the KV pool stay the same.

.. note::

  ``fxb build`` takes an option of the same name, but it means something different there:
  at build time ``--max-model-len`` shapes which buckets are compiled into the FXB. At
  serving time the bucket set is fixed by the loaded FXB, and this option only caps
  admission.

.. _ProfileGuidedDecodeBatching:

Profile-Guided Decode Batching
==============================

Profile-guided decode batching uses measured execution costs to choose decode
batch composition and compiled buckets. It can improve throughput and TPOT when
concurrent requests have different context lengths and batching choices affect
execution cost. On startup, Furiosa-LLM profiles NPU execution and input transfers
for different batch and KV-cache shapes, adding an initial profiling cost. It
caches these profiles and reuses them when compatible with the FXB and runtime.

The feature is enabled by default for generative models. Pooling models disable
it automatically because they have no decode phase. ``--batch-scheduling-budget-ms``
(default: ``1`` ms) sets the CPU search budget for the default decode-batch
optimizer. It is not a hard limit on total scheduling latency, and does not
bound the separate experimental aggressive-batching policy. The budget is
ignored when profile-guided batching is disabled. Start with the default budget;
increasing it allows a longer search for efficient batches but can add CPU
scheduling overhead. Compare throughput and TPOT before keeping a larger budget.

.. code-block:: bash

   furiosa-llm serve <model> \
     --profile-guided-decode-batching --batch-scheduling-budget-ms 1

Use ``--no-profile-guided-decode-batching`` to disable profiling and its use in
decode batching. The Python equivalents are
``SchedulerConfig.profile_guided_decode_batching`` and
``SchedulerConfig.batch_scheduling_budget_ms``. The
``profile_guided_decode_batching`` setting replaces ``enable_exec_stats``
without a compatibility alias; the budget setting retains its name.

Profile-guided batching selects batches using execution measurements;
:ref:`overlap scheduling <OverlapScheduling>` overlaps CPU preparation with NPU
execution. They can be used together for supported models. The separate
experimental aggressive-batching policy requires profile-guided batching and
cannot be combined with an explicit ``max_concurrency``.

Benefits depend on the workload's context lengths and available buckets. Compare
throughput, TPOT, TTFT, and startup profiling time with your own traffic. The
:ref:`2026.4 release notes <Release2026_4_0>` include an illustrative ON/OFF comparison.

.. _NPUSampling:

NPU Sampling
============

NPU sampling selects the next token on the NPU for supported generative models.
It helps when CPU token selection or logits transfers contribute significantly to
decode latency, especially with larger batches. Moving sampling to the NPU can
reduce TPOT and improve output throughput. FXB builds include sampling kernels for
every generative model, and the path is enabled by default. To sample on the host
instead:

.. code-block:: bash

   furiosa-llm serve <model> --no-enable-npu-sampling

For the Python API, set ``SchedulerConfig(enable_npu_sampling=False)``. Engine
startup fails if NPU sampling is enabled with an FXB that lacks the required
sampling kernel. Speculative decoding turns NPU sampling off with a warning.

The NPU sampler supports greedy and random sampling, including temperature,
``top_k``, ``top_p``, ``min_p``, repetition penalties, minimum token lengths, and
structured-output masks. Requests for output ``logprobs`` require transferring
and processing logits on the host, while token selection remains on the NPU.
The runtime can skip full-logit transfers when no request in the batch needs
logits on the host.

Experimental Overlap Scheduling
===============================

Overlap scheduling prepares the next batch on the CPU while the NPU executes
the current batch. Use it when CPU batch preparation leaves gaps between NPU
executions, particularly during sustained generation. It can improve throughput
and TPOT, but may increase TTFT for newly arriving requests.

The feature is experimental and disabled by default. Enable it with:

.. code-block:: bash

   furiosa-llm serve <model> --enable-overlap-scheduling

Keep it enabled when throughput and TPOT improve while TTFT remains within your
latency target. See :ref:`OverlapScheduling` for configuration and compatibility.

Experimental Hierarchical Prefix Caching
========================================

Hierarchical prefix caching (HiCache) extends prefix reuse beyond NPU memory
by offloading KV cache to host memory. Use it when requests repeatedly share long
prefixes and the reusable working set exceeds the NPU cache capacity. Avoiding
repeated prefill computation can reduce TTFT and improve throughput when reloading
KV data is cheaper than recomputing the prefix.

The feature is experimental and disabled by default. For example, enable a
process-wide 16 GiB host KV cache with:

.. code-block:: bash

   furiosa-llm serve <model> --experimental-kv-offload-host-memory-gb 16

A larger host cache can retain more reusable prefixes, at the cost of host memory
and transfer bandwidth. Benefits are limited when prompts rarely share prefixes
or the NPU cache already holds the working set. See :ref:`HierarchicalPrefixCaching`
for host-memory sizing and retention policies.
