.. _ModelPreparationWorkflow:

****************************************************
Model Preparation Workflow
****************************************************

This document describes how to prepare the model of Furiosa LLM.
At a high level, the workflow consists of the following four steps:

1. Loading a model from Hugging Face Hub or a local disk
2. Calibrating the model and exporting the quantized model to a checkpoint
3. Building a model artifact
4. Deploying the model

.. note::

  The latest release |release| always requires the calibration and quantization steps for all
  models. The 2025.2.0 release will allow BF16 models to run on Furiosa LLM without the calibration and quantization steps.
  The 2nd step will be required only for INT8, FP8, and INT4 models.


.. figure:: ../_static/imgs/model_preparation_workflow.png
  :alt: Model Preparation Workflow
  :width: 720px
  :class: only-light
  :align: center

.. figure:: ../_static/imgs/model_preparation_workflow.png
  :alt: Model Preparation Workflow
  :width: 720px
  :class: only-dark
  :align: center

Prerequisites
==================================================
Please ensure you meet the prerequisites before starting the model preparation workflow:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`
* Sufficient Storage space for model weights, e.g., about 100 GB for the Llama 3.1 70B model

Loading a model from Hugging Face Hub
=================================================================
The Hugging Face Hub is a platform with over 900k models and 200k datasets which are open source
and publicly available. You can load a model from the Hugging Face Hub using the
``furiosa_llm.optimum`` library. The ``QuantizerForCausalLM`` class provides a simple API to load a
model from the Hugging Face Hub and a ``quantize()`` method to calibrate and quantize the model.
``QuantizerForCausalLM`` is a subclass of `AutoModelForCausalLM <https://huggingface.co/docs/transformers/en/model_doc/auto#transformers.AutoModelForCausalLM>`_,
so it finds automatically the model class from the Hugging Face model id in the same way as
``AutoModelForCausalLM``.

.. code-block:: python

    from furiosa_llm.optimum import QuantizerForCausalLM

    model_id = "meta-llama/Meta-Llama-3.1-8B-Instruct"
    quantizer = QuantizerForCausalLM.from_pretrained(model_id)


Calibrating and Quantizing the model
=================================================================================
Quantization is a popular technique to reduce the required computational and memory resources for
inference while preserving the accuracy of your model by mapping the high precision space of
activations, weights, and KV cache to lower precision INT8, FP8, or INT4 space.

Currently, Furiosa LLM provides the PTQ (Post Training Quantization) technique to quantize a model.
To quantize a model, you need to calibrate the model with a calibration dataset and export the
quantized model to a checkpoint. The ``create_data_loader`` function creates a dataloader for
calibration by specifying the tokenizer, dataset name or path, the dataset split, the number of
samples, and the maximum sample length.

.. tip::

    ``create_data_loader`` is based on the ``datasets``,
    which is a library for easily accessing and sharing datasets
    for Audio, Computer Vision, and Natural Language Processing (NLP) tasks.
    You can learn more about the datasets library in the
    `datasets <https://huggingface.co/docs/datasets/en/index>`_ documentation.
    You can find more datasets at https://huggingface.co/datasets.

``QuantizerForCausalLM`` class provides ``quantize()`` method to quantize the text-generation model
with the calibration dataset and export the quantized model to a checkpoint.
The first argument of ``QuantizerForCausalLM`` is the model id of Hugging Face Hub or the path to the
model artifact. The ``quantize()`` method takes the quantized model, a data loader, and a quantization
configuration as arguments.

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
The options include devices, parallelism degrees, prefill buckets, decode buckets, and other
configurations. You can find more details about the arguments in the :ref:`ArtifactBuilderClass`
document.

.. tip::

    To achieve better performance or to run large language models on multiple NPUs,
    you can make good use of the model parallelism in Furiosa LLM. To learn more about model
    parallelism, please refer to :ref:`ModelParallelism` document.


Deploying the Built Model Artifact
==================================
Once you have a model artifact, you can copy and reuse the model artifact directory
anywhere RNGD NPUs and Furiosa LLM are installed. You can use your preferred compression tool to
compress the model artifact directory and copy it to the target host. Then, you can uncompress
the model artifact directory and run the model through the :ref:`LLMClass` or the :ref:`OpenAIServer`.
You can find quick examples for loading and running model artifacts using Furiosa LLM
in the :ref:`GettingStartedFuriosaLLM` section.


Current Limitations
===================
* ``ArtifactBuilder`` currently supports up to 8k context length even though Furiosa LLM supports
  up to 32k context length. So, you can run 32k context length models only through the pre-built
  artifacts that the FuriosaAI provides. This limitation will be removed in the 2025.1.0 release.
* The current version of ArtifactBuilder requires models quantized by ``QuantizerForCausalLM``.
  The 2025.2.0 release will allow BF16 models to run on Furiosa
  LLM without the calibration and quantization steps.
