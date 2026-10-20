.. _ToolCalling:

****************************************************
Tool Calling
****************************************************

Tool calling (also known as function calling) enables models to interact with external tools and APIs.
Furiosa-LLM supports tool calling for models trained with this capability.

Tool Calling Parsers
====================

The system converts model outputs into the OpenAI response format through a
designated parser implementation. Tool calling parsers are model-dependent,
as different models use different formats for tool calls.

Currently, Furiosa-LLM supports the following tool calling parsers:

* ``hermes``: For models using the Hermes tool calling format (e.g., EXAONE-4.0, Qwen3 series).
  Note that Qwen3-Coder is an exception within the Qwen3 series; use ``qwen3_xml`` for it.
* ``llama3_json``: For Llama series models (e.g., Llama 3.1, Llama 3.2)
* ``openai``: For models using the OpenAI tool calling format (e.g., gpt-oss-20b, gpt-oss-120b)
* ``gemma4``: For Gemma 4 models
* ``qwen3_xml`` (alias ``qwen3_coder``): For Qwen3-Coder models, whose chat template asks for an
  XML call format (``<tool_call><function=...><parameter=...>``) instead of the Hermes JSON.

For automatic tool calling, start the server with ``--enable-auto-tool-choice``
and select the appropriate parser using ``--tool-call-parser``.

Tool Choice Options
===================

The ``tool_choice`` parameter controls how the model selects tools to call. Furiosa-LLM supports the following options:

* ``auto`` (default when tools are supplied): The model decides whether to call a tool or respond directly based on the conversation context.
* ``none`` (default when no tools are supplied): Disables tool calling for the request.
* ``required``: Forces the model to call at least one tool. The model cannot respond without making a tool call.
* ``{"type": "function", "function": {"name": "<function_name>"}}``: Forces the model to call a specific named function.

These options apply to both streaming and non-streaming Chat Completions
requests for models that support tool calling.

Supply a non-empty ``tools`` list for ``required`` or a named function choice.
For these choices, the server constrains generation to the tool's JSON schema
and returns the result as ``tool_calls``.

With ``tool_choice="none"``, tool definitions are omitted from the prompt and
tool calls are suppressed in the response.

For more details on the tool calling specification, refer to the `OpenAI Chat API documentation <https://platform.openai.com/docs/api-reference/chat/create#chat-create-tool_choice>`_.

Online Example with Named Function Calling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. literalinclude:: ../../../examples/online_chat_complete_toolcalling_function.py
   :language: python
   :caption: Online Example with Named Function Calling
