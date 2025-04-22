.. _BuildingModelArtifactsByExamples:

****************************************************
Building Model Artifacts By Examples
****************************************************
This document provides examples of how to build model artifacts using the Furiosa-LLM.
If you’re unfamiliar with the concept of model artifacts or the overall workflow for preparing a model for Furiosa-LLM,
please refer to the :ref:`ModelPreparation` section for an introduction.

Prerequisites
==================================================
Before you start this section, please ensure you have the following prerequisites:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa-LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`


Command Line Tool vs. ArtifactBuilder API
=====================================================
There are two ways to build model artifacts using Furiosa-LLM:

* :ref:`ArtifactBuilder` API
* ``furiosa-llm build`` command line tool

The :ref:`ArtifactBuilder` API is a Python interface that enables you to build model artifacts
with Furiosa-LLM, offering greater programmability and flexibility.

The ``furiosa-llm build`` command is a command line tool that provides the same functionality
as the :ref:`ArtifactBuilder` API. It provides a simpler and more convenient way to build model artifacts
without writing any code. Its syntax is as follows:

.. code-block:: sh

    furiosa-llm build <MODEL_ID_OR_PATH> <OUTPUT_PATH> [OPTIONS]


You can see more options by running ``furiosa-llm build --help``.
Basically, it is a wrapper around the ``ArtifactBuilder`` API,
So, you can find the details about options from the reference of the :ref:`ArtifactBuilder` API.

Examples
=====================================================
The examples in this section are based ``furiosa-llm build`` command and Llama 3.1 8B model.
You can also implement the same examples using the :ref:`ArtifactBuilder` API.

Basic Examples
-----------------
Below is a basic example of building a model artifact using the ``furiosa-llm build`` command.
The first argument can be a Hugging Face model id or a local path.
A local path should start with ``.`` or ``/``, and should point to a directory
that contains a Hugging Face Transformers model or a quantized model by Furiosa-LLM.
The second argument is the output path where the model artifact will be saved.

The following command doesn't have any options. The model artifact will be built with the default options.
You can find the default options from ``furiosa-llm build --help``.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \


Specifying Context Length
-------------------------
``--max-seq-len-to-capture`` option specifies the maximum sequence length that the Furiosa-LLM engine supports.
You can adjust this value according to your model and use case. This value affects how the model is optimized.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --max-seq-len-to-capture 4096


Specifying Buckets
------------------
Furiosa-LLM utilizes buckets to optimize models for varying sequence lengths.
The ``--prefill-buckets BUCKET_SIZE`` option and ``--decode-buckets BUCKET_SIZE``
options specify a comma separated list of bucket sizes for prefill and decode phases respectively.
``-pb`` and ``-db`` are the short options for ``--prefill-buckets`` and ``--decode-buckets`` respectively.
The prefill bucket size is used for input prompt processing phase, and decode bucket size is used for text-generation phase.
Each bucket is defined as a tuple of a batch size and a sequence length, and it is specified as ``BATCH_SIZE, SEQ_LEN``.
You can specify multiple bucket sizes for prefill and decode phases as follows:

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        -pb 1,512 \
        -pb 1,1024 \
        -db 4,1024 \
        -db 4,2048

The above example specifies two prefill buckets: ``1,512`` and ``1,1024``, and two decode buckets: ``4,1024`` and ``4,2048``.
It means that the model can process up to 1024 tokens input prompts in a single batch,
and generate up to 2048 tokens in 4 batch.

.. tip::

    We highly recommend using a batch size of 1 for prefill buckets. For decode buckets, use larger batch sizes to improve
    utilization—preferably powers of two such as 2, 4, or 8. This is because the prefill phase usually saturates compute
    capacity even with small batches, whereas the decode phase suffers from low compute efficiency as it generates
    only one token per request.


Using Parallelism for Large Models
----------------------------------
To serve large models efficiently, model parallelism is required to distribute the model across multiple NPUs.
Furiosa-LLM supports 3D parallelism, which includes tensor parallelism, pipeline parallelism, and data parallelism.

The ``--tensor-parallel-size PE_NUM`` option specifies the number of PEs (Processing Elements) to be used for tensor parallelism.
Note that we use the number of PEs as the unit of tensor parallelism because a single RNGD has 8 partitionable PEs.

The degree of tensor parallelism must be a power of two, starting from 1; e.g., 1, 2, 4, 8, etc.
We recommend using 1, 4, or 8 as the tensor parallelism size.
For example, for BERT models, 1 or 4 is recommended; for Llama models, 4 or 8 is recommended.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --tensor-parallel-size 8


The ``--pipeline-parallel-size DEVICE_NUM`` option accepts the number of devices as its value,
indicating how many devices the model should be partitioned across for pipeline parallelism.
For example, if you have 4 devices and want to use all of them for pipeline parallelism,
you can specify ``--pipeline-parallel-size 4``.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --pipeline-parallel-size 4
        --tensor-parallel-size 8


.. note::

    Note that ``--pipeline-parallel-size`` option can be overrided at model load time.
    So, the settings specified at the build phase will work only as a preferred configuration.
    In contrast, the ``--tensor-parallel-size`` option is fixed at the build phase and cannot be changed at load time.


To learn more about model parallelism, please refer to the :ref:`ModelParallelism` section.


Enabling ``trust_remote_code``
------------------------------
``trust_remote_code`` is an option of Hugging Face Transformers that allows you to trust the remote code
when loading a model from the Hugging Face Hub. You can use this option when you are building a model artifact
from a remote code of Hugging Face hub as follows:

.. code-block::

    LGAI-EXAONE/EXAONE-3.5-7.8B-Instruct \
        /path_to/artifact \
        --auto-bfloat16-cast \
        --trust-remote-code

.. _AutoBfloat16Cast:

Building Float16, Float32 Models
--------------------------------
Furiosa-LLM builds models in bfloat16 format by default. However, if you want to build a model
in float16 or float32 format, you must explicitly request casting to bfloat16
by using the ``--auto-bfloat16-cast`` flag; otherwise, an error will occur.

The following example shows how to build the ``SOLAR-10.7B-Instruct-v1.0`` model in float16 format.

.. code-block::

    furiosa-llm build upstage/SOLAR-10.7B-Instruct-v1.0 \
        /path_to/artifact \
        --auto-bfloat16-cast


.. _ChunkedPrefill:

Long-context Length and Chunked Prefill
---------------------------------------
Chunked prefill is a technique that allows splitting large prefills into small chunks.
This is still an experimental feature and is still under development.
It does not yet batch a single prefill and multiple decode requests.
However, it's still useful for long-context length models.
To enable this feature, you need to specify the ``--prefill-chunk-size CHUNK_SIZE`` option as follows:

.. literalinclude:: ../../../examples/chunked_prefill.sh
  :language: python


Building Quantized Models
-------------------------
You can find an example of building a quantized model from :ref:`ModelQuantization` section.

Building Model Artifacts in Parallel
-----------------------------------------
Building a large model artifact can take a long time.
To speed up the process, you can use multiple workers to build the model artifact.
``--num-pipeline-builder-workers [NUM]`` option specifies the number of
workers to analyze and convert a model graph into multiple execution plans,
each of which is an input of Furiosa Compiler.
This option will be useful when you have to build multiple buckets for a model.

.. note::

    The ``--num-pipeline-builder-workers NUM`` option requires CPU memory
    that is approximately `NUM` times of the total size of the model weights.
    Ensure that you have enough CPU memory available before using this option.

``--num-compile-workers NUM`` option specifies the number of workers to be used for compiling the model artifact.
This option will speed up the compilation process by distributing the compilation tasks across multiple workers.

The following example shows how to build a model artifact using 4 pipeline builder workers and 4 compile workers.

.. code-block::

    furiosa-llm build meta-llama/Llama-3.1-8B-Instruct \
        /path_to/artifact \
        --num-pipeline-builder-workers 4 \
        --num-compile-workers 4
