.. _OpenAIServer:

****************************************************
OpenAI-Compatible Server
****************************************************

Furiosa-LLM offers an OpenAI-compatible server that hosts a single model
and provides OpenAI-compatible chat, completions, responses, and embedding APIs
along with some additional pooling API support.

To launch the server, pass a Hugging Face model ID or local model directory to
``furiosa-llm serve``. A compatible FXB must be bundled with the model, available in the
local FXB cache, or selected explicitly with ``--fxb``:

.. code-block::

    furiosa-llm serve [MODEL_ID_OR_PATH]

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

* Responses API (``/v1/responses``) — `OpenResponses specification <https://www.openresponses.org/specification>`_

  * Applicable to text generation models with a :ref:`chat template <ChatTemplates>`.
  * See :ref:`ResponsesAPI` for details.

* Embeddings API (``/v1/embeddings``)

  * Applicable to embedding models.

In addition, we have the following vLLM-compatible APIs for pooling models:

* Score API (``/score``, ``/v1/score``)

  * Supported for Qwen3-Reranker and BGE-reranker-v2-m3 models.

* Rerank API (``/rerank``, ``/v1/rerank``, ``/v2/rerank``)

  * Supported for Qwen3-Reranker and BGE-reranker-v2-m3 models.

Prerequisites
=============
To use the OpenAI-Compatible server, you need the following:

* A system with the prerequisites installed (see :ref:`InstallingPrerequisites`)
* An installation of :ref:`Furiosa-LLM <InstallingFuriosaLLM>`
* A :ref:`Hugging Face access token <AuthorizingHuggingFaceHub>` for gated models
* A compatible FXB bundled with the model, available in the local cache, or supplied with
  ``--fxb`` (see :ref:`FXB`)
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

.. _ServeMaxModelLen:

Limiting the Context Length
===========================

By default the server accepts requests up to the smallest of the model's
``max_position_embeddings``, the context length covered by the loaded FXB,
and what the KV cache can hold. ``--max-model-len`` lowers that limit:

.. code-block::

    furiosa-llm serve [MODEL_ID_OR_PATH] --max-model-len 8192

The value caps the total tokens a single request may use — prompt plus
generated tokens. It applies to every request, including those that do not set
``max_tokens``: the implicit generation budget shrinks with the cap as well. The
limit advertised on ``/v1/models`` is whichever of those four bounds is smallest,
so on a memory-tight deployment it can be the KV-derived one even with no cap
set. A request exceeding it is rejected with a context-length error.

The cap may only *lower* the limit. A value above either the model's
``max_position_embeddings`` or the context length covered by the loaded FXB
is rejected at startup, so a typo fails fast instead of being ignored.

Lowering the cap also raises concurrency for requests that omit ``max_tokens``.
Admission is conservative by default: a request is charged ``prompt + max_tokens``
KV blocks — one block per token, less any prefix-cache hit — for its whole
lifetime, and a request that omits ``max_tokens`` is budgeted the entire remaining
context window. Each therefore costs a full context, so roughly
``max_kv_len / max_model_len`` of them run concurrently per data-parallel replica.
The gain stops once another limit binds, such as ``--max-concurrency`` or a cap
already at or above the KV capacity. It frees no NPU memory: weights, buckets and
the KV pool are fixed by the loaded FXB.

.. note::

  ``fxb build`` takes an option of the same name, but it means something different there:
  at build time ``--max-model-len`` shapes which buckets are compiled into the FXB. At
  serving time the bucket set is fixed by the loaded FXB, and this option only caps
  admission.

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

    furiosa-llm serve [MODEL_ID_OR_PATH] --chat-template [CHAT_TEMPLATE_PATH]

.. _ToolCallingServer:

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

* ``response.choices[].message.reasoning``
* ``response.choices[].delta.reasoning``

Here's an example that demonstrates how to access the reasoning content:

.. literalinclude:: ../../../examples/online_chat_complete_reasoning.py
  :language: python

.. note::

    The ``reasoning`` field is a Furiosa-LLM-specific extension and is not part of the standard OpenAI API.
    This field will appear only in responses that contain reasoning content, and
    attempting to access this field in responses without reasoning content will raise an ``AttributeError``.

API Reference
=============

