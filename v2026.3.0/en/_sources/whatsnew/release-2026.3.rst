.. _Release2026_3_0:

********************************************
Furiosa SDK Release 2026.3
********************************************

We are happy to announce the release of **Furiosa SDK 2026.3**.

This release is centered on **model coverage**. At its core is the introduction of the
**TCL (Tensor Contraction Language)** kernel framework and the **furiosa-kernels**
package, together with the **FXB (Furiosa Executable Bundle)** packaging format, which
make enabling and distributing a new model architecture dramatically faster. On that
foundation, 2026.3 adds first-class **multimodal (vision-language)** serving with
**Qwen3-VL**, brings up several new large MoE models (gpt-oss, Solar-Open, K-EXAONE,
Qwen3 MoE), introduces an **overlap scheduler** that runs one batch ahead to keep the NPU
continuously fed, and generalizes the Data Parallel router into a **scoring-based** policy.

If you are upgrading from 2026.2, please also read the
:ref:`Release2026_3_0_BreakingChanges` section and the :ref:`UpgradeGuide`.


Highlights
----------

TCL Kernel Framework and furiosa-kernels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The defining change in 2026.3 is the **TCL (Tensor Contraction Language)** kernel
framework. To see why adding a model got so much faster, it helps to look at how we used
to describe one.

The previous path started from a PyTorch model, traced with TorchDynamo into an FX graph
of Torch ATen ops. This was effective for getting a model running quickly, but it diluted the
model's intent: capturing through ATen decomposes the graph into fine-grained primitives.
An ``aten.mm``, for example, becomes a tiling, then an elementwise multiply, then a
reduce — so the compiler no longer sees a "matmul" and has to re-recognize one before it
can optimize it, and axis semantics and op boundaries blur in the same way. Just as
importantly, the modeling stage left few good places to attach the hints a compiler needs
for an RNGD target, such as the intended tensor parallelism (TP) strategy. The compiler
was essentially optimizing after much of the meaning had already been flattened away.

**TCL** was designed to preserve that model meaning and structure. It is a declarative
Python eDSL in which a kernel author writes a high-level, ``@tcl.kernel``-decorated
function that says *what* to compute and leaves *how* to run it — tiling, scheduling,
fusion, hardware mapping — to the compiler. Its key idea is to treat the **tensor
contraction** operations that DNN models intrinsically rely on as first-class primitives,
so a model is described directly in terms of those contractions, with its structure
intact. This matches RNGD's underlying **TCP (Tensor Contraction Processor)** architecture,
whose compiler is built around the same primitive — and a TCL kernel is then compiled down
to the executable binaries (EDFs) that run on it.

That design pays off in three ways:

* **Intuitive and compiler-friendly.** Authors express the computation's meaning rather
  than thread layouts or synchronization, and because TCL is built to be compiled, the
  toolchain can fuse multiple kernels and compile them as a single unit.
* **Modular and reusable.** Kernels like RMSNorm, Linear, or MLP become building blocks
  that many models share, so enabling a new architecture is mostly composing existing
  blocks and adding only what is genuinely new.
* **Native to RNGD.** Padding, sharding, broadcast, and multi-chip collectives are
  expressed at the language and type level, where the compiler can reason about their
  correctness and optimization together.

The upshot is that enablement now scales with the number of reusable blocks, not the
number of models. The **furiosa-kernels** package is the concrete result — a collection
of TCL kernels (attention, MoE, vision encoder, and architecture-specific blocks)
covering the families Furiosa-LLM supports — and the breadth of new models in this release
follows directly from it. As this library grows, we expect to keep bringing up new models
quickly in future releases.

New Model Families
^^^^^^^^^^^^^^^^^^

Building on the TCL framework, 2026.3 brings up a broad set of new models on RNGD,
including several large Mixture-of-Experts (MoE) architectures:

* **Qwen3-VL** (e.g. Qwen3-VL-32B) — the first **vision-language** family on RNGD: a
  dense transformer paired with a vision encoder (see *Multimodal Serving and Qwen3-VL*
  below).
