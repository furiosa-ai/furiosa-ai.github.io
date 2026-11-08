.. _VisionLanguageModels:

****************************************************
Vision-Language Models
****************************************************

Furiosa-LLM serves Vision-Language (VL) models that accept text and
images through the OpenAI-compatible Chat Completions API and the
:ref:`Responses API <ResponsesAPI>`. The first
supported architecture is **Qwen3-VL** (e.g.,
``Qwen/Qwen3-VL-32B-Instruct``); see :ref:`SupportedModels` for the
authoritative list.

Image inputs may be passed as a remote ``https://`` URL, a base64
``data:`` URL, or a local ``file://`` path. Images are automatically
resized and prepared for the selected model.

.. note::

   **Video input is not currently supported.** Support is planned for
   **Furiosa SDK 2026.5.0**.


Quick Start
===========

Launch a server with a vision-language model that includes a compatible FXB:

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

1. **Remote URL** (``http://`` / ``https://``) — the server downloads the
   image.

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
     - Maximum number of videos allowed per request. Video input is not
       currently supported (see the note above), so this flag has no
       effect yet.
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
     - Size of the multimodal preprocessing cache in GiB. Set to ``0`` to
       disable preprocessing caching.
       See *Caching Multimodal Inputs with UUID* below.
   * - ``--media-io-kwargs JSON``
     - ``{}``
     - Default image-fetch options, keyed by modality. See
       :ref:`MultimodalOptions` for timeouts and request overrides.
   * - ``--mm-processor-kwargs JSON``
     - ``{}``
     - Processor defaults keyed by modality. See
       :ref:`MultimodalOptions` for model-specific controls and request overrides.

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


.. _MultimodalOptions:

Multimodal Options
==================

``media_io_kwargs`` and ``mm_processor_kwargs`` use modality-keyed objects.
Only the ``image`` modality is currently supported. Set server defaults with the CLI:

.. code-block:: bash

   furiosa-llm serve furiosa-ai/Qwen3-VL-32B-Instruct \
       --media-io-kwargs '{"image": {"timeout": 10}}' \
       --mm-processor-kwargs '{"image": {"min_pixels": 1024}}'

Chat Completions and Responses requests can override these defaults with
``media_io_kwargs`` and ``mm_processor_kwargs`` in the JSON request body.
With the OpenAI Python client, pass these fields through ``extra_body``:

.. code-block:: python

   import os
   from openai import OpenAI

   client = OpenAI(
       base_url=os.getenv("OPENAI_BASE_URL", "http://localhost:8000/v1"),
       api_key=os.getenv("OPENAI_API_KEY", "EMPTY"),
   )
   response = client.chat.completions.create(
       model=client.models.list().data[0].id,
       messages=[{
           "role": "user",
           "content": [
               {"type": "image_url", "image_url": {
                   "url": "https://developer.furiosa.ai/latest/en/_images/Furiosa_RNGD_image.png"
               }},
               {"type": "text", "text": "Describe this image."},
           ],
       }],
       max_completion_tokens=32,
       extra_body={
           "media_io_kwargs": {"image": {"timeout": 5}},
           "mm_processor_kwargs": {"image": {"min_pixels": 4096}},
       },
   )
   print(response.choices[0].message.content)

Overrides apply only to that request and do not change server defaults.
Omitting either dictionary, setting it to ``null``, or passing ``{}`` retains
defaults. For both dictionaries, request options override defaults within each
modality. Passing ``{"image": {}}`` also retains defaults.

Image-fetch options
~~~~~~~~~~~~~~~~~~~

Request option keys override defaults within each modality. Passing
``{"image": {}}`` retains default image-fetch options.

The following fetch option is supported:

.. list-table::
   :header-rows: 1
   :widths: 25 25 15 35

   * - Option
     - Validation
     - Default
     - Behavior
   * - ``media_io_kwargs.image.timeout``
     - Finite positive number of seconds; strings, booleans, and ``null`` are
       rejected.
     - 30 seconds when omitted from both server defaults and the request.
     - Controls HTTP fetching, not image processing or generation. Has no effect
       on local files, inline data URLs, or references without a URL.

The server's local-path and domain restrictions still apply and cannot be
overridden through this dictionary.

Processor options
~~~~~~~~~~~~~~~~~

Place image processor options inside the ``image`` object:

.. code-block:: json

   {"mm_processor_kwargs": {"image": {"min_pixels": 1024, "max_pixels": 200704}}}

Request options override matching defaults within ``image`` while retaining
other default options. Passing ``{"image": {}}`` preserves all image defaults.
Each option value is replaced atomically: an override of ``size`` replaces the
entire default ``size`` object without merging its dimensions.

Registered image processor options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The catalog lists recommended runtime controls, rather than every upstream
processor argument. The common options below apply to Qwen-VL, EXAONE 4.5,
and Gemma 4. EXAONE uses the Qwen image processor with additional
window-aligned resizing.
Defaults come from the loaded processor configuration. The table shows the
Transformers 5.8 class fallbacks; checkpoint configuration can override them,
including normalization values. Omitted options use those loaded defaults.
An explicit ``null`` is forwarded as such and is not generally equivalent to
omission.

.. list-table::
   :header-rows: 1
   :widths: 20 20 25 35

   * - Option
     - Accepted values
     - Default
     - Behavior
   * - ``do_convert_rgb``
     - Boolean or ``null``
     - ``true``
     - Convert inputs to RGB. EXAONE performs this before window-aligned resizing.
   * - ``resample``
     - Integer ``0`` through ``5``, or ``null``
     - ``3`` (bicubic)
     - Pillow filter: nearest (0), Lanczos (1), bilinear (2), bicubic (3), box (4), or Hamming (5).
       The selected backend determines supported filters.
   * - ``do_rescale``
     - Boolean or ``null``
     - ``true``
     - Multiply pixel values by ``rescale_factor``.
   * - ``rescale_factor``
     - Finite number or ``null``
     - ``1 / 255``
     - Pixel rescaling multiplier, used when ``do_rescale`` is enabled.
   * - ``do_normalize``
     - Boolean or ``null``
     - Qwen: ``true``. Gemma: ``false``.
     - Normalize pixels with ``image_mean`` and ``image_std``.
   * - ``image_mean``
     - Number, sequence of numbers, or ``null``
     - Qwen: ``[0.48145466, 0.4578275, 0.40821073]``. Gemma: ``[0, 0, 0]``.
     - Scalar or per-channel mean; used when normalization is enabled.
   * - ``image_std``
     - Number, sequence of numbers, or ``null``
     - Qwen: ``[0.26862954, 0.26130258, 0.27577711]``. Gemma: ``[1, 1, 1]``.
     - Scalar or per-channel standard deviation; used when normalization is enabled.
   * - ``input_data_format``
     - String or ``null``
     - ``null`` (infer from input)
     - Input channel placement: ``channels_first`` or ``channels_last``.
   * - ``device``
     - Device string or ``null``; offline also accepts ``torch.device``
     - ``null`` (input device)
     - Device for image processing, such as ``cpu``. PIL images start on CPU.

Model-specific controls are listed below. Omitted sizing controls retain
Furiosa's automatic sizing. EXAONE always performs window-aligned resizing.

.. list-table::
   :header-rows: 1
   :widths: 20 20 25 35

   * - Option
     - Model family
     - Accepted values
     - Default and behavior
   * - ``do_resize``
     - Qwen-VL / Gemma 4
     - Boolean or ``null``
     - ``true``. Enable image resizing.
   * - ``disable_grouping``
     - Qwen-VL / EXAONE 4.5
     - Boolean or ``null``
     - ``null`` (automatic). Disable grouping equal-sized images for processing.
   * - ``min_pixels``
     - Qwen-VL / EXAONE 4.5
     - Positive integer
     - Selected automatically for the model.
       Minimum resize pixel count; must not exceed ``max_pixels``.
   * - ``max_pixels``
     - Qwen-VL / EXAONE 4.5
     - Positive integer
     - Selected automatically from the image and the model's supported sizes.
       Maximum resize pixel count; EXAONE also enforces window alignment.
   * - ``max_soft_tokens``
     - Gemma 4
     - Integer: ``70``, ``140``, ``280``, ``560``, or ``1024``
     - Selected automatically from the image and the model's supported limits.

``images`` and ``return_tensors`` cannot be set through processor options.

Strings, booleans, floats, and ``null`` are rejected for ``min_pixels``,
``max_pixels``, and ``max_soft_tokens``. Defaults, request overrides, and their
merged result are validated. Cross-field bounds are checked on the combined
options. Unregistered options continue to pass through; the loaded processor
determines whether they are supported. Changing these options can also require
an image to be processed again.

Images must fit within the selected model's supported sizes. A valid option
value alone does not guarantee that an image can be processed. If a request
exceeds these limits, reduce the image size or the number of images.

Repeated images can reuse previous preprocessing results. Changing a processor
option such as ``do_rescale`` may require an image to be processed again. Adding
or removing other images from a request can also change automatic sizing and
prevent reuse, even when the supplied options are unchanged. Automatic sizing
follows the same rules whether images are supplied again or referenced by UUID,
and whether previous preprocessing results are available.

A UUID continues to identify the original image, but does not guarantee that a
previous result can be reused. If a UUID-only request fails because no reusable
result is available, retry with the image attached. Explicitly setting an option
that was previously left to the loaded processor's default can also prevent
reuse. Empty request overrides still retain server defaults as described above.

Validation errors
~~~~~~~~~~~~~~~~~

Invalid request options return HTTP 400 or 422; invalid offline options raise
``ValueError``. Invalid CLI option structures or fetch options are argument
errors, while model-specific processor validation occurs when loading the
model. These checks also apply to requests without images.

If image processing fails with ``MultimodalTokenCountError``, no text is
generated. Chat Completions and Responses return HTTP 500
``InternalServerError`` as JSON, including for streaming requests.
Offline callers receive ``MultimodalTokenCountError`` (a ``RuntimeError``);
``LLMEngine`` delivers it through ``step()``, and ``AsyncLLMEngine`` raises it
while iterating ``generate()``.

See :ref:`TokenizerAPI` for tokenization-specific behavior and
:ref:`ResponsesMultiTurn` for option handling with stored conversations.


Caching Multimodal Inputs with UUID
===================================

Images with the same content can reuse cached preprocessing results even when
no ``uuid`` is supplied. Reuse skips preprocessing when the image's processing
settings, including automatic sizing, match those used for the cached result,
as described in :ref:`MultimodalOptions`.

You can reuse an image with different text or at a different position in a
later prompt. Each request must still include the required image placeholders
and fit within the model's context length. A new response is generated for
each request.

You can optionally tag each image with a stable ``uuid`` to reference its
cached result on follow-up requests without resending the image. Caching is
available with or without a UUID.

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

You may need to resend an image if its UUID is unknown, its cached result has
been evicted, or its processing options or automatic sizing have changed. If
``image_url`` is supplied, the server can process the image again. Otherwise,
it returns HTTP 400 ``unknown multimodal uuid``; retry with the image attached.

A UUID-only reference also avoids fetching the image. Supplying a URL still
causes the server to fetch and decode the image, even if it can reuse previous
preprocessing results. Offline callers can use ``multi_modal_uuids`` for the
same reuse behavior (see *Offline API* below).

Cache capacity is controlled by ``--mm-processor-cache-gb`` (default
``4.0``, ``0`` disables preprocessing caching). This limits memory used for
cached preprocessing results; it does not limit total server memory. Older
results may be removed when the cache is full. UUID-only requests require an
available cached result and cannot be used when preprocessing caching is
disabled. Send follow-up requests to the same server; cached results and UUIDs
are cleared on restart.


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
           "mm_processor_kwargs": {"image": {"min_pixels": 4096}},
       }
   ]

   with LLM(
       "Qwen/Qwen3-VL-32B-Instruct",
       fxb="/path/to/qwen3-vl-32b.fxb",
       mm_processor_kwargs={"image": {"min_pixels": 1024}},
   ) as llm:
       outputs = llm.generate(prompts, SamplingParams(max_tokens=64))
       print(outputs[0].outputs[0].text)

``LLM(mm_processor_kwargs=...)`` sets processor defaults. Each prompt's
``mm_processor_kwargs`` overrides those defaults using the same
:ref:`processor merge rules <MultimodalOptions>` as the server; the example
uses ``min_pixels=4096`` for this prompt. Offline callers supply decoded images,
so ``media_io_kwargs`` is not an offline ``LLM`` option.

Content-based caching and the UUID references described in
*Caching Multimodal Inputs with UUID* above are also available through the
offline API. Add ``multi_modal_uuids`` alongside ``multi_modal_data``, in the
same order; pass ``None`` for items that do not need a UUID reference. Those
items still participate in content-based caching.

Each modality's UUID list must have exactly as many entries as its image list.
A bare UUID string is also accepted for a single image. Repeating a UUID keeps
each occurrence in the prompt: provide a corresponding image placeholder for
every occurrence, even when they reuse the same cached result.

For example:

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
