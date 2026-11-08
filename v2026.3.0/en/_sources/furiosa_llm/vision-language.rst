.. _VisionLanguageModels:

****************************************************
Vision-Language Models
****************************************************

Furiosa-LLM serves Vision-Language (VL) models that accept text and
images through the OpenAI-compatible Chat Completions API. The first
supported architecture is **Qwen3-VL** (e.g.,
``Qwen/Qwen3-VL-32B-Instruct``); see :ref:`SupportedModels` for the
authoritative list.

Image inputs may be passed as a remote ``https://`` URL, a base64
``data:`` URL, or a local ``file://`` path. Preprocessing (resizing,
patching, prompt-token expansion) is handled by the Hugging Face
``AutoProcessor`` that ships with the model.


Quick Start
===========

Launch a server against a compiled VL artifact:

.. code-block:: bash

   furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct

Send a chat-completion request with an image and a text prompt:

.. literalinclude:: ../../../examples/openai_curl_vl.sh
   :language: bash

(see the :ref:`VLPythonExample` section below for a Python client
example).


Image Input Formats
===================

The ``image_url.url`` field in a chat-completion message accepts three
URL schemes:

1. **Remote URL** (``http://`` / ``https://``) — the server fetches the
   image with ``httpx``.

   .. code-block:: json

      {
        "type": "image_url",
        "image_url": {"url": "https://example.com/photo.png"}
      }

2. **Base64 data URL** — useful for inline payloads:

   .. code-block:: json

      {
        "type": "image_url",
        "image_url": {"url": "data:image/png;base64,iVBORw0KGgoAAAANSUh..."}
      }

3. **Local file** (``file://``) — disabled by default; requires the
   ``--allowed-local-media-path`` server flag (see below).

   .. code-block:: json

      {
        "type": "image_url",
        "image_url": {"url": "file:///srv/media/photo.png"}
      }

A request may contain multiple ``image_url`` parts interleaved with
``text`` parts. The model's chat template controls how image
placeholders are rendered into the prompt:

* **OpenAI-format chat templates** (the common case) iterate the
  ``content`` array themselves, so image placeholders land at their
  original positions automatically — no extra server flag needed.
* **String-format chat templates** flatten ``content`` into a single
  text string. By default, placeholders for any images are prepended
  to the prompt; pass ``--interleave-mm-strings`` to keep them at
  their original positions instead.


Server Options
==============

The following ``furiosa-llm serve`` flags configure multimodal
behaviour. Requests that violate any of them are rejected with HTTP
400 before reaching the NPU.

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 220 100 400

   * - Flag
     - Default
     - Description
   * - ``--image-limit-per-prompt N``
     - unlimited
     - Maximum number of images allowed per request. Exceeding the
       limit returns HTTP 400.
   * - ``--video-limit-per-prompt N``
     - unlimited
     - Maximum number of videos allowed per request. Exceeding the
       limit returns HTTP 400.
   * - ``--allowed-local-media-path PATH``
     - disabled
     - Allow ``file://`` URLs whose resolved path is under ``PATH``.
       Required to serve local images; other ``file://`` URLs are
       rejected.
   * - ``--allowed-media-domains D [D …]``
     - all allowed
     - Whitelist of remote domains for SSRF protection. When set, only
       images from listed domains are fetched; others return HTTP 400.
   * - ``--interleave-mm-strings``
     - off
     - See *Image Input Formats* above. No-op for the common
       OpenAI-format chat templates.
   * - ``--mm-processor-cache-gb GB``
     - ``4.0``
     - Size of the UUID-keyed multimodal cache in GiB. Set to ``0`` to
       disable caching entirely. See *Caching Multimodal Inputs with UUID* below.

Example with multiple allowed domains (``nargs="+"``):

.. code-block:: bash

   furiosa-llm serve Qwen/Qwen3-VL-32B-Instruct \
       --fxb /path/to/qwen3-vl-32b.fxb \
       --allowed-media-domains developer.furiosa.ai cdn.example.com

.. warning::

   Local file access (``file://``) is disabled unless
   ``--allowed-local-media-path`` is set. Treat it as a security
   boundary — only point it at directories whose contents you are
   willing to expose to chat-completion clients.

.. tip::

   For production deployments fetching from the public internet,
   restrict ``--allowed-media-domains`` to the smallest set you need to
   limit SSRF and bandwidth-amplification risk.


Caching Multimodal Inputs with UUID
===================================

You can optionally tag each image or video on a request with a stable
``uuid``. The server caches the processed result under that UUID and
reuses it on follow-up requests carrying the same UUID, skipping
pre-processing.

Note that items without a ``uuid`` are not implicitly hashed by content;
only explicitly tagged items participate in the cache.

Add ``uuid`` next to ``image_url`` on the content part:

.. code-block:: json

   {
     "type": "image_url",
     "image_url": {"url": "https://example.com/receipt.png"},
     "uuid": "receipt-2026-05-01-abc"
   }

A follow-up may then omit the image bytes (``"image_url": null``) and
reference the upload by ``uuid`` alone:

.. code-block:: json

   {
     "type": "image_url",
     "image_url": null,
     "uuid": "receipt-2026-05-01-abc"
   }

If no entry for that UUID exists (unknown UUID or cache eviction),
the server re-runs pre-processing when ``image_url`` is also supplied;
otherwise it returns HTTP 400 ``unknown multimodal uuid``, and the
client should retry with the image attached. The same shape is
available offline via ``multi_modal_uuids`` (see *Offline API* below).

Cache capacity is controlled by ``--mm-processor-cache-gb`` (default
``4.0``, ``0`` disables). The cap is on the total size of cached
entries; the least-recently-used entries are evicted when the cache
is full.


.. _VLPythonExample:

Python Example
==============

Using the OpenAI Python client against a Furiosa-LLM server:

.. literalinclude:: ../../../examples/online_vl_chat_completion.py
   :language: python


Offline API
===========

The same image inputs are accepted by the offline ``LLM`` / ``LLMEngine``
APIs via the ``multi_modal_data`` field on the prompt dict. Pass PIL
images keyed by modality:

.. code-block:: python

   from PIL import Image
   from furiosa_llm import LLM, SamplingParams

   image = Image.open("photo.png")
   prompts = [
       {
           "prompt": "<|vision_start|><|image_pad|><|vision_end|>Describe this image.",
           "multi_modal_data": {"image": [image]},
       }
   ]

   with LLM("Qwen/Qwen3-VL-32B-Instruct", fxb="/path/to/qwen3-vl-32b.fxb") as llm:
       outputs = llm.generate(prompts, SamplingParams(max_tokens=64))
       print(outputs[0].outputs[0].text)

The UUID cache described in *Cached Inputs (UUID)* above is also
available through the offline API. Add ``multi_modal_uuids`` alongside
``multi_modal_data``, in the same order; pass ``None`` for items the
client does not wish to cache:

.. code-block:: python

   prompts = [
       {
           "prompt": "<|vision_start|><|image_pad|><|vision_end|>Describe this image.",
           "multi_modal_data": {"image": [image]},
           "multi_modal_uuids": {"image": ["receipt-2026-05-01-abc"]},
       }
   ]

A follow-up prompt may then omit the image and reference it by UUID
alone — pass ``None`` in the ``multi_modal_data`` slot:

.. code-block:: python

   followup = [
       {
           "prompt": "<|vision_start|><|image_pad|><|vision_end|>Translate to Korean.",
           "multi_modal_data": {"image": [None]},
           "multi_modal_uuids": {"image": ["receipt-2026-05-01-abc"]},
       }
   ]

See :ref:`FuriosaLLMReference` for the full API surface.


See Also
========

* :ref:`SupportedModels`
* :ref:`OpenAIServer`
* :ref:`GettingStartedFuriosaLLM`
