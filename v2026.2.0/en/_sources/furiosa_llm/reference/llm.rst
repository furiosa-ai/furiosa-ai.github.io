.. _LLMClass:

**********************************
LLM class
**********************************

.. autoclass:: furiosa_llm.LLM
    :members:
    :show-inheritance:

Key Methods
===========

generate()
----------

Generate text completions for the given prompts using sampling parameters.
This is the primary method for text generation tasks.

See the :ref:`chat example <FuriosaLLMExamplesChat>` for usage.

embed()
-------

Generate embedding vectors for the given prompts. This method is only applicable to embedding models.

**Parameters:**

* ``prompts`` (PromptType | Sequence[PromptType]): The prompts to encode. Can be a single prompt or a sequence for batch processing.
* ``pooling_params`` (PoolingParams | Sequence[PoolingParams] | None): The pooling parameters. If None, default parameters are used.

**Returns:**

* ``List[EmbeddingRequestOutput]``: A list of embedding outputs containing the embedding vectors in the same order as the input prompts.

**Example:**

.. code-block:: python

    from furiosa_llm import LLM, PoolingParams

    llm = LLM(artifact_path="path/to/embedding/model")
    
    # Single embedding
    outputs = llm.embed("Hello, world!")
    embedding = outputs[0].outputs.embedding
    
    # Batch embedding with normalization disabled
    params = PoolingParams(normalize=False)
    outputs = llm.embed(["First text", "Second text"], pooling_params=params)

See the :ref:`embedding example <FuriosaLLMExamplesEmbed>` for more details.

score()
-------

Generate similarity scores for text pairs. This method is only supported for binary classification models,
including Qwen3-Reranker models or models converted using ``as_binary_seq_cls_model``.

**Parameters:**

* ``data_1`` (PromptType | Sequence[PromptType]): The first input text(s). Can be a single prompt or a list.
* ``data_2`` (PromptType | Sequence[PromptType]): The second input text(s) to pair with the first.
* ``truncate_prompt_tokens`` (int | None): Maximum number of tokens to truncate the prompt to. If None, no truncation is applied.
* ``pooling_params`` (PoolingParams | None): The pooling parameters. If None, default parameters are used.
* ``chat_template`` (str | None): Custom chat template for scoring. If None, the model's default template is used.

**Input Patterns:**

* **1-to-1**: Single text paired with single text
* **1-to-N**: Single text paired with multiple texts (data_1 is replicated N times)
* **N-to-N**: Multiple texts paired element-wise (both lists must have the same length)

**Returns:**

* ``List[ScoringRequestOutput]``: A list of scoring outputs containing similarity scores in the same order as the input pairs.

**Example:**

.. code-block:: python

    from furiosa_llm import LLM, PoolingParams

    llm = LLM(artifact_path="path/to/reranker/model")
    
    # 1-to-N scoring: one query against multiple documents
    query = "What is machine learning?"
    documents = [
        "Machine learning is a subset of AI",
        "Python is a programming language",
        "Deep learning uses neural networks"
    ]
    
    outputs = llm.score(query, documents)
    for i, output in enumerate(outputs):
        print(f"Document {i}: score = {output.outputs.score}")

See the :ref:`score example <FuriosaLLMExamplesScore>` for more details.
