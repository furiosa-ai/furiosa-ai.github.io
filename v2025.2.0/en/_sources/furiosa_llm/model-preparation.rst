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
  Also, Furiosa-LLM provides a set of pre-compiled model artifacts for popular LLMs in the
  `Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_.
  You can use them to quickly run LLM models on the Furiosa NPU.


.. .. figure:: ../_static/imgs/model_preparation_workflow.png
..   :alt: Model Preparation Workflow
..   :width: 900px
..   :class: only-light
..   :align: center

.. .. figure:: ../_static/imgs/model_preparation_workflow.png
..   :alt: Model Preparation Workflow
..   :width: 900px
..   :class: only-dark
..   :align: center

Prerequisites
=============
Ensure that you meet the following prerequisites before starting the model preparation workflow:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa-LLM <InstallingFuriosaLLM>`
* A :ref:`Hugging Face access token <AuthorizingHuggingFaceHub>`
* Sufficient storage space for model weights, e.g., about 100 GB for the
  Llama 3.1 70B model

Authorizing Hugging Face Hub (Optional)
=================================================
Some models, such as meta-llama/Llama-3.1-8B, require a license to run.
For these, you need to create a Hugging Face account, accept the model's
license, and generate a token.
You can create your token at https://huggingface.co/settings/tokens.
Once you get a token, you can authenticate on the Hugging Face Hub as follows:

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
Furiosa-LLM provides a command line tool ``furiosa-llm build`` to optimize and convert models
into model artifacts.

The following shows an example for building a model artifact for ``meta-llama/Llama-3.1-8B-Instruct``
and save it to the ``./Output-Llama-3.1-8B-Instruct`` directory. ``-tp`` is the tensor parallelism degree,
and ``--max-seq-len-to-capture`` is the maximum sequence length that the model can handle.

.. literalinclude:: ../../../examples/furiosa_llm_build_cmd.sh
  :language: sh

Once a model artifact is built, you can deploy it to any machine equipped with FuriosaAI RNGD and
run the model using the :ref:`LLMClass` or the appropriate interface like :ref:`OpenAIServer`.

.. tip::

    To achieve better performance or to run LLM models on multiple NPUs,
    you can take advantage of model parallelism in Furiosa-LLM. To learn more about model
    parallelism, please refer to the :ref:`ModelParallelism` section.


You can also build a model artifact using either the :ref:`ArtifactBuilderClass` API.
Here is an example using the :ref:`ArtifactBuilderClass` API.

.. literalinclude:: ../../../examples/artifact_builder.py
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
model accuracy, it is important to perform thorough experimentation and accuracy comparisons.

Furiosa-LLM currently supports Post-Training Quantization (PTQ) for model quantization.
To apply PTQ, you need to calibrate the model using a calibration dataset and then export the quantized model as a checkpoint.
The following sections explain the PTQ workflow with Furiosa-LLM.


Load a Model to Quantize
-------------------------------------
The first step is to prepare a model to quantize.
The ``QuantizerForCausalLM`` class provides a simple API to load a
model from either the Hugging Face Hub a local path.
``QuantizerForCausalLM`` is a subclass of `AutoModelForCausalLM <https://huggingface.co/docs/transformers/en/model_doc/auto#transformers.AutoModelForCausalLM>`_,
so it finds automatically the model class from the Hugging Face model id in the same way as
``AutoModelForCausalLM``.

.. code-block:: python

    from furiosa_llm.optimum import QuantizerForCausalLM

    model_id = "meta-llama/Llama-3.1-8B-Instruct"
    quantizer = QuantizerForCausalLM.from_pretrained(model_id)


Calibrate and Quantize the Model
--------------------------------
Once a model is loaded, you can calibrate and quantize the model by calling the
``QuantizerForCausalLM.quantize()`` method.

The ``quantize()`` method takes the quantized model, a data loader, and a
quantization configuration as arguments. The ``create_data_loader`` function creates a dataloader
which feeds a quantization process with a proper sample dataset.
When you create a data loader, you can specify the tokenizer, dataset name or path, the dataset split,
the number of samples, and the maximum sample length. These options can impact the accuracy of a quantized model.
So, you will need some experiments to find the best parameters for your model.

.. tip::

    ``create_data_loader`` is based on the ``datasets`` library,
    which provides easy access to datasets for audio, computer vision, and
    natural language processing (NLP) tasks.
    Learn more on the
    `datasets documentation <https://huggingface.co/docs/datasets/en/index>`_
    and explore the datasets at https://huggingface.co/datasets.

The following is an example of creating a data loader for the calibration dataset.
The quantized model is stored to the ``save_dir`` directory.

.. literalinclude:: ../../../examples/optimum_quantize.py
  :language: python

The ``QuantizationConfig`` class allows you to specify various quantization options,
and it also provides a set of pre-defined quantization configurations.
``QuantizationConfig.w_f8_a_f8_kv_f8()`` is one of the pre-defined quantization configuration
that quantizes the weights, activations, and KV cache to 8-bit floats (FP8).

Once you have the quantized model, you can build a model artifact using the
``ArtifactBuilder`` API or the ``furiosa-llm build`` command. The following is an example of using the
``furiosa-llm build`` command to build a model artifact from the quantized model.


.. literalinclude:: ../../../examples/furiosa_llm_build_cmd.sh
  :language: sh


Deploying Model Artifacts
=========================
Once you have a model artifact, you can copy and reuse it on any machine with a
Furiosa NPU and Furiosa-LLM installed.
To transfer a model artifact:

1. Compress the model artifact directory using your preferred compression tool.
2. Copy the file to the target host.
3. Uncompress it on the target machine.
4. Run the model using either the :ref:`LLMClass` or the :ref:`OpenAIServer`.

For quick examples of loading and running model artifacts, refer to the
:ref:`GettingStartedFuriosaLLM` section.
