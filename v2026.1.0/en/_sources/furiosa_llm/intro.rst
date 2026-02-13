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
* Quantization: FP8 (Planned: INT4, INT8, GPTQ, AWQ)
* Support for data, tensor, and pipeline parallelism across multiple NPUs
* OpenAI-compatible API server
* Various decoding algorithms: greedy search, top-k/top-p, and speculative decoding (planned for 2026.3)
* Tool calling and reasoning parser support
* Structured output generation with guided decoding (guided_choice, guided_regex, guided_json, guided_grammar)
* Chunked Prefill
* Integration with Hugging Face models and hub support
* Hugging Face PEFT support (planned)


Documentation
-------------
* :ref:`GettingStartedFuriosaLLM`: A quick start guide to Furiosa-LLM
* :ref:`OpenAIServer`: Details about the OpenAI-compatible server and its features
* :ref:`ToolCalling`: Guide to tool calling with parsers and choice options
* :ref:`StructuredOutput`: Guide to structured output generation with guided decoding
* :ref:`PrefixCaching`: Guide to prefix caching for improved performance
* :ref:`HybridKVCache`: Understanding hybrid KV cache management
* :ref:`ModelPreparation`: How to prepare LLM models to be served by Furiosa-LLM
* :ref:`ModelParallelism`: A guide to model parallelism in Furiosa-LLM
* :ref:`FuriosaLLMReference`: Python API reference for Furiosa-LLM
* :ref:`FuriosaLLMExamples`: Examples of using Furiosa-LLM
* :ref:`K8sFuriosaLLMDeployment`: A guide to deploying Furiosa-LLM on Kubernetes
