.. _OpenAIServer:

****************************************************
OpenAI-Compatible Server
****************************************************

In addition to the Python API, Furiosa LLVM offers an OpenAI-compatible server
that hosts a single model and provides two OpenAI-compatible APIs:
`Completions API <https://platform.openai.com/docs/api-reference/completions>`_ and
`Chat API <https://platform.openai.com/docs/api-reference/chat>`_.

To launch the server, use the ``furiosa-llm serve`` command with the model
artifact path, as follows:

.. code-block::

    furiosa-llm serve [ARTIFACT_PATH]

The following sections describe how to launch and configure the server
and interact with the server using OpenAI API clients.

.. warning::

  This document is based on Furiosa SDK |release| (beta0).
  The features and APIs described herein are subject to change in the future.


Prerequisites
=============
To use the OpenAI-Compatible server, you need the following:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa LLM <InstallingFuriosaLLM>`
* A :ref:`Hugging Face access token <AuthorizingHuggingFaceHub>`
* A model artifact
* Chat template for chat application (Optional)


Using the OpenAI API
====================
Once the server is running, you can interact with it using an HTTP client,
as shown in the following example:

.. literalinclude:: ../../../examples/curl_chat_completions.sh
  :language: sh

You can also use the OpenAI client to interact with the server.
To use the OpenAI client, you need to install the ``openai`` package first:

.. code-block:: sh

    pip install openai


The OpenAI client provides two APIs: ``client.chat.completions`` and
``client.completions``.
To stream responses, you can use the ``client.chat.completions``
API with ``stream=True``, as follows:

.. literalinclude:: ../../../examples/openai_client.py
  :language: python


By default, the Furiosa LLM server binds to ``localhost:8000``.
You can change the host and port using the ``--host`` and ``--port`` options.


Chat Templates
==============
To use a language model in a chat application, we need to prepare a structured
string to give as input.
This is essential because the model must understand the conversation's context,
including the speaker's role (e.g., "user" and "assistant") and the
message content.
Just as different models require distinct tokenization methods, they also have
varying input formats for chat.
This is why a chat template is necessary.

Furiosa LLM supports chat templates based on the Jinja2 template engine, similar
to Hugging Face Transformers.
If the model's tokenizer includes a built-in chat template,
``furiosa-llm serve`` will automatically use it.
However, if the tokenizer lacks a built-in template, or if you want to override
the default, you can specify one using the ``--chat-template`` parameter.

For reference, you can find a well-structured example of a chat template in the
`Llama 3.1 Model Card <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/>`_.

To launch the server with a custom chat template, use the following command:

.. code-block:: bash

    furiosa-llm serve [ARTIFACT_PATH] --chat-template [CHAT_TEMPLATE_PATH]


.. _ToolCalling:

Tool Calling
============
Furiosa LLM supports tool calling (also known as function calling) for models
trained with this capability.

Within the ``tool_choice`` options supported by the
`OpenAI API <https://platform.openai.com/docs/api-reference/chat/create#chat-create-tool_choice>`_,
Furiosa LLM supports ``"auto"`` and ``"none"``.
Future releases will support ``"required"`` and named function calling.

The system converts model outputs into the OpenAI response format through a
designated parser implementation.
At this time, only the ``llama3_json`` parser is available.
Additional parsers will be introduced in future releases.

The following command starts the server with tool calling enabled for Llama 3.1 models:

.. code-block:: bash

    furiosa-llm serve [ARTIFACT_PATH] --enable-auto-tool-choice --tool-call-parser llama3_json

To use the tool calling feature, specify the ``tools`` and ``tool_choice``
parameters. Here's an example:

.. literalinclude:: ../../../examples/tool_calling.py
  :language: python

The expected output is as follows.

.. code-block:: text

    Function called: get_weather
    Arguments: {"location": "San Francisco, CA", "unit": "F"}
    Result: The temperature in San Francisco, CA is 70 °F


Compatibility with OpenAI API
=============================

Below are the API parameters currently supported by Furiosa LLM:

.. warning::

    Please note that using ``use_beam_search`` together with ``stream`` is not
    allowed because beam search requires the whole sequence to produce the
    output tokens.

    In the 2024.2 release, ``n`` works only for beam search. This limitation
    will be fixed in the next release.

Parameters supported by both the
`Completions <https://platform.openai.com/docs/api-reference/completions>`_ and
`Chat <https://platform.openai.com/docs/api-reference/chat>`_ APIs:

* ``n``
* ``temperature``
* ``top_p``
* ``top_k``
* ``early_stopping``
* ``length_penalty``
* ``max_tokens``
* ``min_tokens``
* ``use_beam_search``
* ``best_of``
* ``stream``

Parameters supported by the
`Chat API <https://platform.openai.com/docs/api-reference/chat>`_ only:

* ``tools``
* ``tool_choice``


Launching the OpenAI-Compatible Server Container
================================================

FuriosaAI offers a containerized server that can be used for faster deployment.
Here is an example that launches the Furiosa LLM server in a Docker container
(replace ``$HF_TOKEN`` with your Hugging Face Hub token):

.. code-block::

    docker run -it --rm --privileged \
        --env HF_TOKEN=$HF_TOKEN \
        -v ./Llama-3.1-8B-Instruct:/model \
        -p 8000:8000 \
        furiosaai/furiosa-llm:latest \
        serve /model --devices "npu:0"
