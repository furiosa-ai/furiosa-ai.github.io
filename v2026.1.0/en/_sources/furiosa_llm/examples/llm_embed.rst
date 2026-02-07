.. _FuriosaLLMExamplesEmbed:

****************************************************
Embedding
****************************************************

This example demonstrates how to generate embeddings using the ``LLM.embed()`` method.
Embeddings are dense vector representations of text that capture semantic meaning,
useful for similarity search, clustering, and retrieval applications.

The following example demonstrates basic usage, batch embedding, normalization, and truncation:

.. literalinclude:: ../../../../examples/offline_embed.py
   :language: python
   :caption: Example of using LLM.embed() for embedding generation


Server API Example
==================

You can also generate embeddings through the OpenAI-compatible server:

.. literalinclude:: ../../../../examples/online_embed.py
   :language: python
   :caption: Example of using OpenAI-compatible API for embedding generation

See :ref:`Embeddings API Reference <EmbeddingsAPI>` for complete server API documentation.
