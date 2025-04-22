.. _LLMEngine:

**********************************
LLMEngine class
**********************************

Overview
========

The LLMEngine provides an interface for text generation, supporting configuration through command-line arguments.

Example Usage
=============

.. literalinclude:: ../../../../examples/llm_engine.py
   :language: python

The script can be executed with various arguments defined in EngineArgs, as shown in the following example:

.. code-block:: sh

    python llm_engine.py --model /path/to/model --devices npu:0

For a comprehensive list of available arguments for EngineArgs, please refer to the section below.

.. _LLMEngineArgs:

Arguments supported by LLMEngine
================================

.. help-text-start
.. code-block:: text

    usage: llm_engine.py [-h] --model MODEL [--revision REVISION] [--tokenizer TOKENIZER]
                         [--tokenizer-mode TOKENIZER_MODE] [--seed SEED]
                         [--devices DEVICES]
                         [--pipeline-parallel-size PIPELINE_PARALLEL_SIZE]
                         [--data-parallel-size DATA_PARALLEL_SIZE]
                         [--cache-dir CACHE_DIR]
                         [--paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS]
                         [--npu-queue-limit NPU_QUEUE_LIMIT]
                         [--max-processing-samples MAX_PROCESSING_SAMPLES]
                         [--spare-blocks-ratio SPARE_BLOCKS_RATIO]
    
    options:
      -h, --help            show this help message and exit
      --model MODEL         The Hugging Face model id, or path to Furiosa model artifact.
                            Currently only one model is supported per server.
      --revision REVISION   The specific model revision on Hugging Face Hub if the model
                            is given as a Hugging Face model id. It can be a branch name,
                            a tag name, or a commit id. Its default value is main.
                            However, if a given model belongs to the furiosa-ai
                            organization, the model will use the release model tag by
                            default.
      --tokenizer TOKENIZER
                            The name or path of a HuggingFace Transformers tokenizer.
      --tokenizer-mode TOKENIZER_MODE
                            The tokenizer mode. "auto" will use the fast tokenizer if
                            available, and "slow" will always use the slow tokenizer.
      --seed SEED           The seed to initialize the random number generator for
                            sampling.
      --devices DEVICES     The devices to run the model. It can be a single device or a
                            comma-separated list of devices. Each device can be either
                            "npu:X" or "npu:X:Y", where X is a device index and Y is a
                            NPU core range notation (e.g. "npu:0" for whole npu 0,
                            "npu:0:0" for core 0 of NPU 0, and "npu:0:0-3" for fused core
                            0-3 of npu 0). If not given, all available unoccupied devices
                            will be used.
      --pipeline-parallel-size PIPELINE_PARALLEL_SIZE
                            The size of the pipeline parallelism group. If not given, it
                            will use the default pp value of the artifact.
      --data-parallel-size DATA_PARALLEL_SIZE
                            The size of the data parallelism group. If not given, it will
                            be inferred from total available PEs and other parallelism
                            degrees.
      --cache-dir CACHE_DIR
                            The cache directory for temporarily generated files for this
                            LLM instance. When its value is ``None``, caching is
                            disabled. The default is "$HOME/.cache/furiosa/llm".
      --paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS
                            The maximum number of blocks that each k/v storage per layer
                            can store. This argument must be given if model uses paged
                            attention.
      --npu-queue-limit NPU_QUEUE_LIMIT
                            The NPU queue limit of the scheduler config.
      --max-processing-samples MAX_PROCESSING_SAMPLES
                            The maximum processing samples. Used as an hint for the
                            scheduler.
      --spare-blocks-ratio SPARE_BLOCKS_RATIO
                            The spare blocks ratio. Used as an hint for the scheduler.
.. help-text-end

API Reference
===============

.. autoclass:: furiosa_llm.LLMEngine
    :members:
    :no-special-members:
