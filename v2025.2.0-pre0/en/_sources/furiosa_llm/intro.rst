.. _FuriosaLLM:

****************************************************
Furiosa-LLM
****************************************************

Furiosa-LLM is a high-performance inference engine for LLM and multi-modal
LLM models.
Furiosa-LLM offers state-of-the-art serving efficiency and optimizations.
Key features of Furiosa-LLM include:

* vLLM-compatible API (LLM, LLMEngine, AsyncLLMEngine API)
* Efficient KV cache management with PagedAttention
* Continuous batching of incoming requests
* Quantization: INT4, INT8, FP8, GPTQ, AWQ
* Support for data, tensor, and pipeline parallelism across multiple NPUs
* OpenAI-compatible API server
* Various decoding algorithms: greedy search, beam search, top-k/top-p, and
  speculative decoding (planned for 2025.1)
* Support for context lengths of up to 32k
* Integration with Hugging Face models and hub support
* Hugging Face PEFT support (planned)


Documentation
-------------
* :ref:`GettingStartedFuriosaLLM`: A quick start guide to Furiosa-LLM
* :ref:`OpenAIServer`: Details about the OpenAI-compatible server and its features
* :ref:`ModelPreparation`: How to prepare LLM models to be served by Furiosa-LLM
* :ref:`BuildingModelArtifactsByExamples`: Guide on building model artifacts by examples
* :ref:`ModelParallelism`: Model parallelism guide for Furiosa-LLM
* :ref:`FuriosaLLMBuildCommand`: Command line tool for building model artifacts
* :ref:`FuriosaLLMReference`: Python API reference for Furiosa-LLM
* :ref:`FuriosaLLMExamples`: Examples of using Furiosa-LLM

