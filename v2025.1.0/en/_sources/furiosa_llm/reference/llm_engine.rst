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

.. code-block:: text

    usage: llm_engine.py [-h] --model MODEL [--tokenizer TOKENIZER] [--tokenizer-mode TOKENIZER_MODE] [--seed SEED] [--devices DEVICES] [--pipeline-parallel-size PIPELINE_PARALLEL_SIZE]
                         [--data-parallel-size DATA_PARALLEL_SIZE] [--cache-dir CACHE_DIR] [--paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS] [--npu-queue-limit NPU_QUEUE_LIMIT]
                         [--max-processing-samples MAX_PROCESSING_SAMPLES] [--spare-blocks-ratio SPARE_BLOCKS_RATIO] [--is-offline IS_OFFLINE]
    
    Demo on using the LLMEngine class directly
    
    options:
      -h, --help            show this help message and exit
      --model MODEL         Path to the LLM engine artifact (Pretrained id will be supported in the future releases).
      --tokenizer TOKENIZER
                            The name or path of a HuggingFace Transformers tokenizer.
      --tokenizer-mode TOKENIZER_MODE
                            The tokenizer mode. "auto" will use the fast tokenizer if available, and "slow" will always use the slow tokenizer.
      --seed SEED           The seed to initialize the random number generator for sampling.
      --devices DEVICES     The devices to run the model. It can be a single device or a list of devices. Each device can be either "npu:X" or "npu:X:*" where X is a specific device index.
                            If not given, available devices will be used.
      --pipeline-parallel-size PIPELINE_PARALLEL_SIZE
                            The size of the pipeline parallelism group. If not given, it will use the default pp value of the artifact.
      --data-parallel-size DATA_PARALLEL_SIZE
                            The size of the data parallelism group. If not given, it will be inferred from total avaialble PEs and other parallelism degrees.
      --cache-dir CACHE_DIR
                            The cache directory for temporarily generated files for this LLM instance. When its value is ``None``, caching is disabled. The default is
                            "$HOME/.cache/furiosa/llm".
      --paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS
                            The maximum number of blocks that each k/v storage per layer can store. This argument must be given if model uses paged attention.
      --npu-queue-limit NPU_QUEUE_LIMIT
                            The NPU queue limit of the scheduler config.
      --max-processing-samples MAX_PROCESSING_SAMPLES
                            The maximum processing samples. Used as an hint for the scheduler.
      --spare-blocks-ratio SPARE_BLOCKS_RATIO
                            The spare blocks ratio. Used as an hint for the scheduler.
      --is-offline IS_OFFLINE
                            If True, the scheduler will assume the workload will be offline scenario.

API Reference
===============

.. autoclass:: furiosa_llm.LLMEngine
    :members:
    :no-special-members:
