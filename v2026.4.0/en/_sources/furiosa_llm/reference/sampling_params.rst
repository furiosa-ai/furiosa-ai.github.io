.. _SamplingParams:

**********************************
SamplingParams class
**********************************

.. autoclass:: furiosa_llm.SamplingParams
    :members:
    :show-inheritance:

.. _DefaultSamplingParams:

Default Sampling Parameters from ``generation_config.json``
===========================================================

If a model artifact contains a ``generation_config.json`` file, Furiosa-LLM
uses the values in that file as the effective defaults for the fields listed
below. The file is copied verbatim from the source Hugging Face model during
artifact build — Furiosa-LLM does not customize it. If the file is absent,
the plain ``SamplingParams()`` defaults apply.

The following ``generation_config.json`` keys are honored, and are mapped to
``SamplingParams`` fields as shown:

.. list-table::
   :align: center
   :widths: 200 200
   :header-rows: 1

   * - ``generation_config.json``
     - ``SamplingParams``
   * - ``repetition_penalty``
     - ``repetition_penalty``
   * - ``temperature``
     - ``temperature``
   * - ``top_k``
     - ``top_k``
   * - ``top_p``
     - ``top_p``
   * - ``min_p``
     - ``min_p``
   * - ``max_new_tokens``
     - ``max_tokens``
   * - ``stop_strings``
     - ``stop``

Offline API (``LLM.generate`` / ``LLM.chat`` / ``LLM.stream_generate``)
-----------------------------------------------------------------------

The offline API uses a **binary decision** based on whether the caller
passes ``sampling_params``:

* ``sampling_params=None`` (the default) — the model's
  ``generation_config.json`` defaults are applied. See
  :meth:`furiosa_llm.LLM.get_default_sampling_params`.
* ``sampling_params=SamplingParams(...)`` — the user's object is used
  as-is. No per-field merge with the model's ``generation_config.json``
  is performed.

OpenAI-compatible server
------------------------

The Chat Completions, Completions, and Responses endpoints resolve each
sampling-related field in three tiers:

1. The value specified in the request body, if set.
2. The value from the model's ``generation_config.json``, if present.
3. The API default shown in the endpoint's parameter table
   (see :ref:`ChatCompletionsAPIReference`, :ref:`CompletionsAPIReference`,
   and :ref:`ResponsesAPIReference`).

Examples
========

This section provides examples of how to use the token generation methods available in the SDK.

1. Basic Greedy Search
----------------------

.. code-block:: python

    SamplingParams(min_tokens=10, max_tokens=100)

The Basic Greedy Search method generates a sequence of tokens, ensuring that at least ``min_tokens`` and up to ``max_tokens`` are produced.

- **Parameters:**

  - ``min_tokens``: Minimum number of tokens to generate.
  - ``max_tokens``: Maximum number of tokens to generate.

- **Behavior:**

  - Generation may terminate before reaching ``max_tokens`` if an End Of Sequence (EOS) token is generated.
  - The EOS token will not be generated before reaching the specified ``min_tokens``.

--------

2. Random Sampling with ``top_p`` / ``top_k`` Parameters
-----------------------------------------------------------------------

.. code-block:: python

    SamplingParams(min_tokens=10, max_tokens=100, top_p=0.3, top_k=100)

This method uses random sampling techniques for token generation, allowing for diverse outputs.

- **Parameters:**

  - ``min_tokens``: Minimum number of tokens to generate.
  - ``max_tokens``: Maximum number of tokens to generate.
  - ``top_p``: Cumulative probability for nucleus sampling.
  - ``top_k``: Number of highest probability tokens to consider.

- **Behavior:**

  - Each generation may yield different results, even with the same input text and parameters, enhancing variability.
  - Generation may terminate before reaching ``max_tokens`` if an End Of Sequence (EOS) token is generated.
  - The EOS token will not be generated before reaching the specified ``min_tokens``.

--------

3. Stop Strings
---------------

Pass a list of non-empty strings to ``stop`` to end generation when the first
string is matched. By default, the matched string is removed from the returned
text. A stop-string match sets ``finish_reason`` to ``"stop"`` and records the
matched string in ``stop_reason``:

.. code-block:: python

    params = SamplingParams(stop=["<END>"])
    output = llm.generate("Write a short answer.", params)[0].outputs[0]
    # If the model generates "Answer<END>", output.text is "Answer".
    # output.stop_reason is "<END>".

Set ``include_stop_str_in_output=True`` to retain the matched string. Text
after the match within the completing token is still removed:

.. code-block:: python

    params = SamplingParams(
        stop=["<END>"],
        include_stop_str_in_output=True,
    )
    output = llm.generate("Write a short answer.", params)[0].outputs[0]
    # If the model generates "Answer<END>", output.text is "Answer<END>".

For streamed output with the default exclusion behavior, Furiosa-LLM may delay
emitting a trailing window up to the longest configured stop string so a
partial match does not leak into the output. Retained-stop mode does not need
that delay.
