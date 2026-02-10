.. _OpenAIServer:

****************************************************
OpenAI-Compatible Server
****************************************************

Furiosa-LLM offers an OpenAI-compatible server that hosts a single model
and provides OpenAI-compatible chat, completions and embedding APIs along
with some additional pooling API support.

To launch the server, use the ``furiosa-llm serve`` command with the model
artifact path, as follows:

.. code-block::

    furiosa-llm serve [ARTIFACT_PATH]

.. warning::

  This document is based on Furiosa SDK |release|.
  The features and APIs described herein are subject to change in the future.

Supported APIs
==============

We currently support the following APIs:

* Completions API (``/v1/completions``)

  * Applicable to text generation models.

* Chat Completions API (``/v1/chat/completions``)

  * Applicable to text generation models with a :ref:`chat template <ChatTemplates>`.

* Embeddings API (``/v1/embeddings``)

  * Applicable to embedding models.

In addition, we have the following vLLM-compatible APIs for pooling models:

* Score API (``/score``, ``/v1/score``)

  * Currently supported only for Qwen3-Rerank models.

* Rerank API (``/rerank``, ``/v1/rerank``, ``/v2/rerank``)

  * Currently supported only for Qwen3-Rerank models.

Prerequisites
=============
To use the OpenAI-Compatible server, you need the following:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa-LLM <InstallingFuriosaLLM>`
* A :ref:`Hugging Face access token <AuthorizingHuggingFaceHub>`
* A model artifact
* A chat template for chat applications (optional)

Using the OpenAI API
====================
Once the server is running, you can interact with it using an HTTP client,
as shown in the following example:

.. literalinclude:: ../../../examples/openai_curl.sh
  :language: sh

You can also use the OpenAI client to interact with the server.
To use the OpenAI client, you need to install the ``openai`` package first:

.. code-block:: sh

    pip install openai

The following is an example using OpenAI client to call ``chat.completions`` API
with streaming mode:

.. literalinclude:: ../../../examples/online_chat_complete_async_client.py
  :language: python

.. _ChatTemplates:

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
Tool calling (also known as function calling) enables models to interact with external tools and APIs.
Furiosa-LLM supports tool calling for models trained with this capability.

To start the server with tool calling enabled, use the ``--tool-call-parser`` option
to specify the appropriate parser for your model. For example, to enable tool calling
for EXAONE-4.0 models:

.. code-block:: bash

    furiosa-llm serve furiosa-ai/EXAONE-4.0-32B-FP8 --enable-auto-tool-choice --tool-call-parser hermes

To use tool calling, specify the ``tools`` and ``tool_choice``
parameters. Here's an example:

.. literalinclude:: ../../../examples/online_chat_complete_toolcalling.py
  :language: python

The expected output is as follows.

.. code-block:: text

    Function called: get_weather
    Arguments: {"location": "San Francisco, CA", "unit": "fahrenheit"}
    Result: Getting the weather for San Francisco, CA in fahrenheit...

For detailed information on tool calling parsers, tool choice options, and additional examples, see :ref:`ToolCalling`.

.. _Reasoning:

Reasoning Support
=================

Furiosa-LLM provides support for models with reasoning capabilities such as Deepseek R1.
These models follow a structured approach by first conducting reasoning steps and then providing a final answer.

The reasoning process follows this sequence:

1. The model-specific start-of-reasoning token is appended to the input prompt through the chat template.
2. The model generates its reasoning.
3. Once reasoning is done, the model outputs an end-of-reasoning token followed by the final answer.

Since start-of-reasoning and end-of-reasoning tokens differ across models, we support different reasoning parsers for different models.
You can check the help text's ``--reasoning-parser`` description to see which reasoning parsers are supported.

To launch a server with reasoning capabilities for Qwen3 series, run the following example command:

.. code-block:: bash

    furiosa-llm serve furiosa-ai/Qwen3-32B-FP8 --reasoning-parser qwen3

You can access the reasoning content through these response fields:

* ``response.choices[].message.reasoning_content``
* ``response.choices[].delta.reasoning_content``

Here's an example that demonstrates how to access the reasoning content:

.. literalinclude:: ../../../examples/online_chat_complete_reasoning.py
  :language: python

.. note::

    The ``reasoning_content`` field is a Furiosa-LLM-specific extension and is not part of the standard OpenAI API.
    This field will appear only in responses that contain reasoning content, and
    attempting to access this field in responses without reasoning content will raise an ``AttributeError``.

API Reference
=============

.. warning::

    Please note that using ``use_beam_search`` together with ``stream`` is not
    allowed because beam search requires the whole sequence to produce the
    output tokens.

.. _ChatCompletionsAPIReference:

Chat API (``/v1/chat/completions``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
   * - repetition_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - stop_token_ids
     - array[integer]
     - []
     - See :ref:`Sampling Params <SamplingParams>`.
   * - ignore_eos
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - early_stopping
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - skip_special_tokens
     - boolean
     - true
     - See :ref:`Sampling Params <SamplingParams>`.
   * - return_token_ids
     - boolean
     - false
     - When true, includes token IDs in the response (``prompt_token_ids`` and ``token_ids`` fields).
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
     -
   * - functions
     - array
     - null
     - Legacy parameter superseded by ``tools``.
   * - function_call
     - string or object
     - null
     - Legacy parameter superseded by ``tool_choice``.
   * - logprobs (experimental)
     - boolean
     - false
     -
   * - top_logprobs (experimental)
     - integer
     - null
     -

.. _CompletionsAPIReference:

Completions API (``/v1/completions``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
   * - repetition_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - stop_token_ids
     - array[integer]
     - []
     - See :ref:`Sampling Params <SamplingParams>`.
   * - ignore_eos
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - early_stopping
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - skip_special_tokens
     - boolean
     - true
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_tokens
     - integer
     - 0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - max_tokens
     - integer
     - 16
     -
   * - return_token_ids
     - boolean
     - false
     - When true, includes token IDs in the response (``prompt_token_ids`` and ``token_ids`` fields).
   * - logprobs (experimental)
     - integer
     - null
     - See :ref:`Sampling Params <SamplingParams>`.

.. _EmbeddingsAPI:

Embeddings API (``/v1/embeddings``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Parameters without descriptions inherit their behavior and functionality from the corresponding parameters in `OpenAI Embeddings API <https://platform.openai.com/docs/api-reference/embeddings/create>`_.

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
   * - input
     - string or array
     - required
     -
   * - truncate_prompt_tokens
     - integer
     - null
     - See :ref:`Pooling Params <PoolingParams>`.
   * - normalize
     - boolean
     - true
     - See :ref:`Pooling Params <PoolingParams>`.

