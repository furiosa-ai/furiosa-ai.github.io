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

* ``hermes``: For models using the Hermes tool calling format (e.g., EXAONE-4.0, Qwen3 series)
* ``llama``: For Llama series models (e.g., Llama 3.1, Llama 3.2)
* ``openai``: For models using the OpenAI tool calling format (e.g., gpt-oss-20b, gpt-oss-120b)

When starting the server, specify the appropriate parser using the ``--tool-call-parser`` option.

Tool Choice Options
===================

The ``tool_choice`` parameter controls how the model selects tools to call. Furiosa-LLM supports the following options:

* ``auto`` (default): The model decides whether to call a tool or respond directly based on the conversation context.
* ``required``: Forces the model to call at least one tool. The model cannot respond without making a tool call.
* ``{"type": "function", "function": {"name": "<function_name>"}}``: Forces the model to call a specific named function.

For more details on the tool calling specification, refer to the `OpenAI Chat API documentation <https://platform.openai.com/docs/api-reference/chat/create#chat-create-tool_choice>`_.


Offline Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. literalinclude:: ../../../examples/offline_toolcalling.py
   :language: python
   :caption: Offline Example of Tool Calling


Online Example with Named Function Calling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. literalinclude:: ../../../examples/online_chat_complete_toolcalling_function.py
   :language: python
   :caption: Online Example with Named Function Calling
