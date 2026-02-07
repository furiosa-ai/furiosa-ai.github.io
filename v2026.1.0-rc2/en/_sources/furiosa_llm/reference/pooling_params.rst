.. _PoolingParams:

**********************************
PoolingParams class
**********************************

.. autoclass:: furiosa_llm.PoolingParams
    :members:
    :show-inheritance:

Parameters
==========

task
----

**Type:** ``PoolingTask`` (Literal["embed", "score"])

Specifies the pooling task type:

* ``"embed"``: For embedding generation tasks. The model outputs dense vector representations.
* ``"score"``: For similarity scoring tasks. The model outputs scalar similarity scores.

This parameter is usually inferred from the model's metadata, but can be explicitly set when needed.

normalize
---------

**Type:** ``bool``

**Default:** ``True``

Whether to normalize the embedding outputs using L2 normalization. Only applicable for embedding tasks (``task="embed"``).

When ``True``, the embedding vectors are normalized to unit length, which is useful for:

* Cosine similarity computations
* Reducing the impact of vector magnitude differences
* Standardizing embeddings for downstream tasks

**Example:**

.. code-block:: python

    from furiosa_llm import LLM, PoolingParams

    llm = LLM(artifact_path="path/to/embedding/model")
    
    # With normalization (default)
    params_normalized = PoolingParams(normalize=True)
    outputs = llm.embed("Hello, world!", pooling_params=params_normalized)
    # Output vectors have unit length (L2 norm = 1.0)
    
    # Without normalization
    params_raw = PoolingParams(normalize=False)
    outputs = llm.embed("Hello, world!", pooling_params=params_raw)
    # Output vectors preserve original magnitudes

truncate_prompt_tokens
----------------------

**Type:** ``int | None``

**Default:** ``None``

The maximum number of tokens to truncate the input prompt to. If the input exceeds this length, it will be truncated to fit within the limit.

When ``None``, no truncation is applied, and the input is processed up to the model's maximum sequence length.

This is particularly useful for:

* Handling variable-length inputs in batch processing
* Ensuring inputs fit within model constraints
* Controlling computational costs for long documents

**Example:**

.. code-block:: python

    from furiosa_llm import PoolingParams

    # Truncate to 512 tokens
    params = PoolingParams(truncate_prompt_tokens=512)

dimensions
----------

**Type:** ``int | None``

**Default:** ``None``

Reduces the dimensionality of the embedding output to the specified number of dimensions.

When ``None``, the full embedding dimension from the model is returned.

This parameter is useful for:

* Reducing storage requirements
* Speeding up downstream similarity computations
* Matching embedding dimensions for compatibility with other systems

**Note:** Not all models support dimension reduction. Check your model's capabilities before using this parameter.

**Example:**

.. code-block:: python

    from furiosa_llm import PoolingParams

    # Reduce to 256 dimensions
    params = PoolingParams(dimensions=256)
