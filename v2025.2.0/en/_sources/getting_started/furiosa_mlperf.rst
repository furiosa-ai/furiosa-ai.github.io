.. _Llama-2-70b-chat-hf-FP8-MLPerf: https://furiosa-public-artifacts.s3.ap-northeast-2.amazonaws.com/furiosa-llm-engine-artifacts/v2025.2.0/Llama-2-70b-chat-hf-FP8-MLPerf.tar.zst

.. _GettingStartedFuriosaMLPerf:

***********************************
Running MLPerf™ Inference Benchmark
***********************************

MLPerf™ is a benchmark suite that evaluates the performance of machine
learning (ML) software, hardware, and cloud platforms.
It is commonly used to compare the performance of different systems,
and to assist developers and end users in making decisions about AI systems.

The FuriosaAI software stack provides a ``furiosa-mlperf`` command to run the
MLPerf™ Inference Benchmark more easily.
This section describes how to reproduce the MLPerf™ Inference Benchmark results
using FuriosaAI's NPUs.

.. note::

  ``furiosa-mlperf`` is based on MLPerf™ Inference Benchmark v4.1.
  The only exception is that we replaced the Llama2 benchmark with one using Llama 3.1.


Installing ``furiosa-mlperf``
=============================

Before installing ``furiosa-mlperf``, please ensure you have
the :ref:`prerequisites <InstallingPrerequisites>` installed, as well as
sufficient storage space (100 GB).
Then run the following command:

.. code-block:: sh

  sudo apt install -y furiosa-mlperf

This command installs the ``furiosa-compiler``, ``furiosa-mlperf``,
and ``furiosa-mlperf-resources`` packages.


Running MLPerf™ Inference Benchmark
===================================

Arguments of ``furiosa-mlperf`` command
-------------------------------------------------

The ``furiosa-mlperf`` command provides the following subcommands:

.. code-block::

  FuriosaAI MLPerf™ Inference Benchmark Launcher v2024.2.1

  Usage: furiosa-mlperf <SUBCOMMAND>

  Subcommands:
    bert-offline       Run BERT benchmark with offline scenario
    bert-server        Run BERT benchmark with server scenario
    gpt-j-offline      Run GPT-J benchmark with offline scenario
    gpt-j-server       Run GPT-J benchmark with server scenario
    llama-2-offline    Run Llama 2 70B benchmark with offline scenario
    llama-2-server     Run Llama 2 70B benchmark with server scenario
    llama-3.1-offline  Run Llama 3.1 benchmark with offline scenario
    llama-3.1-server   Run Llama 3.1 benchmark with server scenario
    help               Print this message or the help of the given subcommand(s)

  Options:
    -h, --help     Print help
    -V, --version  Print version


Also, each subcommand has the following arguments:

.. code-block::

  Usage: furiosa-mlperf <SUBCOMMAND> [OPTIONS] <ARTIFACT_PATH> <LOG_DIR>

  Arguments:
    <MODEL_ID_OR_PATH>
            Hugging Face model ID or local path to the model artifact

    <LOG_DIR>
            A directory to store MLPerf™ logs


``<MODEL_ID_OR_PATH>`` can be a Hugging Face model ID or a local path to the model artifact.
``<LOG_DIR>`` is the directory to store the MLPerf™ logs.
Once the ``furiosa-mlperf`` command is executed, it will generate the logs
into the specified directory.
You can check the logs to see the results of the benchmark.

.. tip::

  Each subcommand of furiosa-mlperf provides various options.
  You can use the ``--help`` option to see the detailed options.

  .. code-block:: sh

    furiosa-mlperf bert-offline --help


For example, you will be able to see the MLPerf™ results summary once you run the GPT-J 6B offline scenario.

.. code-block:: sh

  cat gpt-j-offline-result/mlperf_log_summary.txt

  ================================================
  MLPerf Results Summary
  ================================================
  SUT name : GPT-J SUT
  Scenario : Offline
  Mode     : PerformanceOnly
  Samples per second: 12.1491
  Tokens per second (inferred): 838.288
  Result is : VALID
    Min duration satisfied : Yes
    Min queries satisfied : Yes
    Early stopping satisfied: Yes

.. _MLPerfArtifacts:

