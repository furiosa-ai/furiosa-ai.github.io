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

Token-Level Matching
--------------------

Furiosa-LLM's prefix caching operates at the token level, using a radix tree data structure for efficient prefix matching:

.. code-block:: text

    Request 1: [System prompt] + [User message A]
               └─ KV cache stored ─┘

    Request 2: [System prompt] + [User message B]
               └─ Reused cache! ─┘  └─ New computation ─┘

Automatic Management
--------------------

The prefix cache is managed automatically by the scheduler:

* **Cache Population**: KV cache entries are stored after each forward pass
* **Cache Matching**: New requests are checked against the cache tree
* **Cache Eviction**: Least recently used entries are evicted when memory is needed
* **Cache Invalidation**: No manual invalidation required

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

Prefix caching is **enabled by default** in Furiosa-LLM with no configuration required.

Memory Management
-----------------

The prefix cache shares memory with the KV cache pool. The scheduler automatically:

* Allocates cache entries from available KV cache memory
* Evicts cached prefixes when memory pressure increases
* Balances cache retention with active request memory needs

No manual tuning of cache size or eviction policies is necessary.

Best Practices
==============

Maximizing Cache Hits
----------------------

To get the most benefit from prefix caching:

1. **Use consistent system prompts**: Keep system prompts identical across requests
2. **Maintain token-exact matching**: Even small changes (punctuation, whitespace) break the match
3. **Order messages consistently**: The cache matches from the beginning of the prompt sequence
4. **Batch similar requests**: Process requests with shared prefixes together when possible

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

What Doesn't Work
-----------------

Prefix caching **will not match** when:

* Tokenization differs (e.g., different tokenizer configurations)
* Token sequences differ at any position before the divergence point
* Prompts are presented in different formats (chat vs. completion)

Monitoring
==========

Cache Effectiveness
-------------------

To monitor prefix caching effectiveness, observe:

* **Time-to-First-Token (TTFT)**: Should decrease for requests with cached prefixes
* **KV Cache utilization**: Prefix cache uses a portion of total KV cache memory
* **Request latency patterns**: Requests following similar prompts should show consistent speedups

The Furiosa-LLM server logs include prefix cache statistics when detailed logging is enabled.

Limitations
===========

* **Memory overhead**: Cached prefixes consume KV cache memory
* **Exact matching only**: Small prompt variations prevent cache hits
* **Token-level granularity**: Partial token matches are not supported
* **Single-model scope**: Cache is not shared across different model instances

Advanced Topics
===============

Prefix Cache Eviction
----------------------

The cache uses a **Least Recently Used (LRU)** eviction strategy:

* Recently used prefixes are retained longer
* Infrequently accessed prefixes are evicted first when memory is needed
* The eviction happens automatically during request scheduling

Cache Coherency
---------------

Prefix caching is coherent across concurrent requests:

* Multiple requests can share the same cached prefix simultaneously
* Cache entries remain valid until explicitly evicted
* No synchronization overhead for read-only cache access

For More Information
====================

* See :ref:`OpenAI Server API <OpenAIServer>` for server usage
* See :ref:`LLM class reference <LLMClass>` for Python API details
* See :ref:`Supported Models <SupportedModels>` for compatible models
