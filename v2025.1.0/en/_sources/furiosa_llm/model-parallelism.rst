.. _ModelParallelism:

****************************************************
Model Parallelism
****************************************************

Model parallelism is a technique used to split a model into multiple parts
and run them in parallel across multiple devices.
It's particularly useful for handling large models that don't fit in a single
device or for improving the throughput or latency of model inference.

There are three types of model parallelism: tensor parallelism (TP),
pipeline parallelism (PP), and data parallelism (DP).
Each type has its own benefits and drawbacks.
Additionally, it is possible to combine multiple types of parallelism to achieve
better performance.

In this section, we explain the three types of model parallelism, how to
configure them in Furiosa LLM, and provide best practices.


Tensor Parallelism (TP)
***********************
Tensor Parallelism (TP) splits each layer into multiple chunks along a specific dimension.
Each device only holds 1/N of the whole layer, without affecting the correctness
of the computation graph.

Since only a portion of the model and intermediate results reside on each
device, TP reduces the memory requirements for weights, KV cache, and activation
memory in each device.
Hence, TP is particularly useful when the model size exceeds the memory capacity
of a single device.
Increased memory availability also allows larger batch sizes and longer
sequences.

Additionally, TP can reduce the latency of model inference by leveraging
the aggregate computation power and memory bandwidth of multiple devices.

However, TP requires extra communication to synchronize data across multiple
devices.
This operation, known as collective communication (e.g., all-reduce and
all-gather), ensures data consistency across all devices.
If the degree of TP is too high, the communication overhead can hurt
performance, leading to slower model inference.


Pipeline Parallelism (PP)
*************************
Pipeline Parallelism (PP) splits the model vertically (usually at layer level)
across multiple devices, so that only a subset of the model layers reside on a
single device.
Each device processes a different part, or "stage", of the model, passing the
intermediate results to the next device.

Like tensor parallelism, PP also reduces the memory footprint of a single
device.
This is because only a portion of the model and its intermediate results are
stored on each device.

Unlike TP, PP increases the throughput of model inference at the cost of
increased latency
Since each device processes a different part of the model sequentially, the
end-to-end latency is determined by the accumulated latency of each stage.
However, PP increases the throughput by processing multiple requests in
parallel.


Data Parallelism (DP)
*********************
Data parallelism (DP) is based on the SPMD (single program multiple data)
programming model.
It is the most common form of parallelism due to its simplicity.
For inference, DP replicates the same model instance multiple times,
with each replica independently processing a fraction of the input requests.
As a result, DP is highly effective for scaling out the throughput of model
inference.

Each device requires a separate copy of the model weights and KV cache.
Therefore, memory usage scales with the number of replicas.
However, when using DP within a single FuriosaAI device, the weights of replicas
can be shared due to the shared memory architecture of the device.


Configuring Model Parallelism in Furiosa LLM
============================================

The ArtifactBuilder API and the ``furiosa-llm build`` command allows to configure the degree of tensor parallelism
by specifying the ``--tensor_parallel_size`` or ``-tp`` option.

``pipeline_parallel_size`` and ``data_parallel_size`` can be configured when loading a model artifact.
For example, when you run ``furiosa-llm serve``, ``furiosa-mlperf`` command,
you can specify ``--pipeline-parallel-size`` (or ``-pp``) and ``--data-parallel-size`` (or ``-dp``) options.
Similarily, ``LLMEngine``, ``AsyncLLMEngine`` allows to specify the degree of pipeline parallelism and data parallelism.

Best Practices for Furiosa LLM
==============================
Each parallelism type has its own characteristics and constraints.

The ``pipeline_parallel_size`` parameter can be at most the number of devices.
The ``tensor_parallel_size`` parameter can only be 4 or 8 in the |release| release;
future releases will lift this limitation.

The product of ``tensor_parallel_size``, ``pipeline_parallel_size``, and ``data_parallel_size``
should be equal to the total number of PEs in your machine.

For example, let's assume that you have 4 RNGD cards, each of which has 8 PEs.
Then, the total number of PEs is 32.
In that case, you can use the following configurations:

- ``tensor_parallel_size=8``, ``pipeline_parallel_size=4``, ``data_parallel_size=1``
- ``tensor_parallel_size=4``, ``pipeline_parallel_size=2``, ``data_parallel_size=2``