* **gpt-oss** (e.g. gpt-oss-120b) — MoE family with **MXFP4**-quantized
  expert weights.
* **Solar-Open** (e.g. Solar-Open-100B) — MoE family, **NVFP4**-quantized weights with
  16-bit activations and KV cache (NVFP4A16).
* **Qwen3 MoE** (e.g. Qwen3-30B-A3B) — MoE family with **dynamic FP8 activation
  quantization** at runtime; Instruct, Thinking, and Coder variants.
* **K-EXAONE** (e.g. K-EXAONE-236B-A23B) — multilingual MoE family using a **hybrid
  sliding-window + global attention** scheme; NVFP4A16.

Each ships an FXB so it can be served directly from its Hugging Face repository; see the
per-model cards for the exact repository IDs and serving commands.

FXB: Furiosa Executable Bundle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2026.3 introduces the **Furiosa Executable Bundle (FXB)**, Furiosa-LLM's shareable
compiled-artifact format. An ``.fxb`` file is a single archive holding a
``manifest.json`` together with the compiled kernels (``edfs/``) needed to run a model on
the NPU. Once a model is compiled into an ``.fxb``, you can serve it without recompiling,
copy it to another machine, or publish it to the Hugging Face Hub for others to reuse.

The defining property of an FXB is its **architecture fingerprint**. The manifest records
the model architecture and the configuration fields that determine the compiled kernels
(hidden size, head counts, vocabulary size, quantization, and so on), so a single bundle
is reusable across *any* Hugging Face model that shares the same fingerprint — not just
the one it was built from. In practice this means you can serve a model whose own
repository ships no ``.fxb`` — including **fine-tuned or weight-updated variants** of a
supported model — by reusing a compatible bundle from your local cache, instead of
recompiling for every variant.

A dedicated ``fxb`` command manages the full lifecycle — building, downloading, caching,
compatibility checking, and inspection. For example, to serve ``Qwen/Qwen3-8B-FP8`` (which
ships no bundle of its own) by reusing the published, fingerprint-compatible
``furiosa-ai/Qwen3-8B-FP8`` bundle:

.. code-block:: bash

   # Download a published FXB bundle into the local cache
   fxb download furiosa-ai/Qwen3-8B-FP8

   # Confirm it is fingerprint-compatible with the target model
   fxb check Qwen/Qwen3-8B-FP8

   # Serve as usual — the compatible cached bundle is found automatically
   furiosa-llm serve Qwen/Qwen3-8B-FP8

At serving time Furiosa-LLM resolves the bundle in order: an explicit ``--fxb`` path, an
``.fxb`` shipped inside the model repository, then the local cache. FuriosaAI publishes
pre-compiled bundles for popular models on the
`Hugging Face Hub <https://huggingface.co/furiosa-ai>`_.

The fingerprint-based compatibility matching is **experimental**; the exact set of
fingerprint fields may change in future releases, so verify a match with ``fxb check``
before relying on a cached bundle for a different model. See :ref:`FXB` for the bundle
format, the compatibility-matching rules, and the full ``fxb`` command reference.

Multimodal Serving and Qwen3-VL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2026.3 introduces **vision-language (multimodal) serving** on RNGD, with **Qwen3-VL-32B**
as the first supported model. Image-and-text requests are served through the standard
OpenAI-compatible Chat Completions API using ``image_url`` content parts.

Multimodal serving is still **experimental** in this release: the scheduling and batching
of multimodal requests are still being optimized, with further improvements planned for
the next release.

A minimal example using the OpenAI Python client:

.. code-block:: python

   from openai import OpenAI

   client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

   response = client.chat.completions.create(
       model="Qwen/Qwen3-VL-32B-Instruct",
       messages=[{
           "role": "user",
           "content": [
               {"type": "image_url",
                "image_url": {"url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"}},
               {"type": "text", "text": "Describe this image."},
           ],
       }],
   )
   print(response.choices[0].message.content)

To avoid re-sending and re-preprocessing the same image across requests, multimodal
inputs can be tagged with a stable **UUID** and reused from a server-side processor
cache, sized via ``--mm-processor-cache-gb``. See :ref:`VisionLanguageModels` for the
supported models, request format, and reuse semantics.

