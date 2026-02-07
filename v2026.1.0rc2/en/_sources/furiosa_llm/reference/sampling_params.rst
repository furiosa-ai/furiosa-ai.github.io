.. _SamplingParams:

**********************************
SamplingParams class
**********************************

.. autoclass:: furiosa_llm.SamplingParams
    :members:
    :show-inheritance:

Key Parameters
==============

This section describes important parameters for controlling text generation.

repetition_penalty
------------------

**Type:** ``float``

**Default:** ``1.0``

**Range:** ``(0, 2]``

Penalizes tokens based on whether they appear in the prompt and the generated text so far.

* **Values > 1.0**: Encourage the model to use new tokens, reducing repetition
* **Values < 1.0**: Encourage the model to repeat tokens
* **Value = 1.0**: No penalty applied (default behavior)

This is useful for:

* Reducing repetitive output in creative text generation
* Encouraging more diverse vocabulary usage
* Controlling output style (more varied vs. more consistent)

**Example:**

.. code-block:: python

    from furiosa_llm import SamplingParams

    # Discourage repetition
    params = SamplingParams(repetition_penalty=1.2, max_tokens=100)

stop_token_ids
--------------

**Type:** ``list[int] | None``

**Default:** ``None``

A list of token IDs that will stop generation when encountered. This is in addition to the model's EOS (End of Sequence) token.

The returned output will contain the stop tokens unless they are special tokens and ``skip_special_tokens=True``.

This is useful for:

* Implementing custom stopping conditions
* Stopping at specific delimiters or punctuation
* Creating structured output formats

**Example:**

.. code-block:: python

    from furiosa_llm import LLM, SamplingParams

    llm = LLM(artifact_path="path/to/model")
    
    # Stop generation when encountering specific tokens
    # For example, stop at commas and periods
    stop_ids = llm.tokenizer.encode([',', '.'], add_special_tokens=False)
    params = SamplingParams(stop_token_ids=stop_ids, max_tokens=100)
    
    outputs = llm.generate("Generate a short list:", sampling_params=params)

ignore_eos
----------

**Type:** ``bool``

**Default:** ``False``

When ``True``, the model will ignore the EOS (End of Sequence) token and continue generating tokens until reaching ``max_tokens`` or encountering a ``stop_token_ids`` token.

This is useful for:

* Forcing the model to generate the maximum number of tokens
* Bypassing premature stopping
* Generating fixed-length outputs regardless of natural stopping points

**Example:**

.. code-block:: python

    from furiosa_llm import SamplingParams

    # Generate exactly 100 tokens, ignoring EOS
    params = SamplingParams(ignore_eos=True, max_tokens=100)

skip_special_tokens
-------------------

**Type:** ``bool``

**Default:** ``True``

When ``True``, special tokens (such as ``<|begin_of_text|>``, ``<|end_of_text|>``, ``<pad>``, etc.) are excluded from the decoded output text.

When ``False``, all tokens including special tokens are included in the output.

This is useful for:

* Debugging tokenization and generation issues (set to ``False``)
* Clean user-facing output (set to ``True``, the default)
* Analyzing model behavior with special tokens

**Example:**

.. code-block:: python

    from furiosa_llm import SamplingParams

    # Include special tokens in output for debugging
    params = SamplingParams(skip_special_tokens=False, max_tokens=50)

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

3. Beam Search with ``best_of`` Beams
-------------------------------------

.. code-block:: python

    SamplingParams(min_tokens=10, max_tokens=100, use_beam_search=True, best_of=4)

Beam Search enhances the generation process by exploring multiple sequences simultaneously.

- **Parameters:**

  - ``min_tokens``: Minimum number of tokens to generate.
  - ``max_tokens``: Maximum number of tokens to generate.
  - ``use_beam_search``: Must be set to True to enable beam search.
  - ``best_of``: Number of beams to consider for generating the best output.

- **Behavior:**

  - The generation process explores multiple possible sequences to determine the best output.
  - Generation may terminate before reaching ``max_tokens`` if the number of End Of Sequence (EOS) tokens generated across all beams reaches the ``best_of`` count.
  - The EOS token will not be generated before reaching the specified ``min_tokens``.
