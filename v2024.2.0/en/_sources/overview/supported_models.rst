.. _SupportedModels:

**********************************
Supported Models
**********************************

FuriosaAI Software Stack supports a variety of Transformer-based models in HuggingFace Hub.
The following is the list of model architectures that are currently supported by Furiosa SDK.
If your model is based on the following architectures,
you can use Furiosa SDK to compile, quantize, and run the model on FuriosaAI RNGD.

Decoder-only Models
====================================

.. list-table:: Decoder-only Models
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Architecture
     - Model Name
     - Example HuggingFace Models
   * - ``LlamaForCausalLM``
     - Llama 2, Llama 3.1
     - ``meta-llama/Llama-2-70b-hf``, ``meta-llama/Llama-3.1-70B``, ``meta-llama/Llama-3.1-70B-Instruct``, ``meta-llama/Llama-3.1-8B``, ``meta-llama/Llama-3.1-8B-Instruct``, ..
   * - ``GPTJForCausalLM``
     - GPT-J
     - ``EleutherAI/gpt-j-6b``


Encoder-only Models
====================================

.. list-table:: Encoder-only Models
   :align: center
   :header-rows: 1
   :widths: 130 120 300

   * - Architecture
     - Model Name
     - Example HuggingFace Models
   * - ``BertForQuestionAnswering``
     - Bert
     - ``google-bert/bert-large-uncased``, ``google-bert/bert-base-uncased``, ..


Planned Models for Future Releases
===============================================

* 2024.2 (October 31, 2024)
    * Language Models
        * SOLAR-10.7B-v1.0, SOLAR-10.7B-Instruct-v1.0
        * EXAONE-3.0-7.8B-Instruct
        * vicuna-7b-v1.5
        * CodeLlama-7b-Instruct-hf
        * RoBERTa-base, RoBERTa-large
    * Vision Models
        * MobileNetV1, MobileNetV2
        * YOLOv8m
* TDB:
    * Falcon
    * Mistral-7B-v0.1, Mistral-7B-Instruct-v0.1
    * Mixtral-8x7B-v0.1, Mixtral-8x7B-Instruct-v0.1
    * Phi-3, Phi-3.5,
    * Phi-3.5-MoE
    * Qwen2
    * gemma-2
    * DeepSeek-Coder-V2-Instruct
    * Stable Diffusion XL
    * Llama 3.2
