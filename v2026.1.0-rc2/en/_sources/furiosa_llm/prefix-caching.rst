.. _PrefixCaching:

**********************************
Prefix Caching
**********************************

Prefix caching is a performance optimization technique that reuses previously computed KV cache entries for common prompt prefixes. This significantly reduces latency and computational costs for requests that share the same prompt beginning.

Overview
========

When processing multiple requests with similar prefixes (e.g., system prompts, common instructions, or shared context), prefix caching eliminates redundant computation by:

1. **Storing** KV cache entries from previous computations
2. **Matching** new prompts against cached prefixes
3. **Reusing** matching cache entries instead of recomputing them
4. **Computing** only the non-cached portion of the prompt

This can dramatically improve performance for scenarios with:

* Repeated system prompts across multiple user queries
* Common instruction templates
* Shared conversation history in multi-turn dialogues
* Document-based question answering where the document prefix is constant

How It Works
============

Automatic Management
--------------------

The prefix cache is managed automatically by the scheduler:

* **Cache Population**: KV cache entries are stored after each forward pass
* **Cache Matching**: New requests are checked against the cache tree
* **Cache Eviction**: Least recently used, non-active entries are evicted when memory is needed (starting from leaf suffixes)
* **Cache Invalidation**: No manual invalidation required

Token-Level Matching
--------------------

Furiosa-LLM's prefix caching operates at the token level, using a radix tree data structure for efficient prefix matching:

.. code-block:: text

    Request 1: [S1] + [A]
           │
           │    ┌──────────────┐   ┌──────────────┐
           └──► │ new compute  │ + │ new compute  │
                └──────────────┘   └──────────────┘
           Cache: ROOT ── [S1] ── [A]

    Request 2: [S1] + [B]
           │
           │    ┌──────────────┐   ┌──────────────┐
           └──► │  cache hit   │ + │ new compute  │
                └──────────────┘   └──────────────┘
           Cache: ROOT ── [S1] ──┬── [A]
                                 └── [B]

In the standard radix-cache flow (non-hybrid attention), Furiosa-LLM:

* Finds the **longest token-exact prefix** from the start of the prompt
* Reuses all matched KV blocks from that prefix
* Computes and caches only the unmatched suffix

If a request diverges in the middle of an existing branch, the cache can split that branch internally so future requests can still reuse the shared portion precisely.

.. _PrefixCachingHybridAttentionModels:

Hybrid Attention Models
-----------------------

Prefix caching fully supports hybrid attention models that use both global attention and sliding-window attention.
For these models, a prefix is reusable only when both cache types are valid for the same matched range:

* Tokens match from the beginning of the prompt
* Global KV cache entries are present for that range
* Sliding-window KV entries are available as a valid contiguous window for the current position

If the token match continues but the sliding-window requirement is no longer satisfied, Furiosa-LLM reuses only the longest safe prefix and computes the rest.
This keeps results correct while still maximizing reuse.

In practice, this means cache hits in hybrid models may be shorter than raw token overlap, especially when recent-window context differs across requests or has been evicted.

This behavior is also affected by eviction policy. For sliding-window attention, Furiosa-LLM intentionally allows sliding-window blocks to be evicted independently of global blocks, including at intermediate (non-leaf) prefix-tree nodes. 
The rationale is memory efficiency: blocks outside the frequently reused window are treated as stale and can be reclaimed earlier to make room for more useful cache content. However, under high memory pressure, this might cause a scenario where token-prefix matching continues while sliding-window validity ends sooner; in that case, Furiosa-LLM safely reuses the longest prefix that still has a valid window.

Performance Benefits
====================

Latency Reduction
-----------------

Prefix caching can provide significant latency improvements:

* **50-90% reduction** in time-to-first-token (TTFT) for requests with long shared prefixes
* **Proportional improvement** based on the length of the matched prefix
* **No impact** on generation quality or accuracy

Example Scenarios
-----------------

**Scenario 1: System Prompt Reuse**

.. code-block:: python

    system_prompt = "You are a helpful assistant specialized in Python programming..."  # 200 tokens

    # First request: Full computation
    response1 = llm.generate([
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "How do I read a file?"}
    ])  # Computes all 200 + N tokens

    # Second request: Only user message computed
    response2 = llm.generate([
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "How do I write to a file?"}
    ])  # Reuses 200 cached tokens, computes only new N tokens

**Scenario 2: Document QA**

.. code-block:: python

    long_document = "..."  # 2000 tokens

    # Multiple questions about the same document
    for question in questions:
        response = llm.generate([
            {"role": "system", "content": f"Answer based on: {long_document}"},
            {"role": "user", "content": question}
        ])  # Document prefix cached after first query

Throughput Impact
-----------------

By reducing computation per request:

* **Higher throughput** for workloads with shared prefixes
* **Better resource utilization** by serving more requests with the same compute
* **Reduced memory bandwidth** usage

Configuration
=============

Prefix caching is not **enabled by default** in Furiosa-LLM. To enable prefix caching, set ``--enable-prefix-caching`` option in the Furiosa-LLM server settings:

.. code-block:: bash

    furiosa-llm serve --enable-prefix-caching ...


Memory Management
-----------------

The prefix cache shares memory with the KV cache pool. The scheduler automatically:

* Allocates cache entries from available KV cache memory
* Evicts cached prefixes when memory pressure increases

No manual tuning of cache size or eviction policies is necessary.

Best Practices
==============

Maximizing Cache Hits
----------------------

To get the most benefit from prefix caching:

1. **Use consistent system prompts**: Keep system prompts identical across requests
2. **Maintain token-exact matching**: Even small changes (punctuation, whitespace) break the match
3. **Order messages consistently**: The cache matches from the beginning of the prompt sequence

Example - Consistent Prompting:

.. code-block:: python

    # Good: Exact same system prompt
    system_prompt = "You are a helpful AI assistant."

    response1 = llm.generate([{"role": "system", "content": system_prompt}, ...])
    response2 = llm.generate([{"role": "system", "content": system_prompt}, ...])
    # ✓ Cache hit

    # Bad: Slightly different prompts
    response3 = llm.generate([{"role": "system", "content": "You are a helpful AI assistant."}, ...])
    response4 = llm.generate([{"role": "system", "content": "You are a helpful assistant."}, ...])
    # ✗ Cache miss due to difference

Monitoring
==========

To monitor prefix caching effectiveness, observe:

* **Prefix cache hit ratio**: Higher ratios indicate better reuse of cached prefixes. You can find this in the server logs.
* **Time-to-First-Token (TTFT)**: Should decrease for requests with cached prefixes
* **Request latency patterns**: Requests following similar prompts should show consistent speedups
