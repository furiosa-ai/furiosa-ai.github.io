.. _FXB:

****************************************************
Furiosa Executable Bundles (FXB)
****************************************************

A model served by Furiosa-LLM consists of two parts: the model assets in Hugging Face
format and the compiled program that runs those assets on the Furiosa NPU. The model assets
provide the configuration, tokenizer, and weights, while a **Furiosa Executable Bundle
(FXB)** provides the compiled kernels. A runnable deployment needs both parts.

Furiosa-LLM needs an FXB because the code executed on the NPU is compiled ahead of time for
the model architecture and execution configuration. The resulting ``.fxb`` file is a single
archive containing the compiled binaries and the metadata needed to load and run them.

Keeping the compiled program separate from the model assets has several benefits:

* A model can start without compiling kernels on the serving host.
* The FXB can be copied, cached, or published with a model.
* Compatible models can reuse the same FXB when only their weights change.

An FXB does not contain the model weights and does not replace the Hugging Face model
directory. Furiosa-LLM still reads the configuration, tokenizer, and weights from that
directory or repository.

This page follows the model lifecycle from building an FXB to supplying it with the model
assets and reusing it with compatible models. Detailed command behavior, bundle metadata,
and compatibility rules are described in the later sections.

.. tip::

  FuriosaAI publishes a set of pre-compiled FXB bundles for popular models on the
  `Hugging Face Hub 🤗 - FuriosaAI organization <https://huggingface.co/furiosa-ai>`_.
  You can download one of these and serve it for any compatible model with the workflow below.


Installation
============

FXB includes the ``.fxb`` bundle format and the framework for building, distributing, and
loading bundles. Furiosa-LLM exposes this lifecycle through the ``fxb`` CLI.

The ``fxb`` CLI ships with the Furiosa-LLM package, so there is nothing extra to install.
If you have not installed Furiosa-LLM yet, follow :ref:`InstallingFuriosaLLM`:

.. only:: stable

  .. code-block:: sh

    pip install --upgrade pip setuptools wheel uv
    uv pip install --upgrade --torch-backend=auto furiosa-llm

.. only:: prerelease

  .. parsed-literal::

    pip install --upgrade pip setuptools wheel uv
    uv pip install --prerelease=allow --torch-backend=auto furiosa-llm==\ |release|

Verify that the CLI is available:

.. code-block::

  fxb --help

The ``fxb`` CLI covers the whole FXB lifecycle: building, downloading, caching, compatibility
checking, and inspection. The cache and inspection subcommands use the same local cache that
``furiosa-llm serve`` consults at serving time.


.. _BuildingFXB:

Building an FXB for a model
===========================

Use ``fxb build`` to compile a model into an ``.fxb`` bundle:

.. code-block::

  fxb build Qwen/Qwen3-8B-FP8 qwen3-8b-fp8

The first argument is a Hugging Face model ID or a local path (a path must start with ``.`` or
``/``). The second is the output path; the ``.fxb`` extension is appended automatically when the
path has none, so the command above writes ``qwen3-8b-fp8.fxb``.

Preview the resolved configuration and bucket plan without compiling using ``--dry-run``:

.. code-block::

  fxb build Qwen/Qwen3-8B-FP8 qwen3-8b-fp8 --dry-run

Common options:

* ``-tp/--tensor-parallel-size`` — number of PEs per tensor-parallel group (default: the model
  preset).
* ``-pp/--pipeline-parallel-size`` — pipeline-parallel size (default: ``1``).
* ``--max-model-len`` — override the model's maximum context length for bucket selection. This
  shapes what gets compiled into the bundle; ``furiosa-llm serve`` takes an option of the same
  name that only caps admission at serve time (see :ref:`ServeMaxModelLen`).
* ``-O/--optim-level`` — bucket-filtering optimization level: ``O0`` (minimal), ``O1`` (half),
  ``O2`` (quarter), ``O3`` (full, the default).
* ``--convert`` — task override for models with an ambiguous architecture: ``embed`` rewrites
  ``*ForCausalLM`` → ``*Model``; ``score``/``classify`` → ``*ForSequenceClassification``
  (default: ``auto``, i.e. keep the architecture from ``config.json``).

After building, register the bundle in the cache so ``fxb check`` and ``furiosa-llm serve`` can
find it:

.. code-block::

  fxb add ./qwen3-8b-fp8.fxb

For production build recommendations, see :ref:`FXBBestPractices`.

Models runnable with Furiosa-LLM
================================

