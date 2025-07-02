.. _SupportedModels:

**********************************
Supported Models
**********************************

FuriosaAI's software stack supports a wide range of Transformer-based models available on the Hugging Face Hub.
Below is a list of model architectures currently supported by the Furiosa SDK.
If your model is based on any of these architectures, you can use the
Furiosa SDK to compile, quantize, and run the model efficiently on
Furiosa's NPUs.

.. tip::

  Pre-compiled models are available in `Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_.
  You can download and run them quickly with Furiosa-LLM.


Decoder-only Models
===================

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Model Name
     - Architecture
     - Example Hugging Face Models
   * - CodeLlama
     - ``LlamaForCausalLM``
     - ``meta-llama/CodeLlama-7b-Instruct-hf``
   * - DeepSeek R1
     - ``LlamaForCausalLM``
     - ``deepseek-ai/DeepSeek-R1-Distill-Llama-8B``, ``deepseek-ai/DeepSeek-R1-Distill-Llama-70B``
   * - EXAONE 3.0, EXAONE 3.5
     - ``ExaoneForCausalLM``
     - ``LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct``, ``LGAI-EXAONE/EXAONE-3.5-7.8B-Instruct``
   * - GPT-J
     - ``GPTJForCausalLM``
     - ``EleutherAI/gpt-j-6b``
   * - Llama 2, Llama 3.1, Llama 3.3
     - ``LlamaForCausalLM``
     - ``meta-llama/Llama-3.1-8B-Instruct``, ``meta-llama/Llama-3.1-70B-Instruct``, ``meta-llama/Llama-3.3-70B-Instruct``, ..
   * - Solar
     - ``LlamaForCausalLM``
     - ``upstage/SOLAR-10.7B-v1.0``, ``upstage/SOLAR-10.7B-Instruct-v1.0``


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
(The order of the models listed below indicate the priority of support.)

* Qwen/QwQ-32B
* gemma-3
* Llama 4
* Mistral-7B-v0.1, Mistral-7B-Instruct-v0.1
* Mixtral-8x7B-v0.1, Mixtral-8x7B-Instruct-v0.1
* Phi-3, Phi-3.5,
* Phi-3.5-MoE
* DeepSeek-Coder-V2-Instruct
* Stable Diffusion XL

