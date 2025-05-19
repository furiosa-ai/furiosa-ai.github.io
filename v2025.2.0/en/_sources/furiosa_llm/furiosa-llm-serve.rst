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

  This document is based on Furiosa SDK |release|.
  The features and APIs described herein are subject to change in the future.


Prerequisites
=============
To use the OpenAI-Compatible server, you need the following:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa-LLM <InstallingFuriosaLLM>`
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


By default, the Furiosa-LLM server binds to ``localhost:8000``.
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

Furiosa-LLM supports chat templates based on the Jinja2 template engine, similar
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

Tool Calling Support
====================
Furiosa-LLM supports tool calling (also known as function calling) for models
trained with this capability.

Within the ``tool_choice`` options supported by the
`OpenAI API <https://platform.openai.com/docs/api-reference/chat/create#chat-create-tool_choice>`_,
Furiosa-LLM supports ``"auto"`` and ``"none"``.
Future releases will support ``"required"`` and named function calling.

The system converts model outputs into the OpenAI response format through a
designated parser implementation.
At this time, only the ``llama3_json`` parser is available.
Additional parsers will be introduced in future releases.

The following example command starts the server with tool calling enabled for Llama 3.1 models:

.. code-block:: bash

    furiosa-llm serve furiosa-ai/Llama-3.1-8B-Instruct-FP8 --enable-auto-tool-choice --tool-call-parser llama3_json

To use the tool calling feature, specify the ``tools`` and ``tool_choice``
parameters. Here's an example:

.. literalinclude:: ../../../examples/llm_serve_tool_calling.py
  :language: python

The expected output is as follows.

.. code-block:: text

    Function called: get_weather
    Arguments: {"location": "San Francisco, CA", "unit": "fahrenheit"}
    Result: Getting the weather for San Francisco, CA in fahrenheit...


.. _Reasoning:

Reasoning Support
=================

Furiosa-LLM provides support for models with reasoning capabilities, such as Deepseek R1 series. These models follow a structured approach by first conducting reasoning steps and then providing a final answer.

The reasoning process follows this sequence:

* The model-specific start-of-reasoning token is appended to the input prompt through the chat template.
* The model generates its reasoning.
* Once reasoning is complete, the model outputs an end-of-reasoning token followed by the final answer.

Since start-of-reasoning and end-of-reasoning tokens are model-specific, we support different reasoning parsers for different models.
Currently, ``deepseek_r1`` parser is available. This parser expects ``<token>`` and ``</token>`` as the start-of-reasoning and end-of-reasoning tokens respectively.
Any models that follow the same token scheme (such as Qwen QWQ) can use this parser.

To launch a server with reasoning capabilities for Deepseek R1 series, use the following example command:

.. code-block:: bash

    furiosa-llm serve furiosa-ai/DeepSeek-R1-Distill-Llama-8B --enable-reasoning --reasoning-parser deepseek_r1

You can access the reasoning content through these response fields:

* ``response.choices[].message.reasoning_content``
* ``response.choices[].delta.reasoning_content``

Here's an example that demonstrates how to access the reasoning content:

.. literalinclude:: ../../../examples/llm_serve_reasoning.py
  :language: python


.. note::

    The ``reasoning_content`` field is a Furiosa-LLM-specific extension and is not part of the standard OpenAI API.
    This field will appear only in responses that contain reasoning content, and
    attempting to access this field in responses without reasoning content will raise an ``AttributeError``.


Supported OpenAI API Parameters
===============================

The following table outlines the supported parameters for Chat and Completions APIs.
Any parameters not listed in the table are unsupported and will be ignored by the server.

.. warning::

    Please note that using ``use_beam_search`` together with ``stream`` is not
    allowed because beam search requires the whole sequence to produce the
    output tokens.


Chat API (``POST /v1/chat/completions``)
----------------------------------------

Parameters without descriptions inherit their behavior and functionality from the corresponding parameters in `OpenAI Chat API <https://platform.openai.com/docs/api-reference/chat>`_.

.. list-table::
   :align: center
   :widths: 100 100 100 200
   :header-rows: 1

   * - Name
     - Type
     - Default
     - Description
   * - model
     - string
     -
     - Required by the client, but the value is ignored on the server.
   * - messages
     - array
     -
     -
   * - stream
     - boolean
     - false
     -
   * - stream_options
     - object
     - null
     -
   * - n
     - integer
     - 1
     - Currently limited to 1.
   * - temperature
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_p
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - best_of
     - integer
     - 1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - use_beam_search
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_k
     - integer
     - -1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_p
     - float
     - 0.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - length_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - early_stopping
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_tokens
     - integer
     - 0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - max_tokens
     - integer
     - null
     - Legacy parameter superseded by ``max_completion_tokens``
   * - max_completion_tokens
     - integer
     - null
     - If null, the server will use the maximum possible length considering the prompt.
       The sum of this value and the prompt length must not exceed the model's maximum context length.
   * - tools
     - array
     - null
     -
   * - tool_choice
     - string or object
     - null
     - Supports named function calling, ``"none"``, and ``"auto"``.
       ``"auto"`` is available only when ``--enable-auto-tool-choice`` is set.
   * - logprobs (experimental)
     - boolean
     - false
     -
   * - top_logprobs (experimental)
     - integer
     - null
     -


Completions API (``POST /v1/completions``)
------------------------------------------

