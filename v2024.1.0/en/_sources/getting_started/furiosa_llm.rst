.. _GettingStartedFuriosaLLM:

**********************************
Quick Start with Furiosa LLM
**********************************

Furiosa LLM is a serving framework for LLM models that utilizes FuriosaAI's NPU.
It provides a Python API compatible with vLLM and a server compatible with the OpenAI API.
This document explains how to install and use Furiosa LLM.

.. warning::

   This document is based on Furiosa SDK 2024.1.0 (alpha) version,
   and the features and APIs described in this document may change in the future.


.. _InstallingFuriosaLLM:

Installing Furiosa LLM
=========================================

The minimum requirements for Furiosa LLM are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`
* Python 3.8, 3.9, or 3.10
* PyTorch 2.1.0
* Enough storage space for model weights; e.g., about 100GB for Llama 3.1 70B model

You need to install ``furiosa-compiler`` and the Furiosa LLM with the following command:

.. code-block:: sh

  sudo apt install -y furiosa-compiler

  pip install --upgrade pip setuptools wheel
  pip install furiosa-llm


.. _AuthorizingHuggingFaceHub:
Authorizing HuggingFace Hub (Optional)
-----------------------------------------
Some models, such as meta-llama/Meta-Llama-3.1-8B require you to accept their license,
hence, you need to create a HuggingFace account, accept the model's license, and generate a token.
You can create your token at https://huggingface.co/settings/tokens.
Once you get a token, you can authenticate on the HuggingFace Hub as following:

.. code-block::

  huggingface-cli login --token $HF_TOKEN


Offline Batch Inference with Furiosa LLM
------------------------------------------------------
In this section, we will explain how to perform offline LLM inference using the Python API of Furiosa LLM.
First, import the ``LLM`` class and ``SamplingParams`` from the furiosa_llm module.
``LLM`` class is used to load LLM models and provides the core API for LLM inference.
``SamplingParams`` allows to specify various parameters for text generation.

.. code-block:: python

  from furiosa_llm import LLM, SamplingParams

  # Loading an artifact of Llama 3.1 8B Instruct model
  path = "./Llama-3.1-8B-Instruct"
  llm = LLM.from_artifacts(path)

  # You can specify various parameters for text generation
  sampling_params = SamplingParams(min_tokens=10, top_p=0.3, top_k=100)

  # Generate text
  prompts = ["Say this is test"]
  response = llm.generate(prompts, sampling_params)

  # Print outputs
  print(response[0].outputs[0].text)


Streaming Inference with Furiosa LLM
------------------------------------------------------
In addition to batch inference, Furiosa LLM also supports streaming inference.
The key difference of streaming inference is that tokens are returned as soon as tokens are generated, allowing you to start printing or processing partial tokens before the full completion is finished.
To perform streaming inference, the ``stream_generate`` method is used instead of ``generate``. This method is asynchronous and returns a stream of tokens as they are generated.

.. code-block:: python

  import asyncio
  from furiosa_llm import LLM, SamplingParams

  async def main():
      # Loading an artifact of Llama 3.1 8B Instruct model
      path = "./Llama-3.1-8B-Instruct"
      llm = LLM.from_artifacts(path)

      # You can specify various parameters for text generation
      sampling_params = SamplingParams(min_tokens=10, top_p=0.3, top_k=100)

      # Generate text and print outputs
      prompt = "Say this is test"
      async for output_txt in llm.stream_generate(prompt, sampling_params):
          print(output_txt, end="", flush=True)

  # Run the async main function
  if __name__ == "__main__":
      asyncio.run(main())

Workaround for Chat Template
------------------------------------------
Chat models have been trained with very different prompt formats.
Especially, Llama 3.1 models require a specific prompt format.
You can refer to `Llama model card <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/>`_
for more details.

HuggingFace Transformers 4.41.0 starts to support chat templates.
However, Furiosa LLM currently is based on Transformers 4.31.0, which does not include a chat template feature.
Here, we provide a workaround to apply a chat template to the prompt.
We use Llama 3.1 8B Instruct model as an example.

Here, we define a function that applies a chat template to the prompt.
This function is based on the Llama 3.1 8B Instruct model's chat template format described at
`Instruct Model Prompt <https://www.llama.com/docs/model-cards-and-prompt-formats/llama3_1/#-instruct-model-prompt->`_.`

.. code-block:: python

  from furiosa_llm import LLM, SamplingParams

  def apply_template(prompt):
    return f"""<|begin_of_text|><|start_header_id|>system<|end_header_id|>

    You are a helpful assistant<|eot_id|><|start_header_id|>user<|end_header_id|>

    {prompt}<|eot_id|><|start_header_id|>assistant<|end_header_id|>"""


  path = "./Llama-3.1-8B-Instruct"
  llm = LLM.from_artifacts(path)

  prompt1 = apply_template("What is the capital of France?")
  prompt2 = apply_template("Say something nice about me.")

  sampling_params = SamplingParams(min_tokens=10, top_p=0.3, top_k=100)
  responses = llm.generate([prompt1, prompt2], sampling_params)

  for response in responses:
    print(response.outputs[0].text)


Launching the OpenAI-Compatible Server
=========================================

Furiosa LLM can be deployed as a server that provides an API compatible with OpenAI API.
Since many LLM frameworks and applications are built on top of OpenAI API protocol,
you can easily integrate Furiosa LLM into your existing applications.

By default, the server provides the HTTP endpoint http://localhost:8000.
You can change the binding address and port by specifying the ``--host`` and ``--port`` options.
The server can host only one model at a time for now and provides a chat template feature.
You can find more details at :ref:`OpenAIServer`.

The following is an example of launching the server with the Llama 3.1 8B Instruct model.

.. code-block::

  # Download the chat template for LLama 3.1 Instruct Model
  wget https://gist.githubusercontent.com/hyunsik/16f1906af7ac2b4db41af9957a66e168/raw/62935b0c24c03669208cf90f3f87b1694521053d/Llama-3.1-Instruct.tpl

  # Launch the server
  furiosa-llm serve \
    --model ./Llama-3.1-8B-Instruct \
    --chat-template ./Llama-3.1-Instruct.tpl

You can simply test the server using the following curl command:

.. code-block::

  curl http://localhost:8000/v1/chat/completions \
      -H "Content-Type: application/json" \
      -d '{
      "model": "EMPTY",
      "messages": [{"role": "user", "content": "What is the capital of France?"}]
      }' \
      | python -m json.tool

Output:

.. code-block:: json

  {
    "id": "chat-fa68698788084cc6ae2f327dae37d74c",
    "object": "chat.completion",
    "created": 1728618939,
    "model": "meta-llama/Meta-Llama-3.1-8B-Instruct",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "The capital of France is Paris.",
                "tool_calls": []
            },
            "logprobs": null,
            "finish_reason": null,
            "stop_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 42,
        "total_tokens": 49,
        "completion_tokens": 7
    },
    "prompt_logprobs": null
  }
