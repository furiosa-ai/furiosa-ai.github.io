.. _HierarchicalPrefixCaching:

**********************************
Hierarchical Prefix Caching
**********************************

Hierarchical prefix caching extends :ref:`PrefixCaching` with a host-memory
cache tier. It allows Furiosa-LLM to preserve more reusable KV cache than can fit
in NPU memory, which is useful for workloads with long, repeated prefixes and a
working set larger than the NPU KV cache.

This document describes the local two-tier cache:

* **NPU cache (L1)** provides the lowest-latency prefix reuse, but has limited
  capacity and is also needed by active requests.
* **Host cache (L2)** provides additional capacity in system memory. A matching
  host-resident prefix is copied back to NPU memory before it is reused.

Each data-parallel replica owns an independent NPU and host cache within the
Furiosa-LLM process. Cache contents are neither persistent nor shared between
server processes.

.. warning::

   Hierarchical prefix caching is experimental. Its CLI options therefore use
   an ``--experimental-`` prefix and do not appear in ``furiosa-llm serve
   --help``. Option names and behavior may change in a future release.

When to Use It
==============

Hierarchical prefix caching is most effective when all of the following are
true:

* Requests repeatedly use long, token-identical prefixes.
* The reusable prefix working set is too large to remain in NPU memory.
* Recomputing an evicted prefix costs more than copying its KV cache from host
  memory.
* The host has enough free memory and bidirectional host-NPU bandwidth for the
  configured cache.

Typical examples include repeated system prompts, multi-turn conversations,
RAG requests that reuse document prefixes, and applications with a collection
of large prompt templates.

It may provide little benefit when prompts rarely share a prefix, shared
prefixes are short, or NPU prefix-cache capacity is already sufficient. A host
cache hit is normally slower than an NPU cache hit because the KV data must be
reloaded, but it can be substantially cheaper than recomputing a long prefix.
Measure the result with a representative request distribution before deploying
it broadly.

How It Works
============

Furiosa-LLM continues to match prefixes by token ID from the beginning of each
prompt. Enabling hierarchical caching changes where a matched prefix may
reside, not what constitutes a match.

Conceptually, a cached prefix can be in one of three states:

* **NPU only**: immediately reusable.
* **Host only**: retained outside NPU memory and reloaded on the next hit.
* **NPU and host**: immediately reusable, with a host copy that can survive NPU
  eviction.

For a new request, the scheduler finds the longest safe token-exact match. The
beginning of that match may already be NPU-resident, while a following portion
is host-resident. Furiosa-LLM reuses the NPU portion, reloads the host portion,
and computes only the remaining unmatched prompt suffix.

.. code-block:: text

   New prompt:       [       matching prefix       ][ new suffix ]
   Cache residency:  [ NPU-resident ][ host-only   ][ not cached ]
   Request action:   [ reuse         ][ reload     ][ compute    ]

Host and NPU capacity are managed automatically. Entries used by active
requests are protected. Under NPU memory pressure, inactive cached entries may
lose their NPU copy; whether they remain in host memory depends on the
retention policy described below. Under host memory pressure, older eligible
host entries are evicted to make room for more useful cache content.

Host Retention Policies
-----------------------

Hierarchical caching provides two complementary ways to create a host copy.

Write-through promotion
~~~~~~~~~~~~~~~~~~~~~~~

Write-through promotion identifies repeatedly reused prefixes while they are
still NPU-resident. When a prefix reaches the configured hit threshold,
Furiosa-LLM proactively copies it to host memory. The prefix then has both NPU
and host copies, so a later NPU eviction can retain the host copy without an
eviction-time transfer.

This is the default policy. The default threshold is ``2``, and write-back is
disabled. Consequently, frequently reused prefixes are preserved in host
memory while NPU-only prefixes that have not reached the threshold can be
discarded during eviction. This is a conservative starting point because it
limits host traffic and host-cache pollution.

The initial write to the NPU prefix cache does not count as a hit. A threshold
of ``1`` therefore promotes a prefix on its first subsequent matching request,
while the default threshold of ``2`` promotes it on the second.

Increase the threshold to admit only hotter prefixes. Set the threshold to
``0`` to disable write-through promotion.

.. note::

   Support for a write-through threshold of ``0`` as an immediate-promotion
   mode is planned. In that mode, Furiosa-LLM will copy a prefix to the host tier
   when it is first written to the NPU prefix cache. In the current experimental
   release, ``0`` disables write-through promotion; it does not request an
   immediate host copy.

Write-back on NPU eviction
~~~~~~~~~~~~~~~~~~~~~~~~~~

When write-back is enabled, Furiosa-LLM attempts to copy an NPU-only cached
prefix to host memory when NPU memory pressure evicts it. This preserves a
broader set of prefixes, including prefixes that did not reach the
write-through threshold.

Write-back can improve reuse for a large or less predictable working set, but
it also increases NPU-to-host traffic and can fill the host cache with prefixes
that are never used again. Enable it when reuse after NPU eviction is common
and benchmarks show that the additional transfers are worthwhile.

Write-through promotion remains active when write-back is enabled unless its
threshold is set to ``0``.

