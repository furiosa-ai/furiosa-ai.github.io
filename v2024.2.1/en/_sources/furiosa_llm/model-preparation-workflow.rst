.. _ModelPreparationWorkflow:

****************************************************
Model Preparation Workflow
****************************************************

This document describes how to prepare the model of Furiosa LLM.
At a high level, the workflow consists of the following steps:

1. Loading a model from Hugging Face Hub or a local disk
2. (Optional) Calibrating the model and exporting the quantized model to a checkpoint
3. Building a model artifact
4. Deploying the model

The 2nd step is optional. It's necessary if you need to calibrate the model, quantize the model
to an lower precision like INT4, INT8, or FP8.

Prerequisites
==================================================
For the following instructions, you need the following prerequisites:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`

Loading a model from Hugging Face Hub
=================================================================
The Hugging Face Hub is a platform with over 900k models and 200k datasets which are open source and publicly available.
Hugging Face provides a Python library called `transformers` to easily load and use these models.
Furiosa LLM provides a compatible API with the `transformers` library,
so you just need to replace the `transformers` library with the `furiosa_llm.optimum` library
to load a model from the Hugging Face Hub or a local directory.

.. code-block:: python

    # from transformers import AutoModelForCausalLM
    from furiosa_llm.optimum import AutoModelForCausalLM, AutoTokenizer

    config = AutoModelForCausalLM.from_pretrained("meta-llama/Meta-Llama-3.1-8B-Instruct")

Please note that you can run BF16 models directly on Furiosa LLM without further steps,
such as calibration and quantization. For quantization,
you can directly use the `QuantizerForCausalLM` class to load a model from the Hugging Face Hub or a local directory.

.. code-block:: python

    from furiosa_llm.optimum import QuantizerForCausalLM

    model_id = "meta-llama/Meta-Llama-3.1-8B-Instruct"
    quantizer = QuantizerForCausalLM.from_pretrained(model_id)


Instead of ``model_id``, you can specify the path to the model artifact directory,
which can be usually a downloaded model from the Hugging Face Hub.


Calibrating and Quantizing the model
=================================================================================
Generally, quantization is a technique to reduce the required computational and memory resources for inference
while still preserving the accuracy of your model by mapping the high precision space of
activation, weight, KV cache to lower precision INT8, FP8, or INT4 space.

Currently, Furiosa LLM provides the PTQ (Post Training Quantization) technique to quantize a model.
To quantize a model, you need to calibrate the model with a calibration dataset and export the quantized model to a checkpoint.
``create_data_loader`` function creates a dataloader for calibration by specifying
the tokenizer, dataset name or path, the dataset split, the number of samples, and the maximum sample length.

.. tip::

    ``create_data_loader`` is based on the ``datasets``,
    which is a library for easily accessing and sharing datasets
    for Audio, Computer Vision, and Natural Language Processing (NLP) tasks.
    You can learn more about the datasets library in the
    `datasets <https://huggingface.co/docs/datasets/en/index>`_ documentation.
    You can find more datasets at https://huggingface.co/datasets.

``QuantizerForCausalLM`` class provides ``quantize()`` method to quantize the text-generation model
with the calibration dataset and export the quantized model to a checkpoint.
The first argument of ``QuantizerForCausalLM`` is the model id of Hugging Face Hub or path to the model artifact.
``quantize()`` method takes the quantized model, dataloader, and quantization configuration as arguments.

Here is an example of quantizing a model:

.. literalinclude:: ../../../examples/optimum_quantize.py
  :language: python

``QuantizationConfig.w_f8_a_f8_kv_f8()`` is a quantization configuration
that quantizes the weights, activations, and KV cache to FP8 space.

Once you quantize the model, you will get the quantized model checkpoint in the ``save_dir`` path.

Building a Model Artifact
==================================================
The next step is to build a model artifact.
A model artifact is a set of files, including model weights, configuration,
compiled NPU program binary files and other necessary information to run
the model on the Furiosa LLM engine.

Once a model artifact is built, you can deploy the model artifact to any host
with RNGD and Furiosa LLM installed. Then, you can run the model
on the Furiosa LLM engine without any additional steps.

How to build a Furiosa LLM model artifact
**********************************************

Once you have a quantized model, you can build a model artifact using
the :ref:`ArtifactBuilderClass` API or via the ``furiosa-llm build`` command.

Here is an example of building a model artifact using the :ref:`ArtifactBuilderClass` API.

.. literalinclude:: ../../../examples/artifact_builder.py
  :language: python

``ArtifactBuilder`` class provides various options to build a furiosa-llm model artifact.
The options include devices, parallelism degrees, prefill buckets, decode buckets, and other configurations.
You can find more details about the arguments in the :ref:`ArtifactBuilderClass` document.

.. Also, you can use the :ref:`FuriosaLLMBuildCommand` to build a model artifact via a CLI command.


.. tip::

    To achieve better performance or to run large language models on multiple NPUs,
    you can use the model parallelism in Furiosa LLM. To learn more the model parallelism,
    please refer to :ref:`ModelParallelism` document.


Deploying the model
==================================================
Once you have a model artifact, you can just copy the model artifact directory.
You can use your preferred compression tool to compress the model artifact directory and copy it to the target host.
Then, you can uncompress the model artifact directory and run the model through :ref:`LLMClass` or :ref:`OpenAIServer`.
