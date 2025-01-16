.. _ModelParallelism:

****************************************************
Model Parallelism
****************************************************

The model parallelism is a technique to split up the model into multiple parts
and run them in parallel across multiple devices.
It's useful to handle a large model that doesn't fit into a single device
or to increase the throughput or latency of the model inference.

Generally, there are three types of model parallelism: Tensor Parallelism, Pipeline Parallelism,
and Data Parallelism. Each parallelism type has its own benefits and drawbacks.
Also, you can combine multiple types of parallelism to achieve better performance.

In this section, we will explain the three types of model parallelism, how to configure them in Furiosa LLM,
and the best practices.


Tensor Parallelism (TP)
****************************
Tensor Parallelism (TP) splits each layer into multiple chunks along a specific dimension.
Each device only holds 1/N of the whole layer while not affecting the correctness of the computation graph.

TP can reduce the memory footprint of a single device for weights, KV cache, and activation memory
because only a part of the model and intermediate result reside on a single device.
As a result, TP is very useful when the model size is too large to fit into a single device.
Larger memory availability also allows higher batch sizes and longer sequences.

TP can reduce the latency of the model inference by leveraging
the aggregate computation power and memory bandwidth of multiple devices.

TP requires additional communication to synchronize the data across multiple devices.
The communication is called collective communication operation, and it synchronizes the data across
all multiple devices. If TP degree is too high, the communication can be overhead,
causing the performance degration of of the model inference.

Pipeline Parallelism (PP)
******************************
Pipeline Parallelism (PP) splits the model vertically (layer-level) across multiple NPUs,
so that only some layers of the model reside on a single NPU.
Each NPU processes a different part called 'stage' of the model, and the intermediate results are passed to the next NPU.

TP can reduce the memory footprint of a single device for weights, KV cache, and activation memory
because only a part of the model and intermediate result reside on a single device.
As a result, TP is very useful when the model size is too large to fit into a single device.
Larger memory availability also allows higher batch sizes and longer sequences.

Unlike TP, PP can increase the throughput of the model inference
rather than reducing latency because each NPU sequentially processes a different part of the model.
The end-to-end latency is determined by the accumulated latency of each stage.
Instead, PP can increase the throughput of the model inference by processing multiple requests in parallel.

Data Parallelism (DP)
******************************
Data parallelism (DP) is based on SPMD (single program multiple data) programming model.
It's the most common form of parallelism due to its simplicity.
For inference, DP replicates the same model instance multiple times,
and each replica computes independently a split of input requests.
So, DP is very useful to scale out the throughput of the model inference.

Each model replica requires a separate copy of the model weights and KV cache memory
across the multiple RNGD cards. So, the memory usage is proportional to the number of replicas.
If you use DP within the single RNGD card, the weights of replicas in the same RNGD card
can be shared due to the shared memory architecture.

How to Configure Model Parallelism in Furiosa LLM
=================================================================

ArtifactBuilder API or ``furiosa-llm build`` command provides the following options to configure the model parallelism:

- ``tensor_parallel_size``: the number of devices to split each layer with tensor parallelism
- ``pipeline_parallel_size``: the number of devices to split the model vertically
- ``data_parallel_size``: the number of model replicas

Each option specifies the parallelism degree for each parallelism type.
By default, the parallelism degrees will be automatically inferred based
on the total number of available NPUs in your machine.
So, you don't have to specify all the parallelism degrees.
You can refer to the best practices section to configure the parallelism degrees.

Best Practices in Furiosa LLM
=================================================================
Each parallelism type has its own natures and constraints.

The ``pipeline_parallel_size`` can be at most the number of RNGD cards; e.g.,

- With 2 RNGD cards, ``pipeline_parallel_size`` should be ``<=2``.
- With 8 RNGD cards, ``pipeline_parallel_size`` should be ``<=8``.

The ``tensor_parallel_size`` should be 4 or 8 in the 2024.2 release.
In the future release, it can be higher than 8.

The product of ``tensor_parallel_size``, ``pipeline_parallel_size``, and ``data_parallel_size``
should be equal to the total number of NPUs in your machine; i.e.,

.. code-block::

    TP size x PP size x DP size = total number of NPU PEs

For examples, let's assume that you have 4 RNGD cards, each of which has 8 PEs. Then, the total number of PEs is 32.
In that case, you can use the following configurations:

- ``tensor_parallel_size=8``, ``pipeline_parallel_size=4``, ``data_parallel_size=1``
- ``tensor_parallel_size=4``, ``pipeline_parallel_size=2``, ``data_parallel_size=2``









