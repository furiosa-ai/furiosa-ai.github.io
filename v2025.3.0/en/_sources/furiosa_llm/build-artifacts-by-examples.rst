.. _BuildingModelArtifactsByExamples:

****************************************************
Building Model Artifacts By Examples
****************************************************
This document provides examples of how to build model artifacts using Furiosa-LLM.
If you’re unfamiliar with the concept of model artifacts or the overall workflow for preparing a model for Furiosa-LLM,
please refer to the :ref:`ModelPreparation` section for an introduction.

Prerequisites
==================================================
Before starting this section, please ensure you have the following prerequisites:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa-LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`


Command-Line Tool vs. ArtifactBuilder API
=====================================================
There are two ways to build model artifacts using Furiosa-LLM:

* The :ref:`ArtifactBuilder` API
* The ``furiosa-llm build`` command-line tool

The :ref:`ArtifactBuilder` API is a Python interface that enables you to build model artifacts
with Furiosa-LLM, offering greater programmability and flexibility.

The ``furiosa-llm build`` command is a command-line tool that provides the same functionality
as the :ref:`ArtifactBuilder` API. It offers a simpler and more convenient way to build model artifacts
without writing any code. Its syntax is as follows:

.. code-block:: sh

    furiosa-llm build <MODEL_ID_OR_PATH> <OUTPUT_PATH> [OPTIONS]


You can see more options by running ``furiosa-llm build --help``.
Essentially, it is a wrapper around the ``ArtifactBuilder`` API,
so you can find details about the options in the reference for the :ref:`ArtifactBuilder` API.

Examples
=====================================================
The examples in this section are based on ``furiosa-llm build`` command and the Llama 3.1 8B model.
You can also implement the same examples using the :ref:`ArtifactBuilder` API.

Basic Examples
-----------------
Below is a basic example of building a model artifact using the ``furiosa-llm build`` command.
The first argument can be a Hugging Face model ID or a local path.
A local path should start with ``.`` or ``/`` and should point to a directory
containing a Hugging Face Transformers model or a quantized model by Furiosa-LLM.
The second argument is the output path where the model artifact will be saved.

The following command doesn't have any options, so the model artifact will be built with the default options.
You can find the default options by running ``furiosa-llm build --help``.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact


Specifying Context Length
-------------------------
The ``--max-seq-len-to-capture`` option specifies the maximum sequence length supported by the Furiosa-LLM engine.
You can adjust this value according to your model and use case, as it affects how the model is optimized.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --max-seq-len-to-capture 4096


Specifying Buckets
------------------
Furiosa-LLM utilizes buckets to optimize models for varying sequence lengths.
The ``--prefill-buckets BUCKET_SIZE`` and ``--decode-buckets BUCKET_SIZE``
options specify a comma-separated list of bucket sizes for prefill and decode phases, respectively.
``-pb`` and ``-db`` are the short options for ``--prefill-buckets`` and ``--decode-buckets``, respectively.
The prefill bucket size is used during the input prompt processing phase, while the decode bucket size is used during the text-generation phase.
Each bucket is defined as a tuple of batch size and sequence length, specified as ``BATCH_SIZE,SEQ_LEN``.
You can specify multiple bucket sizes for the prefill and decode phases as follows:

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        -pb 1,512 \
        -pb 1,1024 \
        -db 4,1024 \
        -db 4,2048

The above example specifies two prefill buckets: ``1,512`` and ``1,1024``, and two decode buckets: ``4,1024`` and ``4,2048``.
This means that the model can process up to 1024 tokens in input prompts in a single batch,
and generate up to 2048 tokens in 4 batches.

.. tip::

    We highly recommend using a batch size of 1 for prefill buckets. For decode buckets, use larger batch sizes to improve
    utilization—preferably powers of two, such as 2, 4, or 8. This is because the prefill phase typically saturates compute
    capacity even with small batches, whereas the decode phase suffers from low compute efficiency, as it generates
    only one token per request.


Using Parallelism for Large Models
----------------------------------
To serve large models efficiently, model parallelism is required to distribute the model across multiple NPUs.
Furiosa-LLM supports 3D parallelism, which includes tensor parallelism, pipeline parallelism, and data parallelism.

The ``--tensor-parallel-size PE_NUM`` option specifies the number of PEs (Processing Elements) to use for tensor parallelism.
Note that we use the number of PEs as the unit of tensor parallelism because a single RNGD contains 8 partitionable PEs.

The degree of tensor parallelism must be a power of two, starting from 1 (e.g., 1, 2, 4, 8, etc.).
We recommend using 1, 4, or 8 as the tensor parallelism size.
For example, a value of 1 or 4 is recommended for BERT models, while 4 or 8 is recommended for Llama models.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --tensor-parallel-size 8


The ``--pipeline-parallel-size DEVICE_NUM`` option specifies the number of devices across which the model should be partitioned for pipeline parallelism.
For example, if you have 4 devices and want to use all of them for pipeline parallelism,
you can specify ``--pipeline-parallel-size 4``.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --pipeline-parallel-size 4 \
        --tensor-parallel-size 8


.. note::

    Note that the ``--pipeline-parallel-size`` option can be overriden at model load time.
    Therefore, the settings specified at build time serve only as a preferred configuration.
    In contrast, the ``--tensor-parallel-size`` option is fixed at build time and cannot be changed at load time.


To learn more about model parallelism, please refer to the :ref:`ModelParallelism` section.


.. _TrustRemoteCode:

Enabling ``trust_remote_code``
------------------------------
``trust_remote_code`` is an option in Hugging Face Transformers that allows you to trust remote code
when loading a model from the Hugging Face Hub. You can use this option when building a model artifact
from remote code on the Hugging Face Hub as follows:

.. code-block::

    furiosa-llm build LGAI-EXAONE/EXAONE-3.5-7.8B-Instruct \
        /path_to/artifact \
        --auto-bfloat16-cast \
        --trust-remote-code

.. _AutoBfloat16Cast:

Building Float16, Float32 Models
--------------------------------
Furiosa-LLM builds models in bfloat16 format by default. However, if you want to build a model
in float16 or float32 format, you must explicitly request casting to bfloat16
using the ``--auto-bfloat16-cast`` flag; otherwise, an error will occur.
The example in :ref:`TrustRemoteCode` uses ``--auto-bfloat16-cast`` option because the EXAONE model uses float16 as dtype.

.. _ChunkedPrefill:

Long-context Length and Chunked Prefill
---------------------------------------
Chunked prefill is a technique that splits large prefills into small chunks.
This is an experimental feature and is still under development.
It does not yet support batching a single prefill with multiple decode requests.
However, it can still be useful for long-context length models.
To enable this feature, you need to specify the ``--prefill-chunk-size CHUNK_SIZE`` option as follows:

.. literalinclude:: ../../../examples/chunked_prefill.sh
  :language: python


Building Quantized Models
-------------------------
You can find an example of building a quantized model in the :ref:`ModelQuantization` section.

Building Model Artifacts in Parallel
-----------------------------------------
Building a large model artifact can take a long time.
To speed up the process, you can use multiple workers to build the model artifact.
The ``--num-pipeline-builder-workers [NUM]`` option specifies the number of
workers used to analyze and convert a model graph into multiple execution plans,
each of which serves as input to the Furiosa compiler.
This option is especially useful when building multiple buckets for a model.

.. note::

    The ``--num-pipeline-builder-workers NUM`` option requires CPU memory
    approximately equal to `NUM` times the total size of the model weights.
    Ensure that you have sufficient CPU memory available before using this option.

The ``--num-compile-workers NUM`` option specifies the number of workers to use for compiling a model artifact.
This option can speed up the compilation process by distributing the compilation tasks across multiple workers.

The following example shows how to build a model artifact using 4 pipeline builder workers and 4 compile workers.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --num-pipeline-builder-workers 4 \
        --num-compile-workers 4
