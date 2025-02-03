.. _AsyncLLMEngine:

**********************************
AsyncLLMEngine class
**********************************


Overview
============

The AsyncLLMEngine provides an asynchronous interface for text generation, supporting configuration through command-line arguments.

Example Usage
=============

.. literalinclude:: ../../../../examples/async_llm_engine.py
   :language: python

The script can be executed with various arguments defined in AsyncEngineArgs, as shown in the following example:

.. code-block:: sh

    python async_llm_engine.py --model /path/to/model --devices npu:0:*

For a comprehensive list of available arguments for AsyncEngineArgs, please refer to the section below.

.. _AsyncLLMEngineArgs:

Arguments supported by AsyncLLMEngine
=====================================

The arguments are identical to those specified in :ref:`LLMEngineArgs`.

.. code-block:: text

    usage: async_llm_engine.py [-h] --model MODEL [--num-speculative-tokens NUM_SPECULATIVE_TOKENS] [--tokenizer TOKENIZER]
                               [--tokenizer-mode TOKENIZER_MODE] [--seed SEED] [--devices DEVICES]
                               [--data-parallel-size DATA_PARALLEL_SIZE] [--cache-dir CACHE_DIR] [--backend BACKEND]
                               [--paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS] [--npu-queue-limit NPU_QUEUE_LIMIT]
                               [--max-processing-samples MAX_PROCESSING_SAMPLES] [--spare-blocks-ratio SPARE_BLOCKS_RATIO]
                               [--is-offline IS_OFFLINE] [--use-speculative-decoding-if-possible USE_SPECULATIVE_DECODING_IF_POSSIBLE]
                               [--speculative-model-paged-attention-num-blocks SPECULATIVE_MODEL_PAGED_ATTENTION_NUM_BLOCKS]
                               [--packing-type PACKING_TYPE]
    
    options:
      -h, --help            show this help message and exit
      --model MODEL         Path to the LLM engine artifact (Pretrained id will be supported in the future releases).
      --num-speculative-tokens NUM_SPECULATIVE_TOKENS
                            The number of tokens that specualtive model will generate speculatively during each iteration of the decoding
                            process.
      --tokenizer TOKENIZER
                            The name or path of a HuggingFace Transformers tokenizer.
      --tokenizer-mode TOKENIZER_MODE
                            The tokenizer mode. "auto" will use the fast tokenizer if available, and "slow" will always use the slow
                            tokenizer.
      --seed SEED           The seed to initialize the random number generator for sampling.
      --devices DEVICES     The devices to run the model. It can be a single device or a list of devices. Each device can be either "npu:X"
                            or "npu:X:*" where X is a specific device index. If not given, available devices will be used.
      --data-parallel-size DATA_PARALLEL_SIZE
                            The size of the data parallelism group. If not given, it will be inferred from total avaialble PEs and other
                            parallelism degrees.
      --cache-dir CACHE_DIR
                            The cache directory for all generated files for this LLM instance. When its value is ``None``, caching is
                            disabled. The default is "$HOME/.cache/furiosa/llm".
      --backend BACKEND     The backend implementation to run forward() of a model for the LLM. If not specified, the backend will be
                            chosen based on the device kind.
      --paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS
                            The maximum number of blocks that each k/v storage per layer can store. This argument must be given if model
                            uses paged attention.
      --npu-queue-limit NPU_QUEUE_LIMIT
                            The NPU queue limit of the scheduler config.
      --max-processing-samples MAX_PROCESSING_SAMPLES
                            The maximum processing samples. Used as an hint for the scheduler.
      --spare-blocks-ratio SPARE_BLOCKS_RATIO
                            The spare blocks ratio. Used as an hint for the scheduler.
      --is-offline IS_OFFLINE
                            If True, the scheduler will assume the workload will be offline scenario.
      --use-speculative-decoding-if-possible USE_SPECULATIVE_DECODING_IF_POSSIBLE
                            If True, speculative decoding will be used if possible (`speculative_model` is given or there's artifacts for
                            specualtive model in the artifacts.). Otherwise, speculative decoding will not be used.
      --speculative-model-paged-attention-num-blocks SPECULATIVE_MODEL_PAGED_ATTENTION_NUM_BLOCKS
                            The maximum number of blocks that each k/v storage per layer can store for the specualtive model. This argument
                            must be given if the specualtive model uses paged attention.
      --packing-type PACKING_TYPE
                            Packing algorithm. Possible values are "IDENTITY" only for now


API Reference
====================

.. autoclass:: furiosa_llm.AsyncLLMEngine
    :members:
    :no-special-members:
