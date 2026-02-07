.. _ModelPreparation:

****************************************************
Model Preparation
****************************************************
To run a trained model, fine-tuned model or quantized model by yourself,
Furiosa-LLM needs to convert the model into a model artifact.
Also, if you want to apply specific optimization configurations or parallelization
strategies for your workload, you must perform this model preparation step.

.. tip::

  This section is intended for users who wish to prepare their own model artifacts
  for further optimization or customization. If you are looking for a quick start,
  please refer to the :ref:`GettingStartedFuriosaLLM` section.
  Additionally, Furiosa-LLM provides a set of pre-compiled model artifacts for popular LLMs in the
  `Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_.
  You can use these to quickly run LLM models on the Furiosa NPU.


Prerequisites
=============
Ensure that you meet the following prerequisites before starting the model preparation workflow:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa-LLM <InstallingFuriosaLLM>`
* A :ref:`Hugging Face access token <AuthorizingHuggingFaceHub>`
* Sufficient storage space for model weights (varies depending on the model size)

Authorizing Hugging Face Hub (Optional)
=================================================
Some models, such as ``meta-llama/Llama-3.1-8B``, require a license to run.
For these models, you need to create a Hugging Face account, accept the model's
license, and generate an access token.
You can create your access token at https://huggingface.co/settings/tokens.
Once you get the access token, you can authenticate on the Hugging Face Hub as follows:

.. code-block::

  pip install --upgrade "huggingface_hub[cli]"
  huggingface-cli login --token $HF_TOKEN

Download a model from Hugging Face Hub (Optional)
=================================================
When using a model from the Hugging Face Hub, Furiosa-LLM automatically downloads the model weights
during the artifact building process.
However, depending on your network environment, this download may take a significant amount of time.
If you prefer to download the model in advance, you can do so using the
``huggingface-cli`` command.

The following command downloads the model weights and configuration files
for the Llama 3.1 8B model to the Hugging Face Hub cache directory.

.. code-block::

  huggingface-cli download "meta-llama/Llama-3.1-8B-Instruct"


.. _ModelQuantization:

Model Quantization (Optional)
====================================
Quantization is a widely used technique to reduce the computational and memory requirements for inference
by mapping the high-precision space of activations, weights, and KV cache to lower-precision formats
such as INT8, FP8, or INT4 — while aiming to preserve model accuracy.

It is typically applied when higher throughput or lower latency is needed. However, since quantization may affect
model accuracy, it is important to perform thorough experimentation and accuracy evaluations.

Furiosa-LLM currently supports fine-grained FP8 dynamic quantization for causal language models.
We plan to support more quantization schemes based on llm-compressor in the next releases.

To quantize a model, you first need to load it using the
``AutoModelForCausalLM.from_pretrained()`` method from the ``transformers`` library.
When loading the model, you can specify the ``quantization_config`` parameter to define
the quantization scheme to be applied. Below is an example of loading and quantizing the Qwen3-32B model
using fine-grained FP8 dynamic quantization.

.. code-block:: python

    import torch
    from transformers import AutoModelForCausalLM, AutoTokenizer, FineGrainedFP8Config

    model_id = "Qwen/Qwen3-32B"
    save_path = "./qwen3-32b-fp8-dynamic"

    # FineGrained FP8 Quantization Example
    # Using Block-wise(128x128) Dynamic FP8 Quantization for weights
    quantization_config = FineGrainedFP8Config(
        activation_scheme="dynamic",
        weight_block_size=(128, 128)
    )

    # Loading the model with quantization configuration
    model = AutoModelForCausalLM.from_pretrained(
        model_id,
        device_map="auto",
        quantization_config=quantization_config,
        torch_dtype=torch.bfloat16
    )
    tokenizer = AutoTokenizer.from_pretrained(model_id)

    # Saving the quantized model
    model.save_pretrained(save_path)
    tokenizer.save_pretrained(save_path)


.. _BuildingModelArtifact:

Converting a Model to a Model Artifact
===============================================
The Furiosa-LLM :ref:`ArtifactBuilderClass` API offers a variety of options to customize the model artifact.
The ``model_id_or_path`` parameter accepts either a Hugging Face model ID (e.g., ``Qwen/Qwen3-32B``)
or a local path, allowing you to build artifacts from pre-trained, fine-tuned, or quantized models.

Additionally, the API allows you to define bucket configurations for prefill and decode phases,
as well as token-wise sequence lengths to optimize the model performance based on your workload.
You can also specify parallelization strategies such as tensor parallelism and pipeline parallelism
to efficiently utilize multiple NPUs for inference.

The following example demonstrates how to create a model artifact from a quantized model
with specific bucket configurations and tensor parallelism settings.

.. code-block:: python

    from furiosa_llm.artifact import ArtifactBuilder, BucketConfig, ParallelConfig

    builder = ArtifactBuilder(
        # Path to the quantized model directory or model ID from Hugging Face Hub
        model_id_or_path="./qwen3-32b-fp8-dynamic",
        bucket_config=BucketConfig(
            prefill_buckets=[(1, 128)],  # Prefill bucket configuration
            decode_buckets=[(4, 256)],   # Decode bucket configuration
            tokenwise_seq_lens=[128],    # Token-wise sequence lengths
        ),
        parallel_config=ParallelConfig(
            tensor_parallel_size=32,   # Tensor parallelism degree (32 PEs means 4 NPUs)
            pipeline_parallel_size=1,  # Pipeline parallelism degree (default: 1)
        )
    )

    builder.build("./qwen3-32b-fp8-dynamic-artifact")

Once a model artifact is built, you can deploy it to any machine equipped with FuriosaAI RNGD and
run the model using the :ref:`LLMClass` or the appropriate interface like :ref:`OpenAIServer`.

.. tip::

    To achieve better performance or to run LLM models on multiple NPUs,
    you can take advantage of model parallelism in Furiosa-LLM. To learn more about model
    parallelism, please refer to the :ref:`ModelParallelism` section.


Deploying Model Artifacts
=========================
Once you have a model artifact, you can transfer and reuse it on any machine with a
Furiosa NPU and Furiosa-LLM installed.
To transfer a model artifact:

1. Compress the model artifact directory using your preferred compression tool.
2. Copy the compressed file to the target host.
3. Uncompress it on the target machine.
4. Run the model using either the :ref:`LLMClass` or the :ref:`OpenAIServer`.

For quick examples of loading and running model artifacts, refer to the
:ref:`GettingStartedFuriosaLLM` section.
