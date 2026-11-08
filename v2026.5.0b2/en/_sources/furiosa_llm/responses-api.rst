.. _ResponsesAPI:

****************************************************
Responses API
****************************************************

Furiosa-LLM implements the `OpenResponses specification <https://www.openresponses.org/specification>`_,
a multi-provider, interoperable LLM interface. The Responses API is available for
text generation models with a chat template. It supports text input, image input
with a compatible vision-language model, streaming,
multi-turn conversations, tool calling with ``tool_choice`` control
(see :ref:`ToolCallingServer`), and structured output via JSON Schema.

.. note::

   The following features from the OpenResponses specification are **not yet supported**:

   * **Audio and file inputs** — ``input_audio`` and ``input_file`` are unsupported.
     Images require ``input_image.image_url``; file-ID-only images are rejected.
   * **Built-in tools** — ``web_search``, ``file_search``, ``code_interpreter``,
     ``computer_use``, and ``mcp`` tools are not supported. Only custom function
     tools are available.
   * **Background processing** — ``background=true`` is accepted but has no effect.
   * **Auto-truncation** — ``truncation="auto"`` is accepted but only ``"disabled"``
     is implemented.

Endpoints
=========

* ``POST /v1/responses`` — Create a response (streaming or non-streaming).
* ``GET /v1/responses/{response_id}`` — Retrieve a previously stored response.
* ``POST /v1/responses/{response_id}/cancel`` — Cancel an in-progress response.

.. _ResponseStore:

Response Store
==============

The response store keeps responses in memory so they can be retrieved later or
referenced by ``previous_response_id`` for multi-turn conversations. The store
is **disabled by default** and must be explicitly enabled with the
``--enable-responses-api-store`` server option:

.. code-block:: bash

    furiosa-llm serve [MODEL_ID_OR_PATH] --enable-responses-api-store

The following server options control the store behavior:

.. list-table::
   :align: center
   :widths: 200 100 200
   :header-rows: 1

   * - Option
     - Default
     - Description
   * - ``--enable-responses-api-store``
     - false
     - Enable the in-memory response store.
   * - ``--responses-api-store-max-entries``
     - 10000
     - Maximum number of responses to keep. Oldest entries are evicted when the
       limit is reached.
   * - ``--responses-api-store-ttl``
     - 3600
     - Time-to-live for stored responses in seconds.

When the store is enabled and ``store=true`` is set in the request (the default),
the server stores the response and its full chat message history, enabling:

* Response retrieval via ``GET /v1/responses/{response_id}``.
* Response cancellation via ``POST /v1/responses/{response_id}/cancel``.
* Multi-turn conversations via ``previous_response_id``.

To skip storage for a specific request, set ``store=false``.

When the store is disabled, ``GET /v1/responses/{response_id}``,
``POST /v1/responses/{response_id}/cancel``, and ``previous_response_id`` are
not available.

.. note::

   Stored responses and their conversation histories are held in memory and are
   lost when the server restarts. Monitor memory consumption on the server if
   you expect a large number of stored responses.

.. _ResponsesMultiTurn:

Multi-Turn Conversations
========================

The Responses API supports two methods for multi-turn conversations:

Using previous_response_id (recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The server automatically prepends the stored conversation history from the
referenced response. This is the simplest approach and requires ``store=true``
on the referenced response.

``previous_response_id`` carries conversation history, but does not inherit
the previous request's ``media_io_kwargs`` or ``mm_processor_kwargs``. Each
request resolves these options against the current server defaults, including
when processing images from the stored history. Resend any desired overrides.

.. literalinclude:: ../../../examples/online_responses_multi_turn.py
   :language: python

Using manual context
~~~~~~~~~~~~~~~~~~~~

Alternatively, you can manually build the conversation context by appending
previous output items to the ``input`` array:

.. code-block:: python

    # Turn 1
    res1 = client.responses.create(model=model, input="My name is Alice.")

    # Turn 2: manually include previous context
    context = [{"role": "user", "content": "My name is Alice."}]
    context += res1.output
    context.append({"role": "user", "content": "What is my name?"})
    res2 = client.responses.create(model=model, input=context)

Examples
========

Basic usage:

.. literalinclude:: ../../../examples/online_responses_basic.py
   :language: python

Streaming:

.. literalinclude:: ../../../examples/online_responses_streaming.py
   :language: python

Image input
~~~~~~~~~~~

Start a compatible vision-language server, for example
``furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct``, then send an
``input_image`` part alongside ``input_text``:

.. code-block:: python

   import os
   from openai import OpenAI

   client = OpenAI(
       base_url=os.getenv("OPENAI_BASE_URL", "http://localhost:8000/v1"),
       api_key=os.getenv("OPENAI_API_KEY", "EMPTY"),
   )
   response = client.responses.create(
       model=client.models.list().data[0].id,
       input=[{
           "role": "user",
           "content": [
               {"type": "input_text", "text": "Describe this image."},
               {
                   "type": "input_image",
                   "image_url": "https://developer.furiosa.ai/latest/en/_images/Furiosa_RNGD_image.png",
                   "detail": "auto",
               },
           ],
       }],
       max_output_tokens=32,
       extra_body={
           "media_io_kwargs": {"image": {"timeout": 10}},
           "mm_processor_kwargs": {"image": {"min_pixels": 1024}},
       },
   )
   print(response.output_text)

``input_image.image_url`` is a URL string. It accepts HTTP(S) URLs, base64
``data:`` URLs, and local ``file://`` URLs permitted by
``--allowed-local-media-path``. Remote URLs obey ``--allowed-media-domains``,
and image counts obey ``--image-limit-per-prompt``. A ``file_id`` without an
``image_url`` is rejected with HTTP 400. See :ref:`VisionLanguageModels` for
input formats and :ref:`MultimodalOptions` for defaults and request overrides.

.. _ResponsesAPIReference:

API Reference
=============

Parameters without descriptions inherit their behavior and functionality from the corresponding parameters in the `OpenResponses specification <https://www.openresponses.org/reference>`__.

.. note::

    For sampling-related fields (``temperature``, ``top_p``, ``top_k``,
    ``max_output_tokens``, ``stop``), each field resolves in this order:

    1. The value specified in the request body, if set.
    2. The value from the model's ``generation_config.json``, if present
       (``max_output_tokens`` is populated from ``max_new_tokens``).
    3. The API default shown in the parameter table below.

    See :ref:`DefaultSamplingParams` for the supported-field mapping and
    offline-API behavior.

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
   * - input
     - string or array
     -
     - Text string, or an array of input items (messages, function call outputs).
       Message content may include ``input_text`` and ``input_image`` parts.
       Images require a compatible vision-language model and an ``image_url``.
       Audio and file inputs are unsupported.
   * - media_io_kwargs
     - object or null
     - null
     - Image-fetch overrides keyed by modality, such as
       ``{"image": {"timeout": 10}}``. See :ref:`MultimodalOptions`.
   * - mm_processor_kwargs
     - object or null
     - null
     - Processor overrides keyed by modality, such as
       ``{"image": {"min_pixels": 1024}}``. Request options override server
       defaults within each modality; see :ref:`MultimodalOptions`.
   * - instructions
     - string
     - null
     - System-level instructions prepended to the conversation.
   * - stream
     - boolean
     - false
     -
   * - store
     - boolean
     - true
     - When true and ``--enable-responses-api-store`` is set, the response is
       stored in-memory for later retrieval via ``previous_response_id`` and
       ``GET /v1/responses/{response_id}``. See :ref:`ResponseStore`.
   * - temperature
     - float
     - 1.0
     -
   * - top_p
     - float
     - 1.0
     -
   * - top_k
     - integer
     - -1
     - Furiosa-LLM extension; not part of the OpenResponses specification.
   * - stop
     - string or array[string]
     - null
     - Stop generation when a non-empty string is matched. A single string is
       normalized to a one-item list. If omitted, ``stop_strings`` from the
       model's ``generation_config.json`` is used when available.
   * - include_stop_str_in_output
     - boolean
     - false
     - When true, retain the matched ``stop`` string in the output; any text
       after the match in the completing token is removed.
   * - skip_special_tokens
     - boolean
     - true
     - Whether tokenizer special tokens are omitted from generated text.
   * - max_output_tokens
     - integer
     - null
     - If null, the server will use the maximum possible length considering the input.
   * - presence_penalty
     - float
     - 0.0
     - Accepted for compatibility but not yet functional.
   * - frequency_penalty
     - float
     - 0.0
     - Accepted for compatibility but not yet functional.
   * - tools
     - array
     - []
     - Function tool definitions. Only custom function tools are supported.
   * - tool_choice
     - string or object
     - "auto"
     - Controls how tools are invoked. Supported values:

       * ``"none"`` — disable tool calling; model output is returned as text.
       * ``"auto"`` — the server detects tool calls in model output using the
         configured tool parser (requires ``--enable-auto-tool-choice``
         and ``--tool-call-parser``).
       * ``"required"`` — model output is parsed as a JSON array of tool
         definitions (``[{"name": ..., "parameters": {...}}]``).
       * ``{"type": "function", "name": "<fn>"}`` — all model output is
         treated as arguments for the named function.
   * - text
     - object
     - null
     - Structured output configuration. Supports ``text.format`` with
       ``type: "json_schema"`` to constrain model output to a JSON Schema.
       Uses the same structured-output engine as the Chat Completions API.
   * - previous_response_id
     - string
     - null
     - ID of a previously stored response to continue the conversation from.
   * - truncation
     - string
     - "disabled"
     - Only ``"disabled"`` is currently supported.
   * - reasoning
     - object
     - null
     - Accepted but not yet functional.
   * - metadata
     - object
     - null
     -
   * - user
     - string
     - null
     -