.. _ScoreAPI:

Score API (``/score``, ``/v1/score``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This API provides text pair scoring functionality, calculating similarity scores between pairs of texts.
This is an extension to the standard OpenAI API specification, originally introduced by vLLM.
For details on the API specification, refer to the `vLLM's Score API documentation <https://docs.vllm.ai/en/latest/serving/openai_compatible_server/#score-api>`_.

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
   * - text_1
     - string or array
     - required
     - The text to be scored as the first input.
   * - text_2
     - string or array
     - required
     - The second input text to be scored.
       If ``text_1`` is a string, ``text_2`` can be either a string or an array,
       allowing 1:N scoring relationships.
       If ``text_1`` is an array, ``text_2`` must be an array of the same length,
       and scores are calculated element-wise.
   * - truncate_prompt_tokens
     - integer
     - null
     - See :ref:`Pooling Params <PoolingParams>`.

.. _RerankAPI:

Rerank API (``/rerank``, ``/v1/rerank``, ``/v2/rerank``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This API provides document reranking functionality, ordering documents by relevance to a given query.
This is an extension to the standard OpenAI API specification, originally introduced by vLLM.
For details on the API specification, refer to the `vLLM's Rerank API documentation <https://docs.vllm.ai/en/latest/serving/openai_compatible_server/#re-rank-api>`_.

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
   * - query
     - string
     - required
     - The query text to rank documents against.
   * - documents
     - array
     - required
     - The list of documents to be reranked.
       Documents are ranked by their relevance to the query,
       with the most relevant documents appearing first in the response.
   * - top_n
     - integer
     -
     - The number of top-ranked documents to return.
       If not specified, all documents are returned in ranked order.
   * - truncate_prompt_tokens
     - integer
     - null
     - See :ref:`Pooling Params <PoolingParams>`.


Additional Endpoints
====================
In addition to the above APIs, the Furiosa-LLM server supports the following endpoints.

.. _ModelsEndpoint:

Models Endpoint
~~~~~~~~~~~~~~~

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
~~~~~~~~~~~~~~~~

``GET /version``

Exposes version information for the Furiosa SDK components.

.. _MetricsEndpoint:

Metrics Endpoint
~~~~~~~~~~~~~~~~

``GET /metrics``

Exposes Prometheus-compatible metrics for monitoring server performance and health.

See :ref:`MonitoringOpenAICompatibleServer` for detailed information about available metrics and their usage.

.. _MonitoringOpenAICompatibleServer:

Monitoring the OpenAI-Compatible Server
=======================================

Furiosa-LLM exposes a Prometheus-compatible metrics endpoint at /metrics,
which provides various metrics compatible with vLLM. These metrics can be
used to monitor LLM serving workloads and the system health.

The following table shows Furiosa-LLM-specific collectors and metrics:

.. list-table::
   :align: center
   :widths: 100 100 100 200
   :header-rows: 1

   * - Metric
     - Type
     - Metric Labels
     - Description
   * - ``furiosa_llm_num_requests_running``
     - Gauge
     - ``model_name``, ``leader_device``
     - Number of requests currently running on RNGD.
   * - ``furiosa_llm_num_requests_waiting``
     - Gauge
     - ``model_name``, ``leader_device``
     - Number of requests waiting to be processed.
   * - ``furiosa_llm_kv_cache_usage_percent``
     - Gauge
     - ``model_name``, ``leader_device``
     - KV-cache usage. 1 means 100 percent usage.
   * - ``furiosa_llm_prefix_cache_hits``
     - Counter
     - ``model_name``, ``leader_device``
     - Prefix cache hits, in terms of number of cached tokens.
   * - ``furiosa_llm_prefix_cache_queries``
     - Counter
     - ``model_name``, ``leader_device``
     - Prefix cache queries, in terms of number of queried tokens.
   * - ``furiosa_llm_prompt_tokens_total``
     - Counter
     - ``model_name``, ``leader_device``
     - Number of prefill tokens processed.
   * - ``furiosa_llm_generation_tokens_total``
     - Counter
     - ``model_name``, ``leader_device``
     - Number of generation tokens processed.
   * - ``furiosa_llm_request_success_total``
     - Counter
     - ``model_name``, ``leader_device``, ``finished_reason``
     - Count of successfully processed requests.
   * - ``furiosa_llm_request_prompt_tokens``
     - Histogram
     - ``model_name``, ``leader_device``
     - Number of prefill tokens processed.
   * - ``furiosa_llm_request_generation_tokens``
     - Histogram
     - ``model_name``, ``leader_device``
     - Number of generation tokens processed.
   * - ``furiosa_llm_request_params_n``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of the n request parameter.
   * - ``furiosa_llm_request_params_max_tokens``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of the max_tokens request parameter.
   * - ``furiosa_llm_time_to_first_token_seconds``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of time to first token in seconds.
   * - ``furiosa_llm_inter_token_latency_seconds``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of inter-token latency in seconds.
   * - ``furiosa_llm_e2e_request_latency_seconds``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of end to end request latency in seconds.
   * - (Experimental) ``furiosa_llm_wire_hit_rate``
     - Gauge
     - ``model_name``, ``leader_device``
     - Wire pipeline hit rate.
   * - (Experimental) ``furiosa_llm_jit_wire_compilation_time_seconds``
     - Histogram
     - ``model_name``, ``leader_device``
     - Histogram of time spent on JIT wire compilations in seconds.

Launching the OpenAI-Compatible Server Container
================================================

Furiosa-LLM offers a containerized server that can be used for faster deployment.
Here is an example that launches the Furiosa-LLM server in a Docker container
(replace ``$HF_TOKEN`` with your Hugging Face Hub token):

.. code-block::

    docker pull furiosaai/furiosa-llm:latest

    docker run -it --rm \
      --device /dev/rngd:/dev/rngd \
      --security-opt seccomp=unconfined \
      --env HF_TOKEN=$HF_TOKEN \
      -v $HOME/.cache/huggingface:/root/.cache/huggingface \
      -p 8000:8000 \
      furiosaai/furiosa-llm:latest \
      serve furiosa-ai/Qwen2.5-0.5B-Instruct

You can also specify additional options for the server and replace
``-v $HOME/.cache/huggingface:/root/.cache/huggingface`` with the path to your
Hugging Face cache directory.
