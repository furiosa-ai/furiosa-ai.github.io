.. _FuriosaLLMExamplesScore:

****************************************************
Scoring (Similarity Scoring)
****************************************************

This example demonstrates how to use the ``LLM.score()`` method for calculating similarity scores between text pairs.
This is applicable to binary classification models, including Qwen3-Reranker models or models converted using ``as_binary_seq_cls_model``.

Python API Example
==================

The following example demonstrates 1-to-1, 1-to-N, and N-to-N scoring with PoolingParams:

.. literalinclude:: ../../../../examples/offline_score.py
   :language: python
   :caption: Example of using LLM.score() for similarity scoring

Use Cases
=========

The ``LLM.score()`` method is useful for:

* **Document Retrieval**: Finding the most relevant documents for a query
* **Semantic Similarity**: Measuring how similar two pieces of text are
* **Question Answering**: Identifying which document best answers a question
* **Duplicate Detection**: Finding similar or duplicate content
* **Content Recommendation**: Suggesting related articles or documents

For ranking multiple documents by relevance, see :ref:`Rerank API example <FuriosaLLMExamplesRerank>`.

Server API Example
==================

You can also use the scoring functionality through the OpenAI-compatible server:

.. literalinclude:: ../../../../examples/online_score.py
   :language: python
   :caption: Example of using Score API for similarity scoring

See :ref:`Score API Reference <ScoreAPI>` for complete server API documentation.