Parameters without descriptions inherit their behavior and functionality from the corresponding parameters in `OpenAI Completions API <https://platform.openai.com/docs/api-reference/completions>`_.

.. list-table::
   :align: center
   :widths: 100 100 100 200
   :header-rows: 1

   * - Name
     - Type
     - Default
     - Description
   * - model
     - string
     - required
     - Required by the client, but the value is ignored on the server.
   * - prompt
     - string or array
     - required
     -
   * - stream
     - boolean
     - false
     -
   * - stream_options
     - object
     - null
     -
   * - n
     - integer
     - 1
     - Currently limited to 1.
   * - best_of
     - integer
     - 1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - temperature
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_p
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - use_beam_search
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_k
     - integer
     - -1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_p
     - float
     - 0.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - length_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - early_stopping
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_tokens
     - integer
     - 0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - max_tokens
     - integer
     - 16
     -
   * - logprobs (experimental)
     - integer
     - null
     - See :ref:`Sampling Params <SamplingParams>`.

Additional API Endpoints
=========================

In addition to the Chat and Completions APIs, the Furiosa-LLM server supports the following endpoints.

.. _ModelsEndpoint:

Models Endpoint
---------------

The Models API enables you to retrieve information about available models through endpoints that are compatible with OpenAI's `Models API <https://platform.openai.com/docs/api-reference/models>`_. The following endpoints are supported:

* ``GET /v1/models``
* ``GET /v1/models/{model_id}``

You can access these endpoints using the OpenAI client's ``models.list()`` and ``models.retrieve()`` methods.

The response includes the standard `model object <https://platform.openai.com/docs/api-reference/models/object>`_ as defined by OpenAI, along with the following Furiosa-LLM-specific extensions:

* ``artifact_id``: Unique identifier for the model artifact.
* ``max_prompt_len``: Maximum allowed length of input prompts.
* ``max_context_len``: Maximum allowed length of the total context window.
* ``runtime_config``: Model runtime configuration parameters, including bucket specifications.


.. _VersionEndpoint:

Version Endpoint
----------------

``GET /version``

Exposes version information for the Furiosa SDK components.


.. _MetricsEndpoint:

Metrics Endpoint
----------------

``GET /metrics``

Exposes Prometheus-compatible metrics for monitoring server performance and health.

This endpoint is available when the server is launched with the ``--enable-metrics`` flag.
See :ref:`MonitoringOpenAICompatibleServer` for detailed information about available metrics and their usage.


.. _MonitoringOpenAICompatibleServer:

Monitoring the OpenAI-Compatible Server
=======================================

Furiosa-LLM exposes a Prometheus-compatible metrics endpoint at /metrics,
which provides various metrics compatible with vLLM. These metrics can be
used to monitor LLM serving workloads and the system health.
The metrics endpoint can be enabled with ``--enable-metrics`` option.

The following table shows Furiosa-LLM-specific collectors and metrics:

.. list-table::
   :align: center
   :widths: 100 100 100 200
   :header-rows: 1

   * - Metric
     - Type
     - Metric Labels
     - Description
   * - ``furiosa_llm:num_requests_running``
     - Gauge
     - ``model_name``
     - Number of requests currently running on RNGD.
   * - ``furiosa_llm:num_requests_waiting``
     - Gauge
     - ``model_name``
     - Number of requests waiting to be processed.
   * - ``furiosa_llm:request_received_total``
     - Counter
     - ``model_name``
     - Number of received requests in total.
   * - ``furiosa_llm:request_success_total``
     - Counter
     - ``model_name``
     -  Number of successfully processed requests in total.
   * - ``furiosa_llm:request_failure_total``
     - Counter
     - ``model_name``
     - Number of request process failures in total.
   * - ``furiosa_llm:prompt_tokens_total``
     - Counter
     - ``model_name``
     - Total number of prefill tokens processed.
   * - ``furiosa_llm:generation_tokens_total``
     - Counter
     - ``model_name``
     - Total number of generation tokens processed.
   * - ``furiosa_llm:time_to_first_token_seconds``
     - Histogram
     - ``model_name``
     - Time to first token (TTFT) in seconds.
   * - ``furiosa_llm:time_per_output_token_seconds``
     - Histogram
     - ``model_name``
     - Time per output token (TPOT) in seconds.
   * - ``furiosa_llm:e2e_request_latency_seconds``
     - Histogram
     - ``model_name``
     - End-to-end request latency in seconds.
   * - ``furiosa_llm:request_prompt_tokens``
     - Histogram
     - ``model_name``
     - Number of prefilled tokens processed per request.
   * - ``furiosa_llm:request_generation_tokens``
     - Histogram
     - ``model_name``
     - Number of generation tokens processed per request.
   * - ``furiosa_llm:request_params_max_tokens``
     - Histogram
     - ``model_name``
     - `max_token` request parameter received per request.


Launching the OpenAI-Compatible Server Container
================================================

FuriosaAI offers a containerized server that can be used for faster deployment.
Here is an example that launches the Furiosa-LLM server in a Docker container
(replace ``$HF_TOKEN`` with your Hugging Face Hub token):

.. code-block::

    docker run -it --rm --privileged \
        --env HF_TOKEN=$HF_TOKEN \
        -v $HOME/.cache/huggingface:/root/.cache/huggingface \
        -p 8000:8000 \
        furiosaai/furiosa-llm:latest \
        serve furiosa-ai/Llama-3.1-8B-Instruct-FP8 --devices "npu:0"