Overlap Scheduling: Toward Zero-Overhead Batching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Between forward passes, an inference engine spends real CPU time on host-side work —
batch scheduling, KV-block allocation, and prefix matching against the radix cache.
When that work runs *between* NPU forward passes, the NPU sits idle waiting for the next
batch to be prepared, and on short decode steps this CPU overhead can become a
significant fraction of each iteration.

2026.3 adds an **overlap scheduler** that removes this stall by running **one
batch ahead**: while the NPU executes the current batch, the scheduler concurrently
prepares all the metadata for the next one. The host-side scheduling cost is overlapped
with NPU compute instead of serialized in front of it, keeping the NPU continuously fed.
For throughput-oriented workloads this improves overall **throughput and time per output
token (TPOT)**, at the cost of a small, bounded increase in **TTFT**.

This feature is still **experimental**, so it is **off by default** and the scheduling
loop remains **eager**; we plan to make it the default once it stabilizes. Enable the
overlap scheduler explicitly:

.. code-block:: bash

   furiosa-llm serve <model> --enable-overlap-scheduling

Scoring-Based Data Parallel Routing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The prefix-aware Data Parallel (DP) router introduced in 2026.2 evolves in
2026.3 into a **scoring-based** policy that balances two signals when picking a replica:
**prefix locality** (preferring a replica that already holds matching prefix cache
entries) and **token-footprint load** (preferring a less-loaded replica). The relative
weight of the two is selected through a scoring **profile**.

.. code-block:: bash

   # Scoring-based routing (default), balanced profile (default)
   furiosa-llm serve <model> --data-parallel-size <N>

   # Bias toward prefix cache affinity (equivalent to the old prefix-aware behavior)
   furiosa-llm serve <model> --data-parallel-size <N> \
       --data-parallel-routing-policy scoring \
       --data-parallel-scoring-profile locality

``--data-parallel-routing-policy`` accepts ``scoring`` (default) or ``round-robin``;
``--data-parallel-scoring-profile`` accepts ``balanced`` (default), ``locality``, or
``load``. See :ref:`DataParallelRouting` for details.

----

The complete list of changes in this release is summarized in the All Changes section
below.


.. _Release2026_3_0_AllChanges:

All Changes
-----------

Furiosa-LLM
^^^^^^^^^^^

Major Features & Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **TCL Kernel Framework and furiosa-kernels**

  * Introduced the **TCL (Tensor Contraction Language)** eDSL and the **furiosa-kernels**
    kernel collection as the primary path for model enablement.
  * Added **Mixture-of-Experts (MoE)** kernels (expert routing and gated expert compute),
    which enable the new MoE models in this release (gpt-oss, Solar-Open, K-EXAONE,
    Qwen3 MoE).
  * Added **sliding-window attention** kernels alongside full attention, so models that
    interleave local and global attention (such as K-EXAONE and gpt-oss) run efficiently.
  * Added **vision-encoder** kernels and **multimodal rotary position embeddings (mRoPE)**
    for the Qwen3-VL vision-language path.

* **FXB (Furiosa Executable Bundle)**

  * New shareable, compiled-artifact format bundling a ``manifest.json`` and the compiled
    kernels (``edfs/``) into a single ``.fxb`` file.
  * **Architecture-fingerprint compatibility** (experimental): a bundle is reusable across
    any Hugging Face model sharing the same fingerprint — including fine-tuned or
    weight-updated variants — so a model whose repository ships no ``.fxb`` can be served
    from a compatible cached bundle without recompiling.
  * New ``fxb`` command for the full lifecycle: ``build``, ``download``, ``add``,
    ``check``, ``cache ls/rm``, ``show``, and ``inspect``.
  * Serving resolves a bundle in order: explicit ``--fxb`` path, an ``.fxb`` inside the
    model repository, then the local cache (matched by fingerprint and FuriosaIR
    revision).
  * Pipeline-parallelism (``-pp``) options are propagated to FXB-based generators.