MLPerf Model Artifacts
----------------------
``furiosa-mlperf`` command requires the model artifacts to run the benchmark.
If you are not familiar with model artifacts, refer to
:ref:`ModelPreparation` for more information.

The following are MLPerf model artifacts that you can use them to run the
MLPerf™ Inference Benchmark with the ``furiosa-mlperf`` command or Furiosa-LLM.

Note that the Bert and GPT-J artifacts are available on the Hugging Face Hub,
but Llama 2 70b provides HTTPS links to download the model artifacts.
You can run Bert and GPT-J benchmarks directly with the Hugging Face Hub model ID.
For Llama 2 70b, you need to download the model artifacts first and then run the benchmark.

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 80 200 160

   * - Model Name
     - Link
     - Description
   * - `Bert <https://github.com/mlcommons/inference/blob/7bf5997/language/bert/README.md>`_
     - `furiosa-ai/bert-large-uncased-INT8-MLPerf <https://huggingface.co/furiosa-ai/bert-large-uncased-INT8-MLPerf>`_
     - Hugging Face Hub
   * - `GPT-J <https://github.com/mlcommons/inference/tree/7bf59976b5f4eb7c5b8f30a88af832e028028446/language/gpt-j>`_
     - `furiosa-ai/gpt-j-6b-FP8-MLPerf <https://huggingface.co/furiosa-ai/gpt-j-6b-FP8-MLPerf>`_
     - Hugging Face Hub
   * - `llama2-70b <https://github.com/mlcommons/inference/tree/7bf59976b5f4eb7c5b8f30a88af832e028028446/language/llama2-70b>`_
     - `Llama-2-70b-chat-hf-FP8-MLPerf`_ (`SHA256 checksum <https://furiosa-public-artifacts.s3.ap-northeast-2.amazonaws.com/furiosa-llm-engine-artifacts/v2025.2.0/Llama-2-70b-chat-hf-FP8-MLPerf.tar.zst.sha256>`_)
     - HTTPS

.. note::

  Llama 2 70b will be available on the Hugging Face Hub since 2025.3 release.

For the Llama 2 70B artifact model, you can download and extract as follows:

.. code-block:: sh

  wget https://furiosa-public-artifacts.s3.ap-northeast-2.amazonaws.com/furiosa-llm-engine-artifacts/v2025.2.0/Llama-2-70b-chat-hf-FP8-MLPerf.tar.zst

  # Validate the checksum
  shasum -c Llama-2-70b-chat-hf-FP8-MLPerf.tar.zst

  # Extract the model artifacts
  tar --zstd -xvf Llama-2-70b-chat-hf-FP8-MLPerf.tar.zst

Offline vs Server Scenario
------------------------------------
The MLPerf™ benchmark provides two scenarios for data center systems: offline and server.
The offline scenario is designed to measure the system's maximum throughput.
The server scenario measures both throughput and tail latencies, ensuring that 99%
of the requests are served within a specified latency threshold.
Depending on your target use case, you can select the appropriate scenario.
For more details on benchmark scenarios, please refer to
`MLPerf™ Inference Rules - 3. Scenarios <https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc#scenarios>`_.

MLPerf™ Configuration
---------------------
You can configure the MLPerf™ benchmark by using the ``--user-conf`` option to
specify a custom configuration file.
For example:

.. code-block::

  cat << EOF > user.conf
  bert.Server.target_qps = 1900
  EOF

.. code-block::

  furiosa-mlperf bert-server furiosa-ai/bert-large-uncased-INT8-MLPerf ./bert-server-result --user-conf ./user.conf


.. tip::

  More information about MLPerf™ configuration files and examples can be found at
  `mlcommons/inference/mlperf.conf <https://github.com/mlcommons/inference/blob/v4.1/mlperf.conf>`_.


Monitoring a running benchmark
------------------------------

Some benchmarks take a long time to complete.
For example, Bert Large and GPT-J 6B take about 10 mins and 20 mins,
respectively, with 1 RNGD card.
However, Llama 3.1 70B with 4 RNGD takes about 2.5 hours.
Therefore, it is important to monitor the running benchmark.

You can check the status of the FuriosaAI NPUs using the :ref:`FuriosaSMICLI`
command as follows:

.. code-block:: sh

  furiosa-smi status


Example output:

.. code-block::

  +------+--------+----------------+------------------+
  | Arch | Device | Cores          | Core Utilization |
  +------+--------+----------------+------------------+
  |      |        | 0 (occupied),  | Core 0: 99.47%,  |
  |      |        | 1 (occupied),  | Core 1: 99.47%,  |
  |      |        | 2 (occupied),  | Core 2: 99.47%,  |
  | rngd | npu0   | 3 (occupied),  | Core 3: 99.47%,  |
  |      |        | 4 (occupied),  | Core 4: 99.45%,  |
  |      |        | 5 (occupied),  | Core 5: 99.45%,  |
  |      |        | 6 (occupied),  | Core 6: 99.45%,  |
  |      |        | 7 (occupied)   | Core 7: 99.45%   |
  +------+--------+----------------+------------------+


Running ``furiosa-mlperf`` in a Container Environment
=====================================================
FuriosaAI provides a containerized version of the ``furiosa-mlperf`` command.
The ``furiosa-mlperf`` container image allows you to run ``furiosa-mlperf``
effortlessly.

.. note::

  The container version still requires the :ref:`InstallingPrerequisites` step
  to install the driver, firmware, and PERT on the host system.


To run the ``furiosa-mlperf`` container for the GPT-J 6B offline scenario,
use the following command:

.. code-block:: sh


  docker pull furiosaai/furiosa-mlperf:latest

  # Please replace the model id by one of other MLPerf models you want to run.
  MODEL_ID=furiosa-ai/gpt-j-6b-FP8-MLPerf

  docker run -it --rm \
    --device /dev/rngd:/dev/rngd \
    --security-opt seccomp=unconfined \
    --env HF_TOKEN=$HF_TOKEN \
    -v $HOME/.cache/huggingface:/root/.cache/huggingface \
    furiosaai/furiosa-mlperf:latest \
    gpt-j-offline --test-mode performance-only $MODEL_ID /result --devices "npu:0"


Benchmark Examples
==================

BERT Large
----------

The BERT benchmark exhibits good performance with a single RNGD card.
Use the following command to run the offline scenario:

.. code-block:: sh

  furiosa-mlperf bert-offline furiosa-ai/bert-large-uncased-INT8-MLPerf ./bert-offline-result \
    --devices "npu:0"


To run the BERT-large server scenario, you need to specify the target QPS
in a user config file to get the expected performance:

.. code-block:: sh

  cat << EOF > user.conf
  bert.Server.target_qps = 1900
  EOF

.. note::

  The default target QPS (queries per second) of MLPerf™ is ``1``.
  This setting does not allow devices to show their full performance with
  lightweight workloads such as BERT.

Then, you can run the benchmark with a custom configuration as follows:

.. code-block:: sh

  furiosa-mlperf bert-server furiosa-ai/bert-large-uncased-INT8-MLPerf ./bert-server-result \
    --devices "npu:0" --user-conf ./user.conf


.. tip::

  You can experience RNGD cards if you specify more device as the following.

  .. code-block:: sh

    furiosa-mlperf bert-offline furiosa-ai/bert-large-uncased-INT8-MLPerf ./bert-offline-result \
      --devices "npu:0,npu:1" --user-conf ./user.conf


GPT-J 6B benchmark
------------------

The GPT-J benchmark also runs on a single RNGD card.

The following commands run the GPT-J 6B serving and offline inference
benchmarks, respectively:

.. code-block:: sh

  furiosa-mlperf gpt-j-server furiosa-ai/gpt-j-6b-FP8-MLPerf ./gpt-j-server-result

  furiosa-mlperf gpt-j-offline furiosa-ai/gpt-j-6b-FP8-MLPerf ./gpt-j-offline-result


Llama 2 70B benchmark
-----------------------

Llama 2 70B requires at least 2 RNGD cards. For the best performance, you will need at least 4 RNGD cards.
To run the Llama 2 70B benchmark, you need to download the model artifacts
by following the instructions in :ref:`MLPerfArtifacts`.

Then, you can run the Llama 2 70B serving and offline inference
benchmarks respectively as follows:

.. code-block:: sh

  furiosa-mlperf llama-2-server ./Llama-2-70b-chat-hf-FP8-MLPerf ./llama-2-server-result

  furiosa-mlperf llama-2-offline ./Llama-2-70b-chat-hf-FP8-MLPerf ./llama-2-offline-result