Furiosa-LLM loads the model assets and the FXB together. The model assets must follow the
Hugging Face Hub directory format and include files such as ``config.json``, tokenizer files,
and model weights. A model can be distributed or invoked in either of the following forms.

Model directory containing an FXB
---------------------------------

Place a compatible ``.fxb`` file in the Hugging Face model directory alongside the model
configuration, tokenizer, and weights:

.. code-block:: text

  my-model/
    config.json
    tokenizer.json
    model-00001-of-00002.safetensors
    model-00002-of-00002.safetensors
    model.safetensors.index.json
    my-model.fxb

The directory can be stored locally or published as a repository on the Hugging Face Hub.
Furiosa-LLM discovers the bundled FXB automatically:

.. code-block:: console

  furiosa-llm serve ./my-model

.. code-block:: console

  furiosa-llm serve furiosa-ai/Qwen3-8B-FP8

This form is convenient for distributing a self-contained model repository that is ready to
serve.

Specifying an FXB directly
--------------------------

Keep the model assets and FXB in separate locations and select the bundle with ``--fxb``:

.. code-block:: console

  furiosa-llm serve Qwen/Qwen3-8B-FP8 --fxb ./qwen3-8b-fp8.fxb

The first argument still identifies the Hugging Face model repository or local directory from
which Furiosa-LLM loads the configuration, tokenizer, and weights. ``--fxb`` selects only the
compiled bundle. This form is useful for testing a newly built FXB, pinning a specific bundle,
or reusing one FXB with multiple compatible models.

The Python API provides the same separation:

.. code-block:: python

  from furiosa_llm import LLM

  llm = LLM("Qwen/Qwen3-8B-FP8", fxb="./qwen3-8b-fp8.fxb")

For automatic discovery through the local FXB cache, see :ref:`ServingFromCachedFXB`.

FXB compatibility
=================

FXB compatibility is determined by an **architecture fingerprint**. The fingerprint includes
the model architecture and the ``config.json`` fields that affect kernel generation, such as
hidden dimensions, attention configuration, mixture-of-experts settings, and quantization
format. Model weights themselves are not part of the fingerprint.

As a result, an existing FXB can usually be reused when:

* a model is fine-tuned without changing its kernel-affecting configuration;
* weights are updated while the architecture and fingerprint fields remain the same; or
* another model repository has the same architecture fingerprint.

A new FXB is required when a change affects the compiled kernels. Examples include changing
the architecture, model dimensions, attention or expert configuration, quantization format,
or another fingerprint field. Compatibility matching is strict; a fine-tuned model is not
automatically compatible if its configuration also changes.

Architecture compatibility is not the only deployment constraint. The FXB must also be
suitable for the installed Furiosa software version and the intended tensor-parallel and
context-length configuration. See :ref:`FXBFingerprint` for the exact matching rules and
compatibility limitations.

.. _ServingFromCachedFXB:

Serving a model with a compatible cached FXB
============================================
A model's own Hugging Face repository may not ship an ``.fxb`` of its own. When that happens,
Furiosa-LLM can still serve it by reusing a **fingerprint-compatible** bundle from your local
cache (see :ref:`FXBFingerprint` for how compatibility is determined). This is especially useful
when the model weights are updated, or when you serve a fine-tuned model or a variation of a
supported model: as long as the architecture fingerprint is unchanged, you can reuse an existing
FXB instead of compiling a new one for every weight update or variant. The workflow is:

#. Get a compatible FXB into the cache — either download one from the Hub, or add one you already
   have on disk.
#. Check that the cached bundle is compatible with the model you want to serve.
#. Serve the model — Furiosa-LLM finds and uses the cached bundle automatically.

The example below serves ``Qwen/Qwen3-8B-FP8`` (which does not ship an FXB) by reusing the
``furiosa-ai/Qwen3-8B-FP8`` bundle, which has the same fingerprint.

Step 1 — Get a compatible FXB into the cache
--------------------------------------------
Download a published bundle from the Hub:

.. code-block::

  fxb download furiosa-ai/Qwen3-8B-FP8

This fetches every ``.fxb`` in the repository into the cache and prints where each was stored.
Restrict the download to a single file with ``--file``, pick a branch/tag/commit with
``--revision``, and re-fetch a cached bundle with ``--force``.

Alternatively, register an ``.fxb`` you already have on disk — for example one you just built (see
:ref:`BuildingFXB`) or copied from another machine:

.. code-block::

  fxb add ./qwen3-8b-fp8.fxb

