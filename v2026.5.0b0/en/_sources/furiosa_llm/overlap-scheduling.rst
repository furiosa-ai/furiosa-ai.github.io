.. _OverlapScheduling:

**********************************
Overlap Scheduling
**********************************

**Overlap scheduling** (also known as *asynchronous scheduling*) prepares the next batch on the CPU
while the NPU executes the current batch. This reduces the time the NPU spends waiting for scheduling
work and can improve throughput and time per output token (TPOT).

The tradeoff is additional time to first token (TTFT): a newly arriving request
cannot join a batch that has already been prepared. In most cases, the decreased TPOT
makes up for the slightly increased TTFT, resulting in overall improved performance across
all metrics aside from TTFT.

Overlap scheduling is disabled by default and can be enabled for supported text-generation models.

Usage
=============

CLI
---

Enable overlap scheduling when starting the server:

.. code-block:: bash

   furiosa-llm serve <model> --enable-overlap-scheduling

To return to eager scheduling, omit the option or disable it explicitly:

.. code-block:: bash

   furiosa-llm serve <model> --no-enable-overlap-scheduling


Python API
----------

For offline generation, select the scheduling loop through ``SchedulerConfig``:

.. code-block:: python

   from furiosa_llm import LLM
   from furiosa_llm.metadata.config_types import SchedulerConfig, SchedulingLoop

   with LLM(
       "furiosa-ai/Qwen3-8B-FP8",
       scheduler_config=SchedulerConfig(
           scheduling_loop_type=SchedulingLoop.OVERLAP,
       ),
   ) as llm:
       outputs = llm.generate([
           "Explain how a transformer generates text.",
           "Describe the difference between prefill and decode.",
       ])

Use ``SchedulingLoop.EAGER`` to select the default loop explicitly.

How It Works
============

The default **eager** scheduling loop prepares each batch after processing the
previous batch's results. This sequential dependency is natural, as the next batch
usually requires the sampled outputs from the previous batch's ongoing decode requests.

With **overlap** scheduling, Furiosa-LLM anticipates which active requests will need
another decode step or another prompt chunk, then selects requests, allocates KV cache blocks,
and prepares the next batch's I/O buffers and actions while the current batch runs on the NPU.

.. code-block:: text

   Eager scheduling:
   CPU: [prepare A]             [prepare B]             [prepare C]
   NPU:             [execute A]             [execute B]             [execute C]

   Overlap scheduling:
   CPU: [prepare A] [prepare B] [prepare C]
   NPU:             [execute A] [execute B] [execute C]

This diagram is conceptual; result processing and synchronization may still take
time in certain cases. Before dispatching the prepared decode batch, Furiosa-LLM fills in the
actual sampled tokens from the preceding batch. It anticipates request
continuation, rather than guessing token values.

However, overlap scheduling may incur additional overhead when a request generates an
end-of-sequence (EOS) token or is cancelled by the client.
An already-prepared step may still run, even though its output is discarded.

When to Use It
==============

Overlap scheduling is most useful when CPU batch preparation accounts for a
meaningful portion of the time between NPU executions. Workloads with sustained
generation and many decode iterations have repeated opportunities to hide this
overhead.

Potential benefits include:

* **Higher throughput**: less NPU idle time between batches can increase the
  number of output tokens served per second.
* **Lower TPOT**: reducing scheduling gaps can shorten the average interval
  between generated tokens.
* **Better NPU utilization**: CPU preparation and NPU execution can make
  progress at the same time.

The improvement depends on the model, batch sizes, prompt and output lengths,
request arrival pattern, and host performance. The first batch still needs to
be prepared before it can run. Short generations have fewer opportunities to
benefit, and workloads dominated by NPU computation may see a smaller relative
improvement. Overlap hides some CPU work; it does not eliminate that work or
make an individual model forward pass faster.

Overlap scheduling can be a useful optimization when its throughput and TPOT gains outweigh the increase in TTFT.

Caveats and Compatibility
=========================

First-Token Latency
-------------------

A prepared batch is dispatched before newly arriving requests are incorporated
into subsequent scheduling work. A new request can therefore wait for an
additional batch before its prompt is processed. This can increase TTFT even
when throughput and TPOT improve.

The added delay depends on batch execution time and load; it is not a fixed
number of milliseconds or an upper bound on total request latency. If your
application prioritizes fast first responses or has a strict TTFT target,
compare both loops under the expected request load.

Work After Completion or Cancellation
-------------------------------------

The scheduler may have already prepared and dispatched another step when it
learns that a request has finished or been cancelled. That step can consume NPU
compute and temporary KV cache space even though its output is discarded.
Furiosa-LLM releases the extra write blocks after the in-flight work completes.
This overhead can reduce the benefit for workloads with very short outputs or
frequent cancellations.

Supported Configurations
------------------------

.. list-table:: Current compatibility
   :header-rows: 1
   :widths: 35 65

   * - Feature or setting
     - Behavior with overlap scheduling
   * - Text-generation models
     - Supported, subject to the restrictions below.
   * - Vision-language models
     - Unsupported. Engine startup fails, including when the intended
       requests contain only text. Will be supported in the next major release.
   * - Pooling models (embedding and reranking)
     - Unsupported. Engine startup fails because these models have no decode phase.
   * - NPU prefix cache (L1)
     - Supported. Overlap scheduling does not require disabling
       :ref:`PrefixCaching`.
   * - Host KV cache (L2)
     - Supported when configured. See :ref:`HierarchicalPrefixCaching`.
   * - Layerwise L2-to-L1 reload
     - Supported with the separate experimental reload option and a host cache
       budget. See :ref:`LayerwiseKVReload`.
   * - Experimental NPU sampling
     - Supported for compatible models and FXBs. See :ref:`NPUSampling`.
   * - Experimental speculative decoding
     - Unsupported. Use eager scheduling.
   * - Remote KV cache (L3, Mooncake)
     - Disabled when the overlap scheduler starts, even if configured.
       Use eager scheduling if your workload relies on remote cache reuse.
   * - ``npu_queue_limit`` / ``--npu-queue-limit``
     - Must remain ``1``, the default. Overlap scheduling prepares CPU work
       ahead of time without requiring a larger NPU execution queue.

For unsupported model or decoding configurations, restart with
``--no-enable-overlap-scheduling`` or ``SchedulingLoop.EAGER``.

Evaluating Performance
======================

Compare eager and overlap scheduling using the same model artifact, device
configuration, concurrency, prompt and output lengths, and sampling settings.
Warm up each configuration and keep prefix-cache conditions comparable.

Measure both throughput and latency:

* **Output tokens per second** to evaluate serving capacity.
* **TPOT or inter-token latency** to evaluate streaming generation speed.
* **TTFT, including tail percentiles**, to check the cost to new requests.
* **End-to-end request latency** to assess the complete user experience.

Include the mix of short and long requests and the arrival rates expected in
production. Prefer overlap scheduling when the throughput or generation-speed
improvement is useful and first-token latency remains within your application's
target. The server logs ``Overlap scheduler has started`` when this loop starts;
use that message to confirm the setting took effect.