Configuration
=============

Hierarchical prefix caching is disabled by default. Supplying one of the two
positive host-capacity options enables it.

.. list-table:: CLI options
   :header-rows: 1
   :widths: 36 14 50

   * - Option
     - Default
     - Description
   * - ``--experimental-kv-offload-host-memory-gb FLOAT``
     - Disabled
     - Enables hierarchical caching with a process-wide host KV cache budget in
       GiB. The budget is divided among scheduler instances and, for hybrid
       attention models, their cache types.
   * - ``--experimental-kv-offload-host-memory-ratio FLOAT``
     - Disabled
     - Enables hierarchical caching with host capacity for each scheduler and
       cache type sized as a ratio of its NPU KV block count. For example,
       ``0.5`` requests half as many host blocks as NPU blocks.
   * - ``--experimental-kv-offload-write-through-threshold INTEGER``
     - ``2``
     - Minimum prefix-cache hit count for write-through promotion. ``0``
       disables promotion. Negative values are invalid.
   * - ``--experimental-kv-offload-write-back``
     - Disabled
     - Preserves NPU-only cached prefixes in host memory when NPU eviction
       occurs, subject to host capacity.

The two host-capacity options are mutually exclusive. A host-memory ratio of
``0`` leaves hierarchical caching disabled; a negative ratio is invalid. The
GiB value must be positive and large enough to hold at least one usable KV
cache block after the process budget is divided.

Choosing a Host Capacity
------------------------

For most deployments, prefer the GiB option because it establishes a
predictable upper bound for the process's host KV cache allocation:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-gb 16

The budget applies to the entire Furiosa-LLM process. With data parallelism it
is divided among data-parallel scheduler instances. For a hybrid-attention
model, each instance's share is further divided between global-attention and
sliding-window cache according to their NPU KV memory footprints. Capacity is
allocated in whole blocks, so the actual allocation can be slightly smaller
than the requested limit.

The ratio option is convenient when host capacity should scale with the model's
NPU KV cache:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-ratio 0.5

The ratio is applied independently to every scheduler instance. Therefore,
total process host-memory use grows with ``data_parallel_size``. A ratio of
``1.0`` requests approximately one host KV block for every NPU KV block in each
cache type; it is not a process-wide memory limit.

The configured budget covers the host KV cache pools, not the model weights,
runtime state, request data, or other process memory. Leave sufficient physical
memory for those components and for the operating system.

CLI Examples
------------

Preserve prefixes that receive repeated hits, using the default promotion
threshold and no eviction-time write-back:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-gb 16

Preserve a wider working set by writing NPU-only cache entries to host during
eviction:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-gb 32 \
     --experimental-kv-offload-write-back

Admit only prefixes that have received at least four cache hits:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-ratio 0.5 \
     --experimental-kv-offload-write-through-threshold 4

Promote prefixes after their first cache hit and preserve colder NPU-only
entries during eviction:

.. code-block:: bash

   furiosa-llm serve <model> \
     --experimental-kv-offload-host-memory-gb 32 \
     --experimental-kv-offload-write-through-threshold 1 \
     --experimental-kv-offload-write-back

Python API
----------

The offline Python API exposes the same behavior through ``SchedulerConfig``.
The following example uses a 16 GiB process-wide host budget and the default
hot-prefix promotion policy:

.. code-block:: python

   from furiosa_llm import LLM
   from furiosa_llm.metadata.config_types import (
       KVOffloadConfig,
       PrefixCacheConfig,
       SchedulerConfig,
   )

   scheduler_config = SchedulerConfig(
       prefix_cache_config=PrefixCacheConfig(enabled=True),
       kv_offload_config=KVOffloadConfig(
           enabled=True,
           host_memory_bytes=16 * 1024**3,
           write_through_threshold=2,
           write_back=False,
       ),
   )

   with LLM(
       "furiosa-ai/Qwen3-8B-FP8",
       scheduler_config=scheduler_config,
   ) as llm:
       first = llm.generate(
           "A long shared document or instruction prefix...\nQuestion one"
       )
       second = llm.generate(
           "A long shared document or instruction prefix...\nQuestion two"
       )

To size the host tier relative to NPU KV capacity, set
``host_memory_ratio`` instead of ``host_memory_bytes``:

.. code-block:: python

   kv_offload_config = KVOffloadConfig(
       enabled=True,
       host_memory_ratio=0.5,
       write_through_threshold=4,
       write_back=True,
   )

Do not set both ``host_memory_bytes`` and a positive
``host_memory_ratio``. In the Python API, ``enabled=True`` must be set
explicitly; setting a capacity field alone does not enable offloading. Use
``write_through_threshold=None`` to disable write-through promotion in a
direct ``KVOffloadConfig``. The CLI uses ``0`` for the same purpose.

Feature Interactions and Limitations
====================================

Prefix caching
--------------

Hierarchical caching requires ordinary prefix caching. Prefix caching is
enabled by default, so no additional option is normally necessary. Do not
combine a host-capacity option with ``--no-enable-prefix-caching``; the server
cannot initialize hierarchical caching in that configuration.

