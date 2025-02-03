.. _GettingStartedFuriosaLLM:

**********************************
Quick Start with Furiosa LLM
**********************************

Furiosa LLM is a serving framework for LLM models that uses FuriosaAI's NPU.
It provides a Python API compatible with vLLM and a server compatible with
OpenAI's API.
This document explains how to install and use Furiosa LLM.

.. warning::

  This document is based on Furiosa SDK |release| (beta0).
  The features and APIs described herein are subject to change in the future.


.. _InstallingFuriosaLLM:

Installing Furiosa LLM
=========================================

The minimum requirements for Furiosa LLM are as follows:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* Python 3.8, 3.9, or 3.10
* PyTorch 2.4.1
* Enough storage space for model weights, e.g., about 100 GB for the
  Llama 3.1 70B model

To install the ``furiosa-compiler`` package and the Furiosa LLM,
run the following commands:

.. code-block:: sh

  sudo apt install -y furiosa-compiler

  pip install --upgrade pip setuptools wheel
  pip install --upgrade furiosa-llm


.. _AuthorizingHuggingFaceHub:

Authorizing Hugging Face Hub (Optional)
---------------------------------------------
Some models, such as meta-llama/Meta-Llama-3.1-8B require a license to run.
For these, you need to create a Hugging Face account, accept the model's
license, and generate a token.
You can create your token at https://huggingface.co/settings/tokens.
Once you get a token, you can authenticate on the Hugging Face Hub as follows:

.. code-block::

  pip install --upgrade "huggingface_hub[cli]"
  huggingface-cli login --token $HF_TOKEN

Building Model Artifacts
==========================

To run Furiosa LLM with a given model, you need to build a model artifact first.
This process starts with a pre-trained model from the Hugging Face Hub,
and involves calibration and quantization, and compilation, ultimately generating
an artifact. Furiosa LLM provides a Python API to perform these steps simply.

.. note::

  If you already have a pre-built model artifact, you can skip this section.
  According to our roadmap, the 2025.2 release will allow BF16 models
  to run on Furiosa LLM without the calibration and quantization steps.
  So, the following steps will be much simpler.

The following examples show how to build a model artifact
from a pre-trained model.

.. literalinclude:: ../../../examples/optimum_quantize.py
  :language: python

The above code snippet shows how to quantize a model using the ``QuantizerForCausalLM`` class.
Eventually, it will save the quantized model to the specified directory.

.. literalinclude:: ../../../examples/artifact_builder.py
  :language: python

``ArtifactBuilder`` applies the parallelism strategy to the quantized model,
compiles the parallelized model, and eventually generates a model artifact.
More details and examples can be found in the :ref:`ModelPreparationWorkflow` section.


Offline Batch Inference with Furiosa LLM
========================================
We now explain how to perform offline LLM inference using the Python API of Furiosa LLM.
First, import the ``LLM`` and ``SamplingParams`` classes from the furiosa_llm module.
The ``LLM`` class is used to load LLM models and provides the core API for LLM inference.
``SamplingParams`` is used to specify various parameters for text generation.

.. literalinclude:: ../../../examples/offline_batch_inference.py
   :language: python

Streaming Inference with Furiosa LLM
====================================
In addition to batch inference, Furiosa LLM also supports streaming inference.
The key difference of streaming inference is that tokens are returned as soon
they are generated.
This allows you to start printing or processing partial tokens before the whole
inference process finishes.
To perform streaming inference, use the ``stream_generate`` method instead of
``generate``.
This method is asynchronous and returns a stream of tokens as they are generated.

.. literalinclude:: ../../../examples/streaming_inference.py
   :language: python

Launching the OpenAI-Compatible Server
=========================================

Furiosa LLM can be deployed as a server that provides an API compatible with
OpenAI's.
Since many LLM frameworks and applications are built on top of OpenAI's API,
you can easily integrate Furiosa LLM into your existing applications.

By default, the server listens on the HTTP endpoint http://localhost:8000.
You can change the binding address and port by specifying the ``--host`` and ``--port`` options.
The server can host only one model at a time for now and provides a chat template feature.
You can find more details in the :ref:`OpenAIServer` section.

Below is an example of how to launch the server with the Llama 3.1 8B Instruct model.

.. literalinclude:: ../../../examples/launch_llm_serve.sh
  :language: sh

The server loads the model and starts listening on the specified port.
When the server is ready, you will see the following message:

.. code-block:: sh

  INFO:     Started server process [27507]
  INFO:     Waiting for application startup.
  INFO:     Application startup complete.
  INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)


Then, you can test the server using the following curl command:

.. literalinclude:: ../../../examples/curl_chat_completions.sh
  :language: sh

Example output:

.. code-block:: json

  {
    "id": "chat-21f0b74b2c6040d3b615c04cb5bf2e2e",
    "object": "chat.completion",
    "created": 1736480800,
    "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "The capital of France is Paris.",
                "tool_calls": []
            },
            "logprobs": null,
            "finish_reason": "stop",
            "stop_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 42,
        "total_tokens": 49,
        "completion_tokens": 7
    },
    "prompt_logprobs": null
  }


Using Chat Templates with Furiosa LLM
=========================================
Chat models are usually trained with a variety of prompt formats.
In particular, Llama 3.x models require a specific prompt format to leverage
multiple tools.
You can find a full guide to prompt formatting in the
`Llama model card <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/>`_.

When using the OpenAI-Compatible Server with a chat model, ``furiosa-llm serve``
will automatically apply the chat template if the model's tokenizer provides it.
Additionally, you can use the ``--chat-template`` option to specify a custom
chat template path.

.. note::

  The Chat API is not yet supported in furiosa-llm.
  Support is planned for the 2025.1 release.

If you are using the LLM API, you will need to manually apply the chat template
to the prompt for now.
Since furiosa-llm does not yet provide a chat API, you need to use the tokenizer
to apply the chat template to the prompt.

.. literalinclude:: ../../../examples/chat_template.py
  :language: python
