.. _HybridKVCache:

**********************************
Hybrid KV Cache Management
**********************************

Hybrid KV cache management is an internal optimization for models that mix global attention and sliding-window attention layers.
Instead of treating all layers as if they had the same memory behavior, Furiosa-LLM manages them with separate KV cache pools and coordinated allocation logic.

This feature is applied automatically when the model uses hybrid attention; no extra user configuration is required.

Overview
========

In hybrid models, global-attention layers and sliding-window layers have different cache growth patterns:

* **Global attention** grows with full sequence length
* **Sliding-window attention** is bounded by the window size

Using one undifferentiated pool can over-provision memory for sliding-window layers, especially for long-context workloads.
Hybrid KV cache management avoids this by separating the two memory paths.

How It Works
============

Pool Partitioning
-----------------

At initialization, Furiosa-LLM partitions KV cache memory into:

* A **global-attention pool**
* A **sliding-window attention pool**

Each pool is then assigned to the corresponding KV cache space for that attention type.
The partitioning logic is attention-aware, so memory is distributed according to expected global vs windowed usage instead of a one-size-fits-all split.

Request Lifecycle
-----------------

For each request phase, the scheduler coordinates both pools together:

* **Prefill**: allocates write blocks in global and sliding-window pools for incoming tokens
* **Extend / Decode**: loads existing cached blocks and allocates new write blocks for new tokens
* **Cleanup**: eagerly releases sliding-window blocks that moved out of the valid window, while keeping global blocks reusable for full-prefix history

This is a key efficiency point: sliding-window cache entries outside the active window are reclaimed early instead of occupying memory until request completion.

Interaction with Prefix Caching
-------------------------------

When prefix caching is enabled, hybrid cache management works with hybrid prefix matching to deduplicate both global and sliding-window blocks.
If part of a matched prefix no longer has valid sliding-window cache, Furiosa-LLM still reuses the valid portion and computes only what is needed.
For details on hybrid prefix-match behavior, see :ref:`Hybrid Attention Models <PrefixCachingHybridAttentionModels>` in :ref:`PrefixCaching`.

Why It Is Efficient
===================

Compared to a single pooled approach, hybrid KV cache management provides:

* **Lower memory waste** in models with sparse or mixed attention patterns
* **Higher effective cache capacity** for long-context global attention
* **Reduced eviction pressure** by reclaiming stale sliding-window blocks earlier
* **Stable serving behavior** without requiring users to manually tune per-attention memory pools

For end users, the main benefit is straightforward: better KV memory utilization and more consistent performance on hybrid-attention models, automatically.

Optional Tuning
===============

For hybrid-attention models, you can optionally set the ``EXPECTED_AVERAGE_SEQ_LENGTH`` environment variable to guide how much KV memory is reserved for global attention versus sliding-window attention.

This is useful when your workload has a stable prompt-length pattern (for example, consistently very long prompts), and you want memory partitioning to better match that pattern.

* If unset, Furiosa-LLM uses a default ratio based on the model's global-attention and sliding-window attention cache requirements.
* If set, Furiosa-LLM uses the provided expected sequence length to compute a more workload-aware split.
* The value must be a positive integer.

.. code-block:: bash

    export EXPECTED_AVERAGE_SEQ_LENGTH=8192
    furiosa-llm serve ...