* **Model Support**

  * :ref:`Qwen3-VL <furiosa-llm-model-qwen3-vl>` — first vision-language architecture;
    dense transformer paired with a vision encoder, served over the OpenAI-compatible API.

    * ``Qwen3-VL-32B-Instruct``

  * :ref:`gpt-oss <furiosa-llm-model-gpt-oss>` — MoE architecture with MXFP4-quantized
    experts and configurable reasoning effort.

    * ``gpt-oss-20b``, ``gpt-oss-120b``

  * :ref:`Solar-Open <furiosa-llm-model-solar-open>` — NVFP4A16 MoE architecture.

    * ``Solar-Open-100B-NVFP4A16``

  * :ref:`Qwen3 Dense <furiosa-llm-model-qwen3>` — dense transformer architecture with
    static FP8 weights and dynamic FP8 activation quantization.

    * ``Qwen3-4B-FP8``, ``Qwen3-8B-FP8``

  * :ref:`Qwen3 MoE <furiosa-llm-model-qwen3-moe>` — MoE architecture with static FP8
    weights and dynamic FP8 activation quantization.

    * ``Qwen3-30B-A3B-FP8``
    * ``Qwen3-30B-A3B-Instruct-2507-FP8`` (Instruct)
    * ``Qwen3-30B-A3B-Thinking-2507-FP8`` (Thinking)
    * ``Qwen3-Coder-30B-A3B-Instruct-FP8`` (Coder)

  * :ref:`K-EXAONE <furiosa-llm-model-k-exaone>` — multilingual MoE architecture with
    hybrid sliding-window + global attention.

    * ``K-EXAONE-236B-A23B-NVFP4A16``

  * Add tool calling and reasoning parser for Solar-Open.

* **Multimodal Infrastructure** (experimental)

  * **Vision-encoder runtime** runs the vision encoder on the NPU and fuses the resulting
    image embeddings into the text-token sequence, exposed through a new ``generate_mm``
    generator API.
  * **End-to-end OpenAI-compatible serving** of image-and-text requests via ``image_url``
    content parts (remote, base64, or local-file URLs), with per-request guardrails such as
    ``--image-limit-per-prompt`` and ``--allowed-media-domains``.
  * **UUID-based multimodal data reuse** via a server-side processor cache
    (``--mm-processor-cache-gb``): an image tagged with a stable ``uuid`` is preprocessed
    once and reused on follow-up requests.


Engine Core
~~~~~~~~~~~

* **Overlap scheduler** (experimental; off by default, enable via
  ``--enable-overlap-scheduling``) runs one batch ahead so host-side scheduling overhead
  is overlapped with NPU compute, improving throughput and TPOT at a small, bounded TTFT
  cost.

* **Scoring-based DP routing** balances prefix locality and token-footprint load across
  data-parallel replicas, configurable through routing profiles.

* **KV cache improvements** (largely foundational refactors in this release):

  * **Unified Radix Cache** consolidates the global-only and sliding-window prefix-cache
    trees into one, separating tree topology from per-component payloads so new cache
    component types no longer need a tree of their own. A follow-up optimization also
    recovers a long-context (~128K-token) TTFT regression.
  * **KV cache offloading infrastructure** (groundwork): a tiered block-placement model
    (NPU, host, or dual-resident) and a dedicated NPU↔host DMA orchestrator, laying the
    foundation for offloading KV blocks beyond NPU memory in a future release.
  * **More robust NPU memory budgeting**: the shared-DRAM buffer pool now reclaims idle
    buffers, and I/O-buffer space is reserved with fragmentation slack — fixing
    out-of-memory failures on large models under overlap scheduling (e.g. gpt-oss-120b
    long-context).

* **Faster model loading** via asynchronous, parallelized Hugging Face downloads, and by
  skipping redundant weight files in ``snapshot_download``.


API & Usability Improvements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Serving Options**:

  * Added ``--default-chat-template-kwargs`` to set server-wide chat-template defaults
    (merged into every request; request-level values take precedence).
  * Added ``--enable-overlap-scheduling``, ``--data-parallel-routing-policy``, and
    ``--data-parallel-scoring-profile``.

