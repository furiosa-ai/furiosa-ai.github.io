.. _ResponsesAPI:

****************************************************
Responses API
****************************************************

Furiosa-LLM implements the `OpenResponses specification <https://www.openresponses.org/specification>`_,
a multi-provider, interoperable LLM interface. The Responses API is available for
text generation models with a chat template. It supports text input, streaming,
multi-turn conversations, tool calling with ``tool_choice`` control
(see :ref:`ToolCallingServer`), and structured output via JSON Schema.

.. note::

   The following features from the OpenResponses specification are **not yet supported**:

   * **Multimodal inputs** — ``input_image``, ``input_audio``, and ``input_file``
     content types are accepted but silently ignored.
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

    furiosa-llm serve [ARTIFACT_PATH] --enable-responses-api-store

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

.. _ResponsesAPIReference:

API Reference
=============

Parameters without descriptions inherit their behavior and functionality from the corresponding parameters in the `OpenResponses specification <https://www.openresponses.org/reference>`_.

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
       Multimodal content types (``input_image``, ``input_audio``, ``input_file``) are not yet supported.
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
