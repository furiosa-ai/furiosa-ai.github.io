.. _FuriosaLLMExamplesRerank:

****************************************************
Reranking (Document Reranking)
****************************************************

This example demonstrates how to use the rerank functionality to order documents by relevance to a query.
Reranking is particularly useful for retrieval-augmented generation (RAG) and information retrieval applications.

Server API Example
==================

The rerank API is available through the OpenAI-compatible server. Currently, there is no direct Python ``LLM.rerank()`` method;
use the ``LLM.score()`` method and sort results manually, or use the HTTP API.

Basic Reranking
---------------

.. literalinclude:: ../../../../examples/online_score_rerank.py
   :language: python
   :caption: Example of using Rerank API for document reranking

Using top_n Parameter
---------------------

Limit the number of returned results to the top N most relevant documents:

.. literalinclude:: ../../../../examples/online_score_rerank_top_n.py
   :language: python
   :caption: Example of using Rerank API with top_n parameter

Truncating Long Documents
--------------------------

Use ``truncate_prompt_tokens`` to handle long documents that exceed the model's context length:

.. literalinclude:: ../../../../examples/online_score_rerank_truncate.py
   :language: python
   :caption: Example of using Rerank API with truncate_prompt_tokens

Using Python Client
===================

You can also use the OpenAI Python client or any HTTP client:

.. code-block:: python

    from openai import OpenAI

    # Initialize client pointing to Furiosa-LLM server
    client = OpenAI(
        base_url="http://localhost:8000/v1",
        api_key="dummy"  # Not used but required by client
    )

    # Note: OpenAI client doesn't have native rerank support
    # Use requests library or implement custom extension

Use Cases
=========

Reranking is essential for:

* **Retrieval-Augmented Generation (RAG)**
  
  - First-stage retrieval returns many candidates
  - Reranking selects the most relevant documents to include in the LLM context
  - Improves answer quality by providing better context

* **Search Engines**
  
  - Initial search returns broad results
  - Reranking orders them by relevance to user query
  - Enhances user experience with more accurate results

* **Question Answering Systems**
  
  - Multiple knowledge base articles retrieved
  - Reranking identifies which article best answers the question
  - Reduces latency by processing fewer documents

* **Content Recommendation**
  
  - Candidate items filtered by basic criteria
  - Reranking personalizes based on user query or context
  - Delivers more relevant recommendations

Workflow Example: RAG Pipeline
===============================

.. code-block:: python

    import requests
    from openai import OpenAI

    # Step 1: Retrieve candidate documents (e.g., from vector database)
    query = "How does attention mechanism work in transformers?"
    candidate_documents = retrieve_from_vector_db(query, top_k=50)  # Get 50 candidates

    # Step 2: Rerank to find most relevant
    rerank_response = requests.post(
        "http://localhost:8000/v1/rerank",
        json={
            "model": "reranker",
            "query": query,
            "documents": candidate_documents,
            "top_n": 5  # Select top 5 for LLM context
        }
    )

    top_documents = [
        result["document"]["text"] 
        for result in rerank_response.json()["results"]
    ]

    # Step 3: Generate answer using reranked documents
    llm_client = OpenAI(base_url="http://localhost:8000/v1", api_key="dummy")
    
    context = "\n\n".join(top_documents)
    completion = llm_client.chat.completions.create(
        model="llama",
        messages=[
            {"role": "system", "content": f"Answer based on this context:\n\n{context}"},
            {"role": "user", "content": query}
        ]
    )

    print(completion.choices[0].message.content)

API Compatibility
=================

Furiosa-LLM's rerank API follows the vLLM rerank API specification and is compatible with:

* Multiple endpoint paths: ``/rerank``, ``/v1/rerank``, ``/v2/rerank``
* JinaAI rerank API format (commonly used in RAG frameworks like RAGFlow)

This ensures compatibility with existing tools and frameworks that support these standards.

For scoring individual pairs without ranking, see :ref:`Score API example <FuriosaLLMExamplesScore>`.

API Reference
=============

See :ref:`Rerank API Reference <RerankAPI>` for complete parameter documentation.