Data parallelism
----------------

Each data-parallel replica has its own NPU and host prefix cache. Cache entries
are not copied between replicas. A process-wide GiB budget is divided among the
replicas, while a ratio-based capacity is applied to each replica and therefore
increases total host allocation as replicas are added.

Cache-aware routing can improve reuse by sending related prompts back to the
replica that already owns their cache entries. See :ref:`DataParallelRouting`
and consider the ``locality`` scoring profile for workloads dominated by large
shared prefixes.

Hybrid attention
----------------

Hierarchical caching supports models that combine global attention and
sliding-window attention. Furiosa-LLM allocates host capacity for both cache
types and reloads only a prefix for which the required sliding-window state is
valid. Consequently, the safely reusable prefix can be shorter than the raw
token match. See :ref:`PrefixCachingHybridAttentionModels` for the matching
semantics and :ref:`HybridKVCache` for the underlying memory model.

Other limitations
-----------------

* Speculative decoding is currently incompatible with prefix caching and,
  consequently, with hierarchical prefix caching.
* Prefix reuse is token-exact. Whitespace, punctuation, chat-template changes,
  or tokenizer changes can prevent reuse.
* Host cache contents are lost when the process exits and are not shared with
  another server process.
* The host cache increases capacity, not NPU capacity. A host-resident prefix
  still needs NPU blocks before inference can continue.

Host Preparation
================

Host KV cache pools are allocated when the engine starts. Engine startup fails
if the requested pool cannot be allocated or if the resulting process share is
too small to provide usable global-attention capacity.

On Linux, reserve enough hugepages for the host KV cache. Furiosa-LLM can fall
back to regular 4 KiB pages when the reserved hugepage pool is insufficient,
but this can reduce transfer throughput. The server emits a warning that
includes the required hugepage count when it detects insufficient reservation.
See :ref:`HostTuning` for system configuration instructions.

Monitoring and Tuning
=====================

Use the ``GET /metrics`` endpoint described in
:ref:`MonitoringOpenAICompatibleServer` and server logs to compare the
following before and after enabling hierarchical caching:

* ``furiosa_llm_prefix_cache_hits``: cumulative cached tokens reused.
* ``furiosa_llm_prefix_cache_queries``: cumulative prompt tokens considered for
  prefix matching.
* Time to first token (TTFT), especially after enough cache churn to exceed NPU
  cache capacity.
* Host memory consumption, NPU utilization, request queueing, and end-to-end
  latency under the expected concurrency.

The prefix-cache counters combine NPU-resident and host-resident hits; they do
not currently identify the cache tier. A higher combined hit rate therefore
does not by itself prove that host offload improved latency. Compare TTFT and
throughput on a warm, steady-state workload that forces NPU cache eviction.

.. note::

   Host- and NPU-specific metrics are planned to make troubleshooting and tuning
   hierarchical caching for different workload characteristics easier.

A practical tuning sequence is:

1. Start with a bounded GiB budget, the default threshold of ``2``, and
   write-back disabled.
2. Confirm that the workload has repeated long prefixes and that cache reuse
   improves after NPU-cache churn.
3. Increase the host budget if valuable prefixes are still lost and the host
   has sufficient memory.
4. Raise the promotion threshold if one-off prefixes consume too much host
   capacity or transfer bandwidth.
5. Test write-back if useful prefixes are evicted before they become hot enough
   for write-through promotion.
6. Retest at production concurrency, because transfer contention and cache
   residency patterns change with load.

Troubleshooting
===============

Server fails to start
---------------------

Check that exactly one positive host-capacity option is set, prefix caching is
enabled, the configured process budget is large enough after data-parallel
division, and sufficient host memory is available. For a ratio, use a value
greater than ``0``. For a write-through threshold, use ``0`` or a positive
integer.

High hit rate but limited latency improvement
---------------------------------------------

The published hit rate combines NPU and host hits. Host hits include reload
cost, and short-prefix reloads may not save enough computation to offset that
cost. Compare long-prefix requests separately and check for host-memory or DMA
bandwidth contention.

Low hit rate
------------

Verify that the fully rendered prompts are token-identical at the beginning,
including their chat templates and system messages. With data parallelism,
use cache-aware routing so related requests return to the same replica. Also
confirm that the workload has warmed the cache and that the host budget is
large enough for its reusable working set.

Host memory grows more than expected
------------------------------------

The ratio option is per scheduler, not process-wide. Total allocation grows
with data-parallel replicas and includes the cache requirements of each
attention type. Use the GiB option when a process-wide bound is required, and
remember that the rest of Furiosa-LLM also consumes host memory.

Related Topics
==============

* :ref:`PrefixCaching` describes token-exact prefix matching and the
  NPU-resident cache.
* :ref:`HybridKVCache` explains the cache layout for hybrid-attention models.
* :ref:`DataParallelRouting` explains how request routing affects cache
  locality across data-parallel replicas.
* :ref:`HostTuning` covers hugepage and host-transfer tuning.
* :ref:`MonitoringOpenAICompatibleServer` describes the server metrics
  endpoint.
