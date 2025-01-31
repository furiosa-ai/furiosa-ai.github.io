.. _SupportedModels:

**********************************
Supported Models
**********************************

FuriosaAI's software stack supports a wide range of Transformer-based models
available on the Hugging Face Hub.
Below is a list of model architectures currently supported by the Furiosa SDK.
If your model is based on any of these architectures, you can leverage the
Furiosa SDK to compile, quantize, and run the model efficiently on
Furiosa's NPUs.

Decoder-only Models
===================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - Llama 2, Llama 3.1
     - ``LlamaForCausalLM``
     - ``meta-llama/Llama-2-70b-hf``, ``meta-llama/Llama-3.1-70B``, ``meta-llama/Llama-3.1-70B-Instruct``, ``meta-llama/Llama-3.1-8B``, ``meta-llama/Llama-3.1-8B-Instruct``, ..
   * - GPT-J
     - ``GPTJForCausalLM``
     - ``EleutherAI/gpt-j-6b``
   * - Solar
     - ``LlamaForCausalLM``
     - ``upstage/SOLAR-10.7B-v1.0, upstage/SOLAR-10.7B-Instruct-v1.0``
   * - EXAONE
     - ``LlamaForCausalLM``
     - ``LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct``
   * - CodeLlama
     - ``LlamaForCausalLM``
     - ``meta-llama/CodeLlama-7b-Instruct-hf``

Encoder-only Models
===================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - Bert
     - ``BertForQuestionAnswering``
     - ``google-bert/bert-large-uncased``, ``google-bert/bert-base-uncased``, ..


Planned Models for Future Releases
===============================================

* Falcon
* Mistral-7B-v0.1, Mistral-7B-Instruct-v0.1
* Mixtral-8x7B-v0.1, Mixtral-8x7B-Instruct-v0.1
* Phi-3, Phi-3.5,
* Phi-3.5-MoE
* Qwen2
* gemma-2
* DeepSeek-Coder-V2-Instruct
* Stable Diffusion XL
* Llama 3.2 multi-model models
* Llama 3.3
