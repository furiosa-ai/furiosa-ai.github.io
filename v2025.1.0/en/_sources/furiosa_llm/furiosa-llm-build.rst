.. _BuildingModelArtifact:

****************************************************
Building a Model Artifact
****************************************************
A compiled model artifact is required to run the LLM Engine using the :ref:`LLMClass` or ``furiosa-llm serve``.

.. note::

    If you are not familiar with what model artifact is,
    please refer to :ref:`ModelPreparationWorkflow`.

You can compile the model with the LLM ArtifactBuilder API or via the ``furiosa-llm build`` command.

.. warning::

   This document is based on Furiosa SDK |release| (beta0) version,
   and the features and APIs described in this document may change in the future.

Prerequisites
==================================================
To use ArtifactBuilder API or ``furiosa-llm build`` command,
you need the following prerequisites:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`


.. _ArtifactBuilder:

ArtifactBuilder API
================================================

.. code-block:: python

    from furiosa_llm.artifact.builder import ArtifactBuilder
    from furiosa_llm.models.config_types import SchedulerConfig


    builder = ArtifactBuilder(
        "meta-llama/Meta-Llama-3.1-8B-Instruct",
        "npu:0:*",
        "mlperf-llama3-1-8b-fp8",
        tensor_parallel_size=4,
        pipeline_parallel_size=1,
        prefill_buckets=[
            (1, 512),
            (1, 1024),
        ],
        decode_buckets=[
            (1, 2048),
            (8, 2048),
            (16, 2048),
            (64, 2048),
            (128, 2048),
        ],
        quantize_artifact_path="/path_to/quantized_artifacts",
        calculate_logit_only_for_last_token=True,
        paged_attention_num_blocks=512_000,
        default_scheduler_config=SchedulerConfig(
            max_processing_samples=24576,
            spare_blocks_ratio=0.6,
            npu_queue_limit=2,
        ),
    )

    builder.build(
        "/path_to/artifact",
        num_pipeline_builder_workers=4,
        num_compile_workers=4,
    )

There are more options available for the ArtifactBuilder API.
You can find the full list of options and arguments in the :ref:`ArtifactBuilderClass` reference.

.. _FuriosaLLMBuildCommand:

``furiosa-llm build`` command
================================================
(WIP)
The following is the list of options and arguments for the serve command:

.. code-block::

    usage: furiosa-llm build [-h] --model-id MODEL_ID [--name NAME] [--devices DEVICES] [-tp TENSOR_PARALLEL_SIZE] [-pp PIPELINE_PARALLEL_SIZE] [-dp DATA_PARALLEL_SIZE] [-pb PREFILL_BUCKETS] [-db DECODE_BUCKETS]
                            [--max-seq-len-to-capture MAX_SEQ_LEN_TO_CAPTURE] [--additional-model-config ADDITIONAL_MODEL_CONFIG] [--quantization-artifact-path QUANTIZATION_ARTIFACT_PATH]
                            [--kv-cache-sharing-across-beams-config KV_CACHE_SHARING_ACROSS_BEAMS_CONFIG] [--paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS] [--num-pipeline-builder-workers NUM_PIPELINE_BUILDER_WORKERS]
                            [--num-compile-workers NUM_COMPILE_WORKERS]
                            output_path

    positional arguments:
    output_path           The path to export the artifacts.

    options:
    -h, --help            show this help message and exit
    --model-id MODEL_ID   The Hugging Face pretrained id (e.g., "meta-llama/Meta-Llama-3.1-8B-Instruct").
    --name NAME           The name of the artifact to build.
    --devices DEVICES     Devices to use (e.g., "npu:0:*,npu:1:*"). If not specified, the artifact will be built using only one device.
    -tp TENSOR_PARALLEL_SIZE, --tensor-parallel-size TENSOR_PARALLEL_SIZE
                            The number of PEs for each tensor parallelism group. (default: 4)
    -pp PIPELINE_PARALLEL_SIZE, --pipeline-parallel-size PIPELINE_PARALLEL_SIZE
                            The number of stages for pipeline parallelism. (default: 1)
    -dp DATA_PARALLEL_SIZE, --data-parallel-size DATA_PARALLEL_SIZE
                            The size of the data parallelism group. If not specified, it will be inferred based on the total available PEs and other parallelism configurations.
    -pb PREFILL_BUCKETS, --prefill-buckets PREFILL_BUCKETS
                            Specify the bucket size for prefill in the format batch_size,context_length. Multiple entries are allowed (e.g., `--pb 1,128 --pb 1,256`).
    -db DECODE_BUCKETS, --decode-buckets DECODE_BUCKETS
                            Specify the bucket size for decode in the format batch_size,context_length. Multiple entries are allowed (e.g., `--db 4,2048 --db 16,2048`).
    --max-seq-len-to-capture MAX_SEQ_LEN_TO_CAPTURE
                            The maximum sequence length supported by the LLM engine. Sequences exceeding this length will not be handled.
    --additional-model-config ADDITIONAL_MODEL_CONFIG
                            Specify compilation settings or optimization settings to apply to your model. You can specify multiple items in the form `key=value`.
    --quantization-artifact-path QUANTIZATION_ARTIFACT_PATH
                            The path where quantization artifacts generated by the Furiosa Model Compressor are saved.
    --kv-cache-sharing-across-beams-config KV_CACHE_SHARING_ACROSS_BEAMS_CONFIG
                            Configuration for sharing k/v caches across beams. Required if the model supports kv cache sharing. Format: beam_width,max_new_token (e.g., `4,128`).
    --paged-attention-num-blocks PAGED_ATTENTION_NUM_BLOCKS
                            The maximum number of blocks each k/v storage layer can store. Required if the model uses paged attention.
    --num-pipeline-builder-workers NUM_PIPELINE_BUILDER_WORKERS
                            The number of workers for building pipelines (excluding compilation). Defaults to 1 (no parallelism). Higher values reduce build time for large models but require more memory.
    --num-compile-workers NUM_COMPILE_WORKERS
                            The number of workers used for compilation.


.. code-block:: sh

    furiosa-llm build /path/to/artifacts \
        --model-id meta-llama/Meta-Llama-3.1-8B-Instruct \
        --devices "npu:0:*" \
        --name mlperf-llama3-1-8b-fp8 \
        -tp 4 -pp 1 \
        -pb 1,512 -pb 1,1024 \
        -db 1,2048 -db 8,2048 -db 16,2048 -db 64,2048 -db 128,2048 \
        --quantization-artifact-path /path/to/quantized_artifacts \
        --paged-attention-num-blocks 512000 \
        --additional-model-config calculate_logit_only_for_last_token=True \
        --num-pipeline-builder-workers 4 \
        --num-compile-workers 4