* **Sampling Defaults**:

  * ``eos_token_id`` values from the model's ``generation_config.json`` are now applied
    as default stop tokens for both ``furiosa-llm serve`` and ``LLM``.

* **Pooling / Embedding / Logprob**:

  * Expanded pooling, embedding, and log-probability support in the ``flm`` interface,
    and fixed ``prompt_logprobs`` accumulation.

* **Structured Output**:

  * Bumped **xgrammar to 0.4.0**.
  * The named and required tool-calling paths now set ``additionalProperties: False`` on
    the generated JSON schema, rejecting fields not defined in the tool schema.

* **Admin / Debugging APIs**:

  * Added a **prefix cache reset** endpoint (``/reset_prefix_cache``, available in dev
    mode) for testing and benchmarking.
  * Added a **RawModel** scheduler-bypassing API for debugging.

* **Reliability**:

  * **Graceful DP termination** for clean shutdown of data-parallel deployments.


Platform & Packaging
^^^^^^^^^^^^^^^^^^^^

* **Python 3.14 support**: supported Python versions are now **3.10–3.14**, tracking the
  PyTorch 2.10 support matrix.

* **Broader arm64 (aarch64) support**: the native Python packages
  (``furiosa-native-llm``, ``furiosa-tcc``) now ship ``aarch64`` wheels alongside
  ``x86_64``, the firmware tooling ships an ``arm64`` package, and the cloud-native
  component images are multi-architecture (``linux/amd64`` and ``linux/arm64``). See
  *Released Components* for the per-package breakdown.

* **Rocky Linux 10 support**: the driver and firmware are now packaged for **Rocky Linux
  10** as ``.el10`` RPMs (in addition to the Debian builds), and the cloud-native
  component images are Red Hat OpenShift–certified (see *Cloud-Native Components*).


Driver & Firmware
^^^^^^^^^^^^^^^^^

The RNGD **driver** and **firmware** are both updated to **2026.3.0**.

* **Firmware**:

  * Added support for **board revision 08**.
  * Added an **SMBus default slave address** option and a configurable **PMIC
    over-temperature protection** threshold for board management.
  * **Rocky Linux 10** is now supported (packages ship as ``.el10`` RPMs alongside the
    Debian builds).
  * Bug fixes and general system-stability improvements.

* **Driver**:

  * **Firmware update is now an explicit step.** Installing the firmware image package no
    longer flashes the device automatically; run ``fw_updater`` explicitly afterward to
    apply the new firmware.
  * Bug fixes and general system-stability improvements.


Cloud-Native Components
^^^^^^^^^^^^^^^^^^^^^^^

The cloud-native components are not version-bumped in 2026.3, but their container images
gained broader platform support:

* **Red Hat certification.** The cloud-native component images have passed Red Hat's
  OpenShift preflight certification and security scans, and are now published in the
  `Red Hat Ecosystem Catalog <https://catalog.redhat.com/en/search?q=furiosa>`_ as
  Red Hat UBI–based images.

* **Multi-architecture images.** Component images now ship for both ``linux/amd64`` and
  ``linux/arm64``: ``furiosa-feature-discovery``, ``furiosa-device-plugin``,
  ``furiosa-dra-driver``, ``furiosa-metrics-exporter``, and ``furiosa-npu-operator`` (from
  ``2026.1.1``), and ``furiosa-system-manager`` (from ``2026.2.0``).


.. _Release2026_3_0_BreakingChanges:

🚨 Breaking Changes & Deprecations
----------------------------------

* **/metrics is GET-only**:

  * The ``/metrics`` endpoint now accepts **GET requests only**. Previously a ``POST``
    to ``/metrics`` also returned ``200``; it now returns ``405 Method Not Allowed``.
  * **Migration**: Ensure any monitoring scrapers or health checks query ``/metrics``
    with ``GET``.

* **Firmware upgrade is now a manual step**:

  * Before 2026.3, the firmware updater ran automatically when the firmware image package
    was installed. Starting with 2026.3, installing the image package no longer triggers
    the update — you must run the updater yourself after installing it.
  * **Migration**: After ``apt install`` of the firmware tool and image packages, run
    ``sudo furiosa_rngd_updater_all`` to upgrade all RNGD devices (or
    ``furiosa_rngd_updater -b <BDF> -f <firmware image>`` for a specific device), then
    perform a cold reboot. See the :ref:`UpgradeGuide` for the full procedure.


