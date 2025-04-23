.. _GettingStartedFuriosaLLM:

**********************************
Quick Start with Furiosa-LLM
**********************************

Furiosa-LLM is a serving framework for LLM models that uses FuriosaAI's NPU.
It provides a Python API compatible with vLLM and a server compatible with
OpenAI's API.
This document explains how to install and use Furiosa-LLM.

.. warning::

  This document is based on Furiosa SDK |release| (beta0).
  The features and APIs described herein are subject to change in the future.


.. _InstallingFuriosaLLM:

Installing Furiosa-LLM
=========================================

The minimum requirements for Furiosa-LLM are as follows:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* Python 3.9, 3.10, 3.11, or 3.12
* PyTorch 2.5.1
* Sufficient storage space for model weights (varies depending on the model size)


To install the ``furiosa-compiler`` package and Furiosa-LLM,
run the following commands:

.. code-block:: sh

  sudo apt install -y furiosa-compiler

  pip install --upgrade pip setuptools wheel
  pip install --upgrade furiosa-llm


.. _AuthorizingHuggingFaceHub:

Authorizing Hugging Face Hub (Optional)
---------------------------------------------
Some models, such as meta-llama/Llama-3.1-8B, require a license to run.
For these, you need to create a Hugging Face account, accept the model's
license, and generate a token.
You can create your token at https://huggingface.co/settings/tokens.
Once you get a token, you can authenticate on the Hugging Face Hub as follows:

.. code-block::

  pip install --upgrade "huggingface_hub[cli]"
  huggingface-cli login --token $HF_TOKEN

Offline Batch Inference with Furiosa-LLM
========================================
We now explain how to perform offline LLM inference using the Python API of Furiosa-LLM.
First, import the ``LLM`` and ``SamplingParams`` classes from the furiosa_llm module.
The ``LLM`` class is used to load LLM models and provides the core API for LLM inference.
``SamplingParams`` is used to specify various parameters for text generation.

.. literalinclude:: ../../../examples/offline_batch_inference.py
   :language: python

Streaming Inference with Furiosa-LLM
====================================
In addition to batch inference, Furiosa-LLM also supports streaming inference.
The key difference of streaming inference is that tokens are returned as soon
they are generated.
This allows you to start printing or processing partial tokens before the whole
inference process finishes.
To perform streaming inference, use the ``stream_generate`` method instead of
``generate``.
This method is asynchronous and returns a stream of tokens as they are generated.

.. literalinclude:: ../../../examples/streaming_inference.py
   :language: python


Chat Inference with Furiosa-LLM
=========================================
Furiosa-LLM provides a high-level ``chat`` method for models with chat capabilities.
This method simplifies interactions by handling prompt templating internally - it applies the appropriate chat template
and invokes the ``generate`` method with the formatted prompt.

For detailed examples of using the chat API, refer to:

* :ref:`FuriosaLLMExamplesChat`
* :ref:`FuriosaLLMExamplesChatWithTools`


Launching the OpenAI-Compatible Server
=========================================

Furiosa-LLM can be deployed as a server that provides an API compatible with
OpenAI's.
Since many LLM frameworks and applications are built on top of OpenAI's API,
you can easily integrate Furiosa-LLM into your existing applications.

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
    "model": "meta-llama/Llama-3.1-8B-Instruct",
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