``add`` copies the file into the cache and standardizes its name from the bundle's manifest, so a
bare ``out.fxb`` becomes consistent with downloaded bundles.

Step 2 — Check compatibility
----------------------------
First, confirm the bundle landed in the cache with ``fxb cache ls``:

.. code-block::

  fxb cache ls

.. code-block:: text

  REPO ID                  ARCH               SIZE     FuriosaIR           FURIOSA-LLM         FXB FILE
  -----------------------  -----------------  -------  ------------------  ------------------  -----------------------------------
  furiosa-ai/Qwen3-8B-FP8  Qwen3ForCausalLM   4.1 GiB  2026.3.0 (a1b2c3)   2026.3.0 (d4e5f6)   Qwen3-8B-FP8-<npu>-<rt>-<ts>.fxb

  1 file(s), 4.1 GiB

A bundle added with ``fxb add`` appears here under the ``local`` repo id. Now confirm it is
compatible with the target model. ``check`` reads the target repository's model configuration and
lists every cached FXB whose fingerprint matches:

.. code-block::

  fxb check Qwen/Qwen3-8B-FP8

.. code-block:: text

  Target: Qwen/Qwen3-8B-FP8  (Qwen3ForCausalLM)
    hidden_size=4096 intermediate_size=12288 num_attention_heads=32 vocab_size=151936 quant_method=fp8

  Compatible cached FXB (1):
       REPO ID                  SIZE       FuriosaIR              FXB FILE
  ---  -----------------------  ---------  ---------------------  -----------------------------------------------
  ✔    furiosa-ai/Qwen3-8B-FP8  4.1 GiB    2026.3.0 (a1b2c3) — match  Qwen3-8B-FP8-<npu>-<rt>-<ts>.fxb

  Recommended: /root/.cache/furiosa/llm/fxb/models--furiosa-ai--Qwen3-8B-FP8/snapshots/<sha>/Qwen3-8B-FP8-<npu>-<rt>-<ts>.fxb

The bundle marked ``✔`` is the **recommended** one. The ``FuriosaIR`` column is annotated with the
build-match status:

* ``— match`` — the bundle's FuriosaIR (compiler) revision matches your running build. This bundle
  is served automatically.
* ``— stale`` — the bundle is fingerprint-compatible but was built with a *different* FuriosaIR
  revision. It is **not** served automatically (see the next step).

If no compatible bundle is cached, ``check`` prints a message and exits with a non-zero status.

Step 3 — Serve the model
------------------------
Serve the target model as usual. Furiosa-LLM resolves the FXB to run in this order:

#. an explicit ``--fxb <path>`` (used as-is);
#. an ``.fxb`` shipped inside the model's own repository;
#. the **local cache** — the recommended compatible bundle, but only if its FuriosaIR revision
   matches the running build.

.. code-block::

  furiosa-llm serve Qwen/Qwen3-8B-FP8

When the cache fallback is used, Furiosa-LLM logs an ``INFO`` line naming the cached bundle it
picked. The same resolution applies to the Python ``LLM(...)`` API, since both go through the same
code path.

.. warning::

  If the cache holds only **stale** (FuriosaIR-revision-mismatched) compatible bundles, serving
  fails rather than silently loading a bundle built for a different compiler revision. The error
  tells you to choose one explicitly:

  .. code-block::

    furiosa-llm serve Qwen/Qwen3-8B-FP8 --fxb /path/to/stale-bundle.fxb

.. note::

  The cache lives at ``~/.cache/furiosa/llm/fxb`` by default
  (``$XDG_CACHE_HOME/furiosa/llm/fxb`` when ``XDG_CACHE_HOME`` is set). Every ``fxb`` cache
  subcommand accepts ``--cache-dir`` to use a different location. The cache is only consulted for a
  Hugging Face repo id — a local model path bypasses it.

.. tip::

  Some repositories are gated. Authorize the Hugging Face Hub before downloading or checking them,
  the same way as in :ref:`AuthorizingHuggingFaceHub`:

  .. code-block::

    hf auth login --token $HF_TOKEN


.. _FXBCLIReference:

Command reference
=================
Every subcommand that touches the cache accepts ``--cache-dir`` (default:
``~/.cache/furiosa/llm/fxb``).

``fxb build``
-------------
Build an FXB artifact from a model.

.. code-block::

  fxb build <model> <output_path> [options]

