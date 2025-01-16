.. _GettingStartedFuriosaLLM:

**********************************
Quick Start with Furiosa LLM
**********************************

Furiosa LLM is a serving framework for LLM models that utilizes FuriosaAI's NPU.
It provides a Python API compatible with vLLM and a server compatible with the OpenAI API.
This document explains how to install and use Furiosa LLM.

.. warning::

   This document is based on Furiosa SDK |release| (beta0) version,
   and the features and APIs described in this document may change in the future.


.. _InstallingFuriosaLLM:

Installing Furiosa LLM
=========================================

The minimum requirements for Furiosa LLM are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* Python 3.8, 3.9, or 3.10
* PyTorch 2.4.1
* Enough storage space for model weights; e.g., about 100GB for Llama 3.1 70B model

You need to install ``furiosa-compiler`` and the Furiosa LLM with the following command:

.. code-block:: sh

  sudo apt install -y furiosa-compiler

  pip install --upgrade pip setuptools wheel
  pip install --upgrade furiosa-llm


.. _AuthorizingHuggingFaceHub:

Authorizing HuggingFace Hub (Optional)
---------------------------------------------
Some models, such as meta-llama/Meta-Llama-3.1-8B require you to accept their license,
hence, you need to create a HuggingFace account, accept the model's license, and generate a token.
You can create your token at https://huggingface.co/settings/tokens.
Once you get a token, you can authenticate on the HuggingFace Hub as following:

.. code-block::

  huggingface-cli login --token $HF_TOKEN


Offline Batch Inference with Furiosa LLM
------------------------------------------------------
In this section, we will explain how to perform offline LLM inference using the Python API of Furiosa LLM.
First, import the ``LLM`` class and ``SamplingParams`` from the furiosa_llm module.
``LLM`` class is used to load LLM models and provides the core API for LLM inference.
``SamplingParams`` allows to specify various parameters for text generation.

.. literalinclude:: ../../../examples/offline_batch_inference.py
   :language: python

Streaming Inference with Furiosa LLM
------------------------------------------------------
In addition to batch inference, Furiosa LLM also supports streaming inference.
The key difference of streaming inference is that tokens are returned as soon as tokens are generated, allowing you to start printing or processing partial tokens before the full completion is finished.
To perform streaming inference, the ``stream_generate`` method is used instead of ``generate``. This method is asynchronous and returns a stream of tokens as they are generated.

.. literalinclude:: ../../../examples/streaming_inference.py
   :language: python

Launching the OpenAI-Compatible Server
=========================================

Furiosa LLM can be deployed as a server that provides an API compatible with OpenAI API.
Since many LLM frameworks and applications are built on top of OpenAI API protocol,
you can easily integrate Furiosa LLM into your existing applications.

By default, the server provides the HTTP endpoint http://localhost:8000.
You can change the binding address and port by specifying the ``--host`` and ``--port`` options.
The server can host only one model at a time for now and provides a chat template feature.
You can find more details at :ref:`OpenAIServer`.

The following is an example of launching the server with the Llama 3.1 8B Instruct model.

.. literalinclude:: ../../../examples/launch_llm_serve.sh
  :language: sh

You can simply test the server using the following curl command:

.. literalinclude:: ../../../examples/curl_chat_completions.sh
  :language: sh

Output:

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
Chat models have been trained with very different prompt formats.
Especially, Llama 3.x models require a specific prompt format to leverage multiple tools.
You can see a full guide to prompt formatting at `Llama model card <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/>`_

If you use the OpenAI-Compatible Server with a chat model, ``furiosa-llm serve`` will automatically apply the chat template
if the model's tokenizer provides its chat template. Also, you can use ``--chat-template`` option to specify that chat template path.

.. note::

  Chat API is not yet supported in furiosa-llm. Chat API is planned to be supported in 2025.1 release.

If you use LLM API, you still need to apply the chat template to the prompt manually for now.
Currently, furiosa-llm does not provide chat API, so you need to use tokenizer to apply a chat template to the prompt.

.. literalinclude:: ../../../examples/chat_template.py
  :language: python
