.. _FuriosaLLM:

****************************************************
Furiosa LLM
****************************************************

Furiosa LLM is a high-performance inference engine for LLM models and Multi-Modal LLM models,
Furiosa LLM is designed to provide the state-of-the-art serving optimization.
The features of Furiosa LLM includes:

* vLLM-compatible API
* Efficient KV cache management with PagedAttention
* Continuous batching of incoming requests in serving
* Quantization: INT4, INT8, FP8, GPTQ, AWQ
* Data Parallelism and Pipeline Parallelism across multiple NPUs
* Tensor Parallelism (planned in release 2024.2) across multiple NPUs
* OpenAI-compatible API server
* Various decoding algorithms, greedy search, beam search, top-k/top-p, speculative decoding (planned in 2024.2)
* HuggingFace model integration and hub support
* HuggingFace PEFT support (planned)


Documentation
--------------------------------------------
* :ref:`GettingStartedFuriosaLLM`: A quick start guide to Furiosa LLM
* :ref:`OpenAIServer`: More details about the OpenAI-compatible server and its features
* :ref:`ModelPreparationWorkflow`: How to quantize and compile Huggingface models
* :ref:`ModelParallelism`: Tensor/Pipeline/Data parallelism in Furiosa LLM
* :ref:`FuriosaLLMReference`: The Python API reference for Furiosa LLM

.. \* :ref:`BuildingModelArtifact`: More details about building model artifacts