* ``model`` — a Hugging Face model id or a local path (a path starts with ``.`` or ``/``).
* ``output_path`` — where to write the ``.fxb`` (``.fxb`` appended when missing).
* ``-tp/--tensor-parallel-size N`` — PEs per tensor-parallel group (default: model preset).
* ``-pp/--pipeline-parallel-size N`` — pipeline-parallel size (default: ``1``).
* ``--max-model-len N`` — maximum context length used for bucket filtering.
* ``-O/--optim-level {O0,O1,O2,O3}`` — bucket-filtering level (default: ``O3``).
* ``--convert {auto,embed,score,classify}`` — task override (default: ``auto``).
* ``--dry-run`` — resolve config and buckets and print the build summary without compiling.
* ``--build-report`` — print a per-kernel compilation timing report after the build.
* ``--concurrency N`` — maximum kernel compilations to run in parallel (default: ``1``).
* ``--dump-tc DIR`` — dump the generated TC/DSL files to ``DIR`` for debugging.

Advanced options (leave unset unless you know you need them):

* ``--no-fast-fail`` — keep building the remaining kernels after a compilation failure.
* ``--skip-bucket-validation`` — skip the prefill/decode attention-size check (bring-up only).
* ``--enable-collective-dq`` — enable collective dynamic quantization; only selected models support it.
* ``--tokenwise-buckets 1,2,4,8`` / ``--attention-buckets '(1,128,128),(1,1024,1)'`` — override the bucket plan.

On success (non-dry-run) it prints ``Artifact Build Completed``.

``fxb download``
----------------
Download an FXB bundle from a Hugging Face repository into the cache.

.. code-block::

  fxb download <repo_id> [--file F] [--revision R] [--force] [--cache-dir D]

* ``repo_id`` — the Hugging Face repository id.
* ``--file F`` — restrict the download to a single ``.fxb`` by filename.
* ``--revision R`` — repository revision (branch, tag, or commit).
* ``--force`` — re-download even if the bundle is already cached.

``fxb add``
-----------
Add local ``.fxb`` files to the cache. Like ``download``, it fills the cache — from files on disk
rather than from the Hub. The stored filename is standardized from each bundle's manifest, and the
files are copied (the originals are left in place). Multiple paths are accepted, so a shell glob such
as ``fxb add *.fxb`` adds every matching bundle; an invalid bundle is reported without aborting the
rest, and the command exits non-zero if any failed.

.. code-block::

  fxb add <path>... [--cache-dir D]

``fxb check``
-------------
Find cached FXBs compatible with a Hugging Face repository's model config (see
:ref:`FXBFingerprint` for how compatibility is determined).

.. code-block::

  fxb check <repo_id> [--cache-dir D]

