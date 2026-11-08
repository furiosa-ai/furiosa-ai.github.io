.. _LLMEngine:

**********************************
LLMEngine class
**********************************

Overview
========

The LLMEngine provides an interface for text generation, supporting configuration through command-line arguments.

Example Usage
=============

.. literalinclude:: ../../../../examples/engine_cli.py
   :language: python

The script can be executed with various arguments defined in EngineArgs, as shown in the following example:

.. code-block:: sh

    python engine_cli.py --model /path/to/model --devices npu:0

For a comprehensive list of available arguments for EngineArgs, please refer to the section below.

.. _LLMEngineArgs:

Arguments supported by LLMEngine
================================

.. help-text-start
.. code-block:: text

    usage: engine_cli.py [-h] --model MODEL [--fxb FXB] [--revision REVISION]
                         [--tokenizer TOKENIZER] [--tokenizer-mode TOKENIZER_MODE]
                         [--devices DEVICES]
                         [--pipeline-parallel-size PIPELINE_PARALLEL_SIZE]
                         [--data-parallel-size DATA_PARALLEL_SIZE]
                         [--npu-queue-limit NPU_QUEUE_LIMIT]
                         [--max-processing-samples MAX_PROCESSING_SAMPLES]
                         [--experimental-enable-npu-sampling]

    options:
      -h, --help            show this help message and exit
      --model MODEL         The Hugging Face model ID or local model directory. Currently
                            only one model is supported per server.
      --fxb FXB             Optional path to a compatible FXB. When omitted, a bundle is
                            discovered from the model directory or local FXB cache.
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
      --devices DEVICES     The devices to run the model. It can be a single device or a
                            comma-separated list of devices. Each device can be either
                            "npu:X" or "npu:X:Y", where X is a device index and Y is a NPU
                            core range notation (e.g. "npu:0" for whole npu 0, "npu:0:0"
                            for core 0 of NPU 0, and "npu:0:0-3" for fused core 0-3 of npu
                            0). If not given, all available unoccupied devices will be
                            used.
      --pipeline-parallel-size PIPELINE_PARALLEL_SIZE
                            The size of the pipeline parallelism group. If not given, it
                            will use the value from the loaded FXB.
      --data-parallel-size DATA_PARALLEL_SIZE
                            The size of the data parallelism group. If not given, it will
                            be inferred from total available PEs and other parallelism
                            degrees.
      --npu-queue-limit NPU_QUEUE_LIMIT
                            The NPU queue limit of the scheduler config.
      --max-processing-samples MAX_PROCESSING_SAMPLES
                            The maximum processing samples. Used as an hint for the
                            scheduler.
      --experimental-enable-npu-sampling
                            [EXPERIMENTAL] Draw the next token with the artifact's on-chip
                            sampling kernel instead of on the host. Requires an artifact
                            compiled with sampling kernels; engine init fails if the
                            artifact carries no sampling kernel. (default: False)
.. help-text-end

API Reference
===============

.. autoclass:: furiosa_llm.LLMEngine
    :members:
    :no-special-members:
