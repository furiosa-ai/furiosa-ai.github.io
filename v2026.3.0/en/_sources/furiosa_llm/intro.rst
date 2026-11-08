.. _FuriosaLLM:

****************************************************
Furiosa-LLM
****************************************************

Furiosa-LLM is a high-performance inference engine for large language models
(LLMs) and multi-modal (vision-language) models.
Furiosa-LLM offers state-of-the-art serving efficiency and optimizations.
Key features of Furiosa-LLM include:

* vLLM-compatible API (LLM, LLMEngine, AsyncLLMEngine API)
* Text and multi-modal (vision-language) serving
* Generative and pooling model support (embedding, reranking, classification)
* Efficient KV cache management with PagedAttention
* Radix-tree prefix caching for reuse of shared prompt prefixes
* Hybrid KV cache for models that mix sliding-window and global attention
* Continuous batching of incoming requests
* Quantization: INT4, INT8, BF16, FP8, MXFP4, and NVFP4
* Support for data, tensor, and pipeline parallelism across multiple NPUs
* OpenAI-compatible API server with Chat Completions and Responses APIs
* Various decoding algorithms: greedy search, top-k/top-p, and speculative decoding (planned)
* Tool calling and reasoning parser support
* Structured output generation (choice, regex, json schema, grammar)
* Chunked prefill with mixed prefill/decode batching
* Prometheus metrics endpoint for serving observability
* Integration with Hugging Face models and hub support


Documentation
-------------
* :ref:`GettingStartedFuriosaLLM`: A quick start guide to Furiosa-LLM
* :ref:`Furiosa-LLM Models <SupportedModels>`: Model cards for the pre-compiled models published by FuriosaAI
* :ref:`OpenAIServer`: Details about the OpenAI-compatible server and its features
* :ref:`ResponsesAPI`: Guide to the OpenResponses-compatible Responses API
* :ref:`ToolCalling`: Guide to tool calling with parsers and choice options
* :ref:`StructuredOutput`: Guide to structured output generation
* :ref:`VisionLanguageModels`: Guide to serving Vision-Language models with image inputs
* :ref:`PrefixCaching`: Guide to prefix caching for improved performance
* :ref:`HybridKVCache`: Understanding hybrid KV cache management
* :ref:`DataParallelRouting`: Understanding scoring-based data-parallel routing
* :ref:`ModelPreparation`: How to prepare LLM models to be served by Furiosa-LLM
* :ref:`FXB`: Building, caching, and serving Furiosa Executable Bundles (FXB)
* :ref:`ModelParallelism`: A guide to model parallelism in Furiosa-LLM
* :ref:`FuriosaLLMReference`: Python API reference for Furiosa-LLM
* :ref:`FuriosaLLMExamples`: Examples of using Furiosa-LLM
* :ref:`K8sFuriosaLLMDeployment`: A guide to deploying Furiosa-LLM on Kubernetes