📦 Released Components
----------------------

Python packages
^^^^^^^^^^^^^^^

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Package name
     - Version
     - Supported architectures
   * - furiosa-native-llm
     - 2026.3.0
     - ``x86_64``, ``aarch64``
   * - furiosa-llm
     - 2026.3.0
     - pure Python (``any``)
   * - furiosa-tcc
     - 2026.3.0
     - ``x86_64``, ``aarch64``
   * - furiosa-models
     - 2026.2.0
     - pure Python (``any``)
   * - furiosa-torch
     - 2026.2.0
     - ``x86_64``, ``aarch64``

APT packages
^^^^^^^^^^^^

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Package name
     - Version
     - Supported architectures
   * - furiosa-driver-rngd
     - 2026.3.0
     - ``all`` (architecture-independent)
   * - furiosa-firmware-image-rngd
     - 2026.3.0
     - ``all`` (architecture-independent)
   * - furiosa-firmware-tools-rngd
     - 2026.3.0
     - ``amd64``, ``arm64``
   * - furiosa-libsmi
     - 2026.1.1
     - ``amd64``, ``arm64``
   * - furiosa-smi
     - 2026.1.1
     - ``amd64``, ``arm64``
   * - furiosa-metrics-exporter
     - 2026.1.1
     - ``amd64``
   * - furiosa-cdi
     - 2026.1.0
     - ``amd64``

YUM packages
^^^^^^^^^^^^

New in 2026.3: the driver and firmware packages are also published as ``.el10``
RPMs for Rocky Linux 10 / RHEL.

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Package name
     - Version
     - Supported architectures
   * - furiosa-driver-rngd
     - 2026.3.0
     - ``noarch`` (architecture-independent)
   * - furiosa-firmware-image-rngd
     - 2026.3.0
     - ``noarch`` (architecture-independent)
   * - furiosa-firmware-tools-rngd
     - 2026.3.0
     - ``x86_64``
   * - furiosa-libsmi
     - 2026.1.1
     - ``x86_64``
   * - furiosa-smi
     - 2026.1.1
     - ``x86_64``
   * - furiosa-metrics-exporter
     - 2026.1.1
     - ``x86_64``
   * - furiosa-cdi
     - 2026.1.0
     - ``x86_64``

Docker images
^^^^^^^^^^^^^

The cloud-native component images on Docker Hub are now multi-architecture and
Red Hat OpenShift–certified.

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Image
     - Tag
     - Supported architectures
   * - furiosaai/furiosa-feature-discovery
     - 2026.1.1
     - ``linux/amd64``, ``linux/arm64``
   * - furiosaai/furiosa-device-plugin
     - 2026.1.1
     - ``linux/amd64``, ``linux/arm64``
   * - furiosaai/furiosa-dra-driver
     - 2026.1.1
     - ``linux/amd64``, ``linux/arm64``
   * - furiosaai/furiosa-metrics-exporter
     - 2026.1.1
     - ``linux/amd64``, ``linux/arm64``
   * - furiosaai/furiosa-npu-operator
     - 2026.1.1
     - ``linux/amd64``, ``linux/arm64``
   * - furiosaai/furiosa-system-manager
     - 2026.2.0
     - ``linux/amd64``, ``linux/arm64``

The Red Hat UBI–based images certified through the OpenShift preflight are
published on ``quay.io``:

.. list-table::
   :widths: 200 50 150
   :header-rows: 1

   * - Image
     - Tag
     - Supported architectures
   * - quay.io/furiosaai/furiosa-feature-discovery
     - 2026.1.0-ubi9
     - ``linux/amd64``
   * - quay.io/furiosaai/furiosa-device-plugin
     - 2026.1.0-ubi9
     - ``linux/amd64``
   * - quay.io/furiosaai/furiosa-metrics-exporter
     - 2026.1.1-ubi9
     - ``linux/amd64``