Prints the target fingerprint, a table of compatible cached bundles (recommended one marked ``✔``,
with each bundle's FuriosaIR revision annotated ``— match`` or ``— stale``), and the recommended
path. Exits with a non-zero status when no compatible bundle is cached.

``fxb cache``
-------------
Inspect and prune what is already cached. Adding to the cache lives at the top level
(``fxb download`` / ``fxb add``); ``fxb cache`` is purely the inventory-management group, with the
subcommands ``ls``, ``rm``, and ``prune``.

``fxb cache ls``
----------------
List cached FXB bundles.

.. code-block::

  fxb cache ls [-q/--quiet] [--cache-dir D]

* ``-q/--quiet`` — print only the ``.fxb`` paths (useful for scripting).

.. code-block:: text

  REPO ID                  ARCH               SIZE     FuriosaIR           FURIOSA-LLM         FXB FILE
  -----------------------  -----------------  -------  ------------------  ------------------  -----------------------------------
  furiosa-ai/Qwen3-8B-FP8  Qwen3ForCausalLM   4.1 GiB  2026.3.0 (a1b2c3)   2026.3.0 (d4e5f6)   Qwen3-8B-FP8-<npu>-<rt>-<ts>.fxb

  1 file(s), 4.1 GiB

``fxb cache rm``
----------------
Remove cached FXB bundles. Requires at least one selector (a repo id or ``.fxb`` filename) or
``--all``. Without ``-y/--yes`` it shows what will be removed and prompts for confirmation.

.. code-block::

  fxb cache rm [<selector>...] [--all] [--dry-run] [-y/--yes] [--cache-dir D]

* ``selectors`` — repo ids or ``.fxb`` filenames to remove.
* ``--all`` — remove everything in the cache.
* ``--dry-run`` — show what would be removed without deleting anything.
* ``-y/--yes`` — do not prompt for confirmation (required when stdin is not interactive).

``fxb cache prune``
-------------------
Remove cached bundles by FuriosaIR version. By default it removes bundles that are *stale* for the
running build — those whose FuriosaIR (compiler) revision does not match the running build, or whose
revision is unknown — so they would never be served automatically anyway. With ``--older-than`` it
instead removes bundles whose FuriosaIR version is below a given semantic version. Like ``rm``, it
previews and prompts unless ``-y/--yes`` is given.

.. code-block::

  fxb cache prune [--older-than IR_VERSION] [--dry-run] [-y/--yes] [--cache-dir D]

* ``--older-than IR_VERSION`` — remove bundles whose FuriosaIR version is older than ``IR_VERSION``,
  e.g. ``2026.3.0``, ``<2026.3.0``, or ``<=2026.2.0`` (a bare version means ``<``). Bundles with no
  recorded FuriosaIR version are kept. When omitted, prune targets the running build's stale bundles.
* ``--dry-run`` — show what would be removed without deleting anything.
* ``-y/--yes`` — do not prompt for confirmation (required when stdin is not interactive).

``fxb show``
------------
Show bundle metadata: general info (format version, UUID, ``created_at``, tool versions), the model
fingerprint, the parallel configuration, and the kernel/bucket summary.

.. code-block::

  fxb show <path>

``path`` may be an ``.fxb`` file or an extracted artifact directory.

.. code-block:: text

  ──────────────────────────────────────────────────────────────────────────
  FXB Bundle — qwen3-8b-fp8.fxb
  ──────────────────────────────────────────────────────────────────────────

  ── General ──────────────────────────────────────────────────────────────
    - format_version       2
    - uuid                 fd0348f5-6361-4586-8013-4d4ba4f70171
    - created_at           2026-06-16T19:49:52.720726644+00:00
    - furiosa_llm          2026.3.0-dev (7a9d13150)
    - furiosa_compiler     0.11.0-dev (795da8b53a)

  ── Model ────────────────────────────────────────────────────────────────
    - architecture         Qwen3ForCausalLM
    - hub_repo_id          Qwen/Qwen3-8B-FP8
    - hidden_size          4096
    - intermediate_size    12288
    - num_attention_heads  32
    - vocab_size           151936
    - num_key_value_heads  8
    - head_dim             128
    - quant_method         fp8

  ── Parallelism ──────────────────────────────────────────────────────────
    - tensor_parallel_size 8
    - pipeline_parallel_size 1

  ── Kernels (151 entries, 4 kernels) ─────────────────────────────────────
    - mid_tokenwise
    - first_tokenwise
    - last_tokenwise_with_lm_head
    - full_attention

  ── Buckets (133) ────────────────────────────────────────────────────────
    - tokenwise            [1, 4, 8, 16, 32, 64, 128, 256, 1024]
    - prefill (7):
        - batch=1    attn=128      input_ids=128
        ...

``fxb inspect``
---------------
Inspect the per-kernel input/output signatures recorded in the bundle. For each kernel and bucket,
it prints the size, shape, and dtype of every input and output tensor.

.. code-block::

  fxb inspect <path>

``path`` may be an ``.fxb`` file or an extracted artifact directory. The output lists one block per
kernel/bucket; the excerpt below shows the first entry:

.. code-block:: text

  first_tokenwise (tw1):
    Inputs:
      [0] size=4         shape=[Broadcast=1]|[0_1=1:1]  dtype=raw_i32
      [1] size=4         shape=[Broadcast=1]|[0_1=1:1]  dtype=raw_i32
      [2] size=16777216  shape=[Broadcast=1]|[0_1=8192:1024, 1_1=1:1024, 2_1=8:128, 3_1=128:1]  dtype=bf16
      [3] size=16777216  shape=[Broadcast=1]|[0_1=8192:1024, 1_1=1:1024, 2_1=8:128, 3_1=128:1]  dtype=bf16
      [4] size=4         shape=[Broadcast=1]|[0_1=1:1, 1_1=1:1]  dtype=raw_i32
      [5] size=1244659712  shape=[Broadcast=1]|[0_1=151936:4096, 1_1=4096:1]  dtype=bf16
      ...
    Outputs:
      [0] size=8192      shape=[Broadcast=1]|[0_1=1:4096, 1_1=4096:1]  dtype=bf16
      [1] size=8192      shape=[Broadcast=1]|[0_1=1:4096, 1_1=4096:1]  dtype=bf16


.. _FXBFingerprint:

Fingerprint and Compatibility
=============================
The key property of an FXB is its **architecture fingerprint** — the value Furiosa-LLM uses to
decide whether a bundle is compatible with a given model. The manifest records the model's
architecture and the configuration fields that determine the compiled kernels, and a bundle is
considered compatible with a model when their fingerprints match. A single FXB is therefore
reusable across *any* Hugging Face model that shares the same fingerprint, not just the one it was
built from — which is what makes it possible to serve a model whose own repository ships no ``.fxb``
by reusing a compatible bundle from your local cache.

The fingerprint is built from the model architecture plus the ``config.json`` fields that affect
kernel generation — the dimensions, attention and mixture-of-experts settings, and quantization
that **furiosa-kernels reads at build time**. Representative examples are ``hidden_size``,
``num_attention_heads``, ``sliding_window``, the expert counts for MoE models, and the quantization
``format``. Two repositories that differ in any of these compile to different kernels and must not
share an FXB; fields used only at load time are not part of the fingerprint.

Matching is **strict**: two models are compatible only if their architecture and all
fingerprint fields are equal. Use ``fxb check`` to verify a match before reusing a bundle.

.. note::

  The fingerprint-based compatibility matching is **experimental**. The exact set of fields that
  make up the fingerprint, and how they are compared, may change in future releases. Always verify
  a match with ``fxb check`` before relying on a cached bundle for a different model.


.. _FXBBestPractices:

Best Practices
==============
``fxb build`` trades build time against the runtime coverage and performance of the resulting
bundle. The right options depend on what you are building for. The scenarios below cover the common
cases; FuriosaAI's own per-model production configurations live in the build matrix at
``.github/fxb-artifacts.yaml``.

Quick test build
----------------
When you just want a runnable bundle as fast as possible — bring-up, a smoke test, or checking that
a model compiles at all — minimize the number of kernels that get compiled and parallelize the
build:

* ``-O O0`` — the minimal bucket set, so far fewer kernels are compiled. This is the single biggest
  lever on build time.
* ``--max-model-len`` — cap the context length to something small so fewer and smaller buckets are
  generated.
* ``--concurrency`` — raise above the default of ``1`` to compile kernels in parallel and use the
  available cores on the build host.
* ``--dry-run`` — resolve the config and bucket plan and print the build summary *without*
  compiling, so you can confirm the plan before spending any compile time.

.. code-block::

  # Preview the plan first, then build a minimal bundle quickly
  fxb build Qwen/Qwen3-8B-FP8 qwen3-8b-test.fxb --max-model-len 4096 --dry-run
  fxb build Qwen/Qwen3-8B-FP8 qwen3-8b-test.fxb -O O0 --max-model-len 4096 --concurrency 8

A bundle built with ``-O O0`` runs, but it only covers a minimal set of buckets; expect reduced
performance and coverage compared with a full build. Do not serve it in production.

Production build
----------------
For the bundle you actually serve, favor full bucket coverage and a configuration that matches the
deployment, accepting a longer build:

* ``-O O3`` — the full bucket set (this is the default; set it explicitly to make the intent clear).
* ``-tp/--tensor-parallel-size`` — match the serving topology (8 PEs per card — e.g. ``8`` for a
  single card, ``32`` for four).
* ``--max-model-len`` — set to the maximum context length you actually serve, so buckets are sized
  for the real workload rather than over-built.
* ``--concurrency`` — set high to saturate the build host and shorten the (longer) full build.
* ``--build-report`` — print per-kernel compilation timing to spot unexpectedly slow kernels.

.. code-block::

  fxb build openai/gpt-oss-120b gpt-oss-120b.fxb \
      -O O3 -tp 32 --max-model-len 32768 --concurrency 24 --build-report

Deployment topology
-------------------

**Tensor parallelism is fixed at build time**, while **pipeline and data parallelism are set when
you serve.** A bundle's ``-tp/--tensor-parallel-size`` shapes the compiled kernels (each kernel is
sharded for that tensor-parallel degree), so it is part of the FXB and cannot be changed
afterward — build with the ``-tp`` you intend to deploy. Pipeline and data parallelism, by
contrast, replicate and stage the already-compiled bundle across more PEs without recompiling, so
they are chosen at ``furiosa-llm serve`` time via ``-pp/--pipeline-parallel-size`` and
``-dp/--data-parallel-size``:

.. code-block::

  # Built once for tensor-parallel size 8; deployed with pipeline and data parallelism
  furiosa-llm serve Qwen/Qwen3-8B-FP8 --fxb ./qwen3-8b-fp8.fxb -pp 2 -dp 4
