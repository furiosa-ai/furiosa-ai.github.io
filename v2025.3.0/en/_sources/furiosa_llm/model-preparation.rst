.. _ModelPreparation:

****************************************************
Model Preparation
****************************************************
To run an LLM model on the Furiosa NPU, Furiosa-LLM must convert the model into a model artifact.
During the conversion process, Furiosa-LLM applies a variety of
optimizations to enable high-performance inference. This document describes the overall workflow for
preparing a model artifact from an LLM model and deploying it.

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
However, depending on your network environment, downloading the model weights may take a long time.
If you want to download a model from the Hugging Face Hub in advance, you can do so using the
``huggingface-cli`` command.

The following command downloads the model weights and configuration files
for the Llama 3.1 8B model to the Hugging Face Hub cache directory.

.. code-block::

  huggingface-cli download "meta-llama/Llama-3.1-8B-Instruct"


.. _BuildingModelArtifact:

Optimize and Convert Models to a Model Artifact
===============================================
Furiosa-LLM provides a command-line tool, ``furiosa-llm build``, to optimize and convert models
into model artifacts.

The following shows an example for building a model artifact for ``meta-llama/Llama-3.1-8B-Instruct``
and saving it to the ``./Output-Llama-3.1-8B-Instruct`` directory. The ``-tp`` option specifies the tensor parallelism degree,
and ``--max-seq-len-to-capture`` defines the maximum sequence length that the model can handle.

.. literalinclude:: ../../../examples/model-prep-build-cmd.sh
  :language: sh

Once a model artifact is built, you can deploy it to any machine equipped with FuriosaAI RNGD and
run the model using the :ref:`LLMClass` or the appropriate interface like :ref:`OpenAIServer`.

.. tip::

    To achieve better performance or to run LLM models on multiple NPUs,
    you can take advantage of model parallelism in Furiosa-LLM. To learn more about model
    parallelism, please refer to the :ref:`ModelParallelism` section.


You can also build a model artifact using the :ref:`ArtifactBuilderClass` API.
Here is an example using the :ref:`ArtifactBuilderClass` API.

.. literalinclude:: ../../../examples/model-prep-build-api.py
  :language: python


Both ``furiosa-llm build`` and :ref:`ArtifactBuilderClass` API offer a variety of options to customize the model artifact.
You can specify the tensor parallelism degree, pipeline parallelism degree, data parallelism degree,
prefill and decode bucket sizes, and other options. Please refer to
:ref:`BuildingModelArtifactsByExamples` section for more examples and details.

.. _ModelQuantization:

Model Quantization (Optional)
====================================
Quantization is a widely used technique to reduce the computational and memory requirements for inference
by mapping the high-precision space of activations, weights, and KV cache to lower-precision formats
such as INT8, FP8, or INT4 — while aiming to preserve model accuracy.

It is typically applied when higher throughput or lower latency is needed. However, since quantization may affect
model accuracy, it is important to perform thorough experimentation and accuracy evaluations.

Furiosa-LLM currently supports Post-Training Quantization (PTQ) for model quantization.
To apply PTQ, you need to calibrate the model using a calibration dataset and then export the quantized model as a checkpoint.
The following sections explain the PTQ workflow with Furiosa-LLM.


Load a Model to Quantize
-------------------------------------
The first step is to prepare a model to quantize.
The ``QuantizerForCausalLM`` class provides a simple API to load a
model from either the Hugging Face Hub or a local path.
``QuantizerForCausalLM`` is a subclass of `AutoModelForCausalLM <https://huggingface.co/docs/transformers/en/model_doc/auto#transformers.AutoModelForCausalLM>`_,
so it automatically determines the model class from the Hugging Face model ID in the same way as
``AutoModelForCausalLM``.

.. code-block:: python

    from furiosa_llm.optimum import QuantizerForCausalLM

    model_id = "meta-llama/Llama-3.1-8B-Instruct"
    quantizer = QuantizerForCausalLM.from_pretrained(model_id)


Calibrate and Quantize the Model
--------------------------------
Once a model is loaded, you can calibrate and quantize it by calling the
``QuantizerForCausalLM.quantize()`` method.

The ``quantize()`` method takes as arguments the model to be quantized, a data loader, and a
quantization configuration. The ``create_data_loader`` function helps generate a data loader
that supplies the quantization process with an appropriate sample dataset.
When creating a data loader, you can configure parameters such as the tokenizer, dataset name or path, dataset split, number of samples, and maximum sample length. These parameters can significantly impact the accuracy of the quantized model, so some experimentation is typically necessary to determine the optimal settings.

.. tip::

    ``create_data_loader`` is based on the ``datasets`` library,
    which provides easy access to datasets for tasks in audio, computer vision, and
    natural language processing (NLP).
    Learn more in the
    `datasets documentation <https://huggingface.co/docs/datasets/en/index>`_
    and explore the available datasets at https://huggingface.co/datasets.

The following example demonstrates how to create a data loader for the calibration dataset.
The quantized model will be saved to the ``save_dir`` directory.

.. literalinclude:: ../../../examples/optimum_quantize.py
  :language: python

The ``QuantizationConfig`` class allows you to specify various quantization options
and offers a set of pre-defined quantization configurations.
For example, ``QuantizationConfig.w_f8_a_f8_kv_f8()``, quantizes the weights, activations, and KV cache to 8-bit floating-point (FP8).

Once you have the quantized model, you can create a model artifact using either the
``ArtifactBuilder`` API or the ``furiosa-llm build`` command. Below is an example of using the
``furiosa-llm build`` command to generate a model artifact from the quantized model.


.. literalinclude:: ../../../examples/furiosa_llm_build_cmd.sh
  :language: sh


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
