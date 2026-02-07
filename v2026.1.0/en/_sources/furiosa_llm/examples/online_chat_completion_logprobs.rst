.. _FuriosaLLMExamplesOpenAILogprobs:

****************************************************
OpenAI-Compatible API with Logprobs
****************************************************

This example demonstrates how to retrieve log probabilities (logprobs) using the
OpenAI-compatible Chat Completion API. There are two types of logprobs available:

1. **Generated Token Logprobs** (Standard OpenAI): Shows the model's probability
   distribution over tokens during text generation.

2. **Prompt Token Logprobs** (vLLM Extension): Shows the model's probability
   distribution over tokens for each position in the input prompt.


Chat Completion API Example
============================

The following example shows how to use both ``logprobs`` (for generated tokens)
and ``prompt_logprobs`` (for prompt tokens) in a single request.

.. literalinclude:: ../../../../examples/online_chat_completion_logprobs.py
   :language: python
   :caption: Example of using Chat Completion API with logprobs and prompt_logprobs


Example Output
--------------

.. code-block:: text

   Using model: Qwen/Qwen2.5-0.5B

   ================================================================================
   Generated text: The capital of France is Paris.

   ================================================================================

   === Generated Token Logprobs (Standard OpenAI) ===
   These show the model's confidence for each token it generated.

   Position 0: 'The' (logprob=-0.298153)
       'The': logprob=-0.298153 <-- chosen
       'Paris': logprob=-1.551574
       'As': logprob=-4.550769

   Position 1: ' capital' (logprob=-0.003914)
       ' capital': logprob=-0.003914 <-- chosen
       ' current': logprob=-6.505387
       ' Capital': logprob=-8.506357

   Position 2: ' of' (logprob=-0.011788)
       ' of': logprob=-0.011788 <-- chosen
       ' city': logprob=-4.769587
       ' and': logprob=-8.139359

   Position 3: ' France' (logprob=-0.003914)
       ' France': logprob=-0.003914 <-- chosen
       ' the': logprob=-10.007491
       'France': logprob=-11.007912

   Position 4: ' is' (logprob=-0.003914)
       ' is': logprob=-0.003914 <-- chosen
       ',': logprob=-5.880869
       ' (': logprob=-9.756177

   Position 5: ' Paris' (logprob=-0.007843)
       ' Paris': logprob=-0.007843 <-- chosen
       ' Lyon': logprob=-6.015181
       '巴黎': logprob=-6.947220

   ...

   === Prompt Token Logprobs (vLLM Extension) ===
   These show how likely each prompt token was, given the preceding context.

   Position 0: token_id=151644 -> None (first token, no prior context)
   Position 1: token_id=8948
       token_id=  8948, token='system', logprob=-11.958041, rank=8063 <-- actual
       token_id= 72030, token='/API', logprob= -1.378512, rank=1
       token_id= 16731, token='/T', logprob= -3.065493, rank=2
       token_id= 59981, token='/block', logprob= -3.948318, rank=3
   Position 2: token_id=198
       token_id=   198, token='\n', logprob= -1.947865, rank=1 <-- actual
       token_id=   271, token='\n\n', logprob= -2.263327, rank=2
       token_id= 69425, token=' 发', logprob= -2.448469, rank=3
   ...


Key Parameters
==============

Generated Token Logprobs (Standard OpenAI)
------------------------------------------

- ``logprobs`` (bool): When True, returns log probabilities for generated tokens.
- ``top_logprobs`` (int): Number of top alternative tokens to return.

These parameters are part of the standard OpenAI Chat Completion API and show
what alternatives the model considered when generating each token.

Prompt Token Logprobs (vLLM Extension)
--------------------------------------

- ``prompt_logprobs`` (int): Number of top log probabilities to return for each
  prompt token. Pass via ``extra_body``.
- ``return_token_ids`` (bool): When True, returns the token IDs of the prompt
  tokens. Pass via ``extra_body``.

These parameters are vLLM extensions and show how likely each token in your
prompt was, given the preceding context.


Response Structure
==================

Generated Token Logprobs
------------------------

Located in the standard response structure:

.. code-block:: python

   response.choices[0].logprobs.content[i].token       # The chosen token
   response.choices[0].logprobs.content[i].logprob     # Its log probability
   response.choices[0].logprobs.content[i].top_logprobs  # Alternative tokens

Prompt Token Logprobs
---------------------

Located in the vLLM extension fields, accessible directly on the response object:

.. code-block:: python

   response.prompt_logprobs    # List of logprobs per prompt position
   response.prompt_token_ids   # List of actual token IDs in the prompt

Each logprob entry contains:

- ``logprob``: The log probability value
- ``rank``: The rank of this token among all possibilities (1 = most likely)
- ``decoded_token``: The string representation of the token


Understanding the Output
========================

Generated Token Logprobs
------------------------

For generated tokens, a logprob closer to 0 indicates higher confidence:

- ``logprob=-0.003914``: :math:`e^{-0.003914}` ≈ 99.6% probability (very confident)
- ``logprob=-2.0`` ≈ 13.5% probability (less confident)
- ``logprob=-5.0`` ≈ 0.7% probability (unlikely alternative)

Prompt Token Logprobs
---------------------

For prompt tokens, the ``rank`` field shows where the actual token ranked among
all possible tokens. A high rank (e.g., 8063) with a low logprob indicates that
the model found that token "surprising" given the context - this is expected for prompt
tokens since they are user-provided.
