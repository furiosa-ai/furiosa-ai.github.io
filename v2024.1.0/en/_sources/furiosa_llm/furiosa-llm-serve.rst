.. _OpenAIServer:

****************************************************
OpenAI Compatible Server
****************************************************

In addition to Python API, ``furiosa-llm`` also offers an OpenAI-compatible server
which hosts a single model and provides two OpenAI-compatible APIs:
`Completions API <https://platform.openai.com/docs/api-reference/completions>`_ and
`Chat API <https://platform.openai.com/docs/api-reference/chat>`_.

You can simply launch the server using the ``furiosa-llm serve`` command with an artifact path
as the following.

.. code-block::

    furiosa-llm serve --model [ARTIFACT_PATH]

The following sections describe how to launch and configure the server
and interact with the server using OpenAI API clients.

.. warning::

   This document is based on Furiosa SDK 2024.1.0 (alpha) version,
   and the features and APIs described in this document may change in the future.

Prerequisites
==================================================
To use the OpenAI-Compatible server, you need the following prerequisites:

* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* :ref:`Furiosa LLM Installation <InstallingFuriosaLLM>`
* :ref:`HuggingFace Access Token <AuthorizingHuggingFaceHub>`
* LLM Engine Artifact
* Chat template for chat application (Optional)


Chat Templates
================================================
To use the language models for chat application, we need a structured string instead of a single string.
It's necessary because the model should be able to understand the context of the conversation,
including the role of the speaker (e.g., "user" and "assistant") and the content of the message.
Similar to tokenization, different models require very different input formats for chat.
That's why we need a chat template.

Furiosa LLM supports chat templates based on Jina2 template engine in the same way as HuggingFace Transformers.
`Chat Templates <https://huggingface.co/docs/transformers/en/chat_templating>`_ offers
a comprehensive guide on what chat templates are and how to write your chat templates.
You can also find one good example of chat template at `Llama 3.1 Model Card <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/>`_.

The following command launches the server with the chat template:

.. code-block::

    furiosa-llm serve --model [ARTIFACT_PATH] --chat-template [CHAT_TEMPLATE_PATH]


Arguments of ``furiosa-llm serve`` command
================================================
By default, the server binds to ``localhost:8000``, and
you can change the host and port using the ``--host`` and ``--port`` options.
The following is the list of options and arguments for the serve command:

.. code-block::

    usage: furiosa-llm serve [-h] --model MODEL [--host HOST] [--port PORT] --chat-template CHAT_TEMPLATE [--response-role RESPONSE_ROLE] [-tp TENSOR_PARALLEL_SIZE] [-pp PIPELINE_PARALLEL_SIZE] [-dp DATA_PARALLEL_SIZE] [--devices DEVICES]

    options:
    -h, --help            show this help message and exit
    --model MODEL         The Hugging Face model id, or path to Furiosa model artifact. Currently only one model is supported per server.
    --host HOST           Host to bind the server to (default: '0.0.0.0')
    --port PORT           Port to bind the server to (default: 8000)
    --chat-template CHAT_TEMPLATE
                            Path to chat template file (must be a jinja2 template)
    --response-role RESPONSE_ROLE
                            Response role for /v1/chat/completions API (default: 'assistant')
    -tp TENSOR_PARALLEL_SIZE, --tensor-parallel-size TENSOR_PARALLEL_SIZE
                            Number of tensor parallel replicas. (default: 4)
    -pp PIPELINE_PARALLEL_SIZE, --pipeline-parallel-size PIPELINE_PARALLEL_SIZE
                            Number of pipeline stages. (default: 1)
    -dp DATA_PARALLEL_SIZE, --data-parallel-size DATA_PARALLEL_SIZE
                            Data parallelism size. If not given, it will be inferred from total avaialble PEs and other parallelism degrees.
    --devices DEVICES     Devices to use (e.g. "npu:0:*,npu:1:*"). If unspecified, all available devices from the host will be used.


Using OpenAI API with Furiosa LLM
=====================================================
Once the server is launched, you can interact with the server using HTTP clients
as the following ``CURL`` command example.

.. code-block:: sh

    curl http://localhost:8000/v1/chat/completions \
        -H "Content-Type: application/json" \
        -d '{
        "model": "EMPTY",
        "messages": [{"role": "user", "content": "Say this is a test!"}]
        }' | python -m json.tool


You can use OpenAI client to interact with the server.
To use OpenAI client, you need to install the ``openai`` package first.

.. code-block:: sh

    pip install openai


OpenAI client provides two APIs: ``client.chat.completions`` and ``client.completions``.
You can use the ``client.chat.completions``
API with ``stream=True`` for streaming responses, as following:

.. code-block::

    from openai import OpenAI

    # Replace the following with your base URL
    base_url = f"http://localhost:8000/v1"
    api_key = "EMPTY"

    client = OpenAI(api_key=api_key,base_url=base_url)

    stream = client.chat.completions.create(
        model="EMPTY",
        messages=[{"role": "user", "content": "Say this is a test"}],
        stream=True,
    )

    for chunk in stream:
        if chunk.choices[0].delta.content is not None:
            print(chunk.choices[0].delta.content, end="")


The compatibility with OpenAI API
=================================================

Currently, ``furiosa serve`` supports the following OpenAI API parameters:
You can find more about each parameter at `Completions API <https://platform.openai.com/docs/api-reference/completions>`_
and `Chat API <https://platform.openai.com/docs/api-reference/chat>`_.

.. warning::

    Please note that using ``use_beam_search`` with ``stream`` is not allowed
    because the beam search cannot determine the tokens until the end of the sequence.

    In 2024.1 release, ``n`` works only for beam search and it will be fixed in the next release.

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

Launching the OpenAI-Compatible Server Container
================================================================

Furiosa LLM can be launched immedaitely as a containerized server.

.. code-block::

    # Download the chat template for LLama 3.1 Instruct Model if necessary
    wget https://gist.githubusercontent.com/hyunsik/16f1906af7ac2b4db41af9957a66e168/raw/62935b0c24c03669208cf90f3f87b1694521053d/Llama-3.1-Instruct.tpl

    docker run -it --rm --privileged \
        --env HF_TOKEN=$HF_TOKEN \
        -v ./Llama-3.1-8B-Instruct:/model \
        -v ./Llama-3.1-Instruct.tpl:/Llama-3.1-Instruct.tpl \
        -p 8000:8000 \
        furiosaai/furiosa-llm:2024.1.0 \
        serve --model /model --chat-template /Llama-3.1-Instruct.tpl
