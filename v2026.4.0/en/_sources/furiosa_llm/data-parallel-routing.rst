.. _DataParallelRouting:

**********************************
Data-Parallel Routing
**********************************

When ``data_parallel_size`` is greater than 1, Furiosa-LLM runs multiple
data-parallel (DP) replicas of the same model. The DP router chooses which
replica receives each new request.

Furiosa-LLM uses scoring-based DP routing by default. It is designed to keep
requests near useful prefix-cache state while avoiding replicas that already
have a larger token workload.

How Scoring Works
=================

For each live DP replica, the router computes two signals:

* **Prefix locality**: how much of the new prompt matches that replica's
  prefix cache, as a fraction of the prompt length.
* **Load pressure**: the number of active context tokens already routed to
  that replica. Prompt tokens are charged when the request is routed, and
  generated tokens are charged until the request completes.

The router normalizes lower load pressure into a higher load score, combines it
with the prefix-locality score, and sends the request to the replica with the
highest final score. Exact ties keep the first live DP replica.

.. code-block:: text

   final_score = prefix_weight * prefix_score + load_weight * load_score

The prefix-locality signal is based on a lightweight virtual prefix cache that
tracks each DP replica's KV-cache events. It does not move KV cache between
replicas. Since prefix caches are DP-local, routing a request to a replica also
affects where later requests can get cache hits.

Routing Policies
================

Use ``--data-parallel-routing-policy`` to choose the routing policy:

.. list-table::
   :header-rows: 1

   * - Policy
     - Behavior
   * - ``scoring``
     - Default. Balances prefix locality and token-load pressure.
   * - ``round-robin``
     - Sends requests to DP replicas in order. This can be useful for simple
       benchmarking or when cache-aware routing is not desired.

Scoring Profiles
================

When using ``scoring``, use ``--data-parallel-scoring-profile`` to tune the
prefix-locality/load tradeoff:

.. list-table::
   :header-rows: 1

   * - Profile
     - Prefix/load weights
     - Use when
   * - ``balanced``
     - ``0.55 / 0.45``
     - Default. Use for most serving workloads when workload characteristics are mixed or unknown.
   * - ``locality``
     - ``0.90 / 0.10``
     - Prefer this for workloads with repeated system
       prompts, multi-turn conversations, or RAG-style document prefixes.
   * - ``load``
     - ``0.10 / 0.90``
     - Prefer this when prompts rarely share prefixes, or when balanced
       compute load matters more than cache locality. Extremely long shared
       system prompts or document prefixes can overload one DP replica when
       locality is prioritized.

If prefix caching is disabled, Furiosa-LLM falls back to the ``load`` profile
because prefix locality no longer provides useful signal. The old
``prefix-aware`` profile name has been removed; use ``locality`` instead.

Configuration
=============

The default behavior is scoring-based routing with the ``balanced`` profile:

.. code-block:: bash

   furiosa-llm serve <model> --data-parallel-size 2

To bias routing toward prefix-cache reuse:

.. code-block:: bash

   furiosa-llm serve <model> \
     --data-parallel-size 2 \
     --data-parallel-routing-policy scoring \
     --data-parallel-scoring-profile locality

To use round-robin routing:

.. code-block:: bash

   furiosa-llm serve <model> \
     --data-parallel-size 2 \
     --data-parallel-routing-policy round-robin

The scoring profile only affects ``scoring``. Do not set a non-default scoring
profile with ``--data-parallel-routing-policy round-robin``.

Python API
==========

The same settings can be passed through ``SchedulerConfig``:

.. code-block:: python

   from furiosa_llm import LLM
   from furiosa_llm.metadata.config_types import (
       DataParallelRoutingPolicy,
       DataParallelScoringProfile,
       SchedulerConfig,
   )

   scheduler_config = SchedulerConfig(
       data_parallel_routing_policy=DataParallelRoutingPolicy.SCORING,
       data_parallel_scoring_profile=DataParallelScoringProfile.LOCALITY,
   )

   with LLM(
       "furiosa-ai/Qwen3-4B-FP8",
       data_parallel_size=2,
       scheduler_config=scheduler_config,
   ) as llm:
       outputs = llm.generate(["Hello"])

Operational Notes
=================

* DP routing happens inside one Furiosa-LLM process. For routing across
  multiple model-server pods, see :doc:`llm-d </cloud_native_toolkit/llm_d>`.
* Scoring-based routing works best with prefix caching enabled. For details,
  see :ref:`PrefixCaching`.
* For background on DP itself, see :ref:`ModelParallelism`.