.. note::

    For sampling-related fields (``temperature``, ``top_p``, ``top_k``,
    ``min_p``, ``repetition_penalty``, ``max_tokens``, ``stop``), each field
    resolves in this order:

    1. The value specified in the request body, if set.
    2. The value from the model's ``generation_config.json``, if present.
    3. The API default shown in the parameter table below.

    See :ref:`DefaultSamplingParams` for the full supported-field mapping
    and offline-API behavior.

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
   * - top_k
     - integer
     - -1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_p
     - float
     - 0.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - repetition_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - stop
     - string or array[string]
     - null
     - Stop string or list of non-empty stop strings. See :ref:`Sampling Params <SamplingParams>`.
   * - include_stop_str_in_output
     - boolean
     - false
     - Whether to retain a matched ``stop`` string. See :ref:`Sampling Params <SamplingParams>`.
   * - stop_token_ids
     - array[integer]
     - []
     - See :ref:`Sampling Params <SamplingParams>`.
   * - ignore_eos
     - boolean
     - false
     - See :ref:`Sampling Params <SamplingParams>`.
   * - skip_special_tokens
     - boolean
     - true
     - Whether tokenizer special tokens are omitted from generated text.
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
   * - temperature
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_p
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - top_k
     - integer
     - -1
     - See :ref:`Sampling Params <SamplingParams>`.
   * - min_p
     - float
     - 0.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - repetition_penalty
     - float
     - 1.0
     - See :ref:`Sampling Params <SamplingParams>`.
   * - stop
     - string or array[string]
     - null
     - Stop string or list of non-empty stop strings. See :ref:`Sampling Params <SamplingParams>`.
   * - include_stop_str_in_output
     - boolean
     - false
     - Whether to retain a matched ``stop`` string. See :ref:`Sampling Params <SamplingParams>`.
   * - stop_token_ids
     - array[integer]
     - []
     - See :ref:`Sampling Params <SamplingParams>`.
   * - ignore_eos
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

Stop-string response fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~

For Chat Completions and Completions, each response choice has a
``stop_reason`` field. This is a Furiosa-LLM extension to the OpenAI-compatible
response schema. When a configured stop string ends generation, the terminal
choice—or the terminal choice in a streamed chunk—has ``finish_reason`` set to
``"stop"`` and ``stop_reason`` set to the matched string. ``stop_reason`` is
``null`` for earlier streamed chunks and when generation ends for another
reason.

The raw JSON response always includes the field. OpenAI Python SDK response
models that do not declare this extension expose it through
``choice.model_extra["stop_reason"]``.

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
   * - dimensions
     - integer
     - null
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

* ``artifact_id``: Unique identifier for the loaded FXB.
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

.. _TokenizerAPI:

Tokenizer API
~~~~~~~~~~~~~

The Tokenizer API is a simple wrapper over HuggingFace-style tokenizers.
It is available for all model types and consists of three endpoints:

* ``POST /tokenize`` corresponds to calling ``tokenizer.encode()``.
  It accepts either a ``prompt`` string (completion-style) or ``messages`` array (chat-style, which applies the chat template before tokenizing).
* ``POST /detokenize`` corresponds to calling ``tokenizer.decode()``.
* ``GET /tokenizer_info`` returns metadata about the tokenizer, including its class name and configuration.

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
     - ``model_name``, ``engine``
     - Number of requests currently running on RNGD.
   * - ``furiosa_llm_num_requests_waiting``
     - Gauge
     - ``model_name``, ``engine``
     - Number of requests waiting to be processed.
   * - ``furiosa_llm_kv_cache_usage_percent``
     - Gauge
     - ``model_name``, ``engine``
     - KV-cache usage. 1 means 100 percent usage.
   * - ``furiosa_llm_prefix_cache_hits``
     - Counter
     - ``model_name``, ``engine``
     - Prefix cache hits, in terms of number of cached tokens.
   * - ``furiosa_llm_prefix_cache_queries``
     - Counter
     - ``model_name``, ``engine``
     - Prefix cache queries, in terms of number of queried tokens.
   * - ``furiosa_llm_prompt_tokens_total``
     - Counter
     - ``model_name``, ``engine``
     - Number of prefill tokens processed.
   * - ``furiosa_llm_generation_tokens_total``
     - Counter
     - ``model_name``, ``engine``
     - Number of generation tokens processed.
   * - ``furiosa_llm_request_success_total``
     - Counter
     - ``model_name``, ``engine``, ``finished_reason``
     - Count of successfully processed requests.
   * - ``furiosa_llm_request_prompt_tokens``
     - Histogram
     - ``model_name``, ``engine``
     - Number of prefill tokens processed.
   * - ``furiosa_llm_request_generation_tokens``
     - Histogram
     - ``model_name``, ``engine``
     - Number of generation tokens processed.
   * - ``furiosa_llm_request_params_n``
     - Histogram
     - ``model_name``, ``engine``
     - Histogram of the n request parameter.
   * - ``furiosa_llm_request_params_max_tokens``
     - Histogram
     - ``model_name``, ``engine``
     - Histogram of the max_tokens request parameter.
   * - ``furiosa_llm_time_to_first_token_seconds``
     - Histogram
     - ``model_name``, ``engine``
     - Histogram of time to first token in seconds.
   * - ``furiosa_llm_inter_token_latency_seconds``
     - Histogram
     - ``model_name``, ``engine``
     - Histogram of inter-token latency in seconds.
   * - ``furiosa_llm_e2e_request_latency_seconds``
     - Histogram
     - ``model_name``, ``engine``
     - Histogram of end to end request latency in seconds.
   * - (Experimental) ``furiosa_llm_wire_hit_rate``
     - Gauge
     - ``model_name``, ``engine``
     - Wire pipeline hit rate.
   * - (Experimental) ``furiosa_llm_jit_wire_compilation_time_seconds``
     - Histogram
     - ``model_name``, ``engine``
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
