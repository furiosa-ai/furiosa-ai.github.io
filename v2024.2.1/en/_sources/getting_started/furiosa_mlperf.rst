.. _GettingStartedFuriosaMLPerf:

**********************************************
Running MLPerf™ Inference Benchmark
**********************************************

MLPerf™ is a benchmark suite that evaluates the performance of machine learning (ML) software, hardware, and
cloud platforms. It is generally used to compare the performance of different systems,
and to help developers and end users make decisions about AI systems.

FuriosaAI Software Stack provides ``furiosa-mlperf`` command to run easily the MLPerf Inference Benchmark.
This section describes how to reproduce the MLPerf™ Inference Benchmark using the FuriosaAI Software Stack.

.. note::

  ``furiosa-mlperf`` is based on MLPerf™ Inference Benchmark v4.1.

  The only exception is that we replaced the Llama2 benchmark with one using Llama 3.1.


Installing ``furiosa-mlperf`` command
=========================================

To install the ``furiosa-mlperf`` command, you need to install ``furiosa-mlperf`` as following:

The minimum requirements for ``furiosa-mlperf`` are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* Configuring the APT server and installing device drivers (:ref:`AptSetup`)
* About 100GB storage space (only for the Llama 3.1 70B)

Then, please install the ``furiosa-mlperf`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-mlperf

This command installs packages ``furiosa-compiler``, ``furiosa-mlperf`` and ``furiosa-mlperf-resources``.

Running MLPerf Inference Benchmark
====================================================

Arguments of ``furiosa-mlperf`` command
-------------------------------------------------

The ``furiosa-mlperf`` command provides the following subcommands:

.. code-block::

  FuriosaAI MLPerf Inference Benchmark Launcher v2024.2.1

  Usage: furiosa-mlperf <SUBCOMMAND>

  Subcommands:
    bert-offline       Run BERT benchmark with offline scenario
    bert-server        Run BERT benchmark with server scenario
    gpt-j-offline      Run GPT-J benchmark with offline scenario
    gpt-j-server       Run GPT-J benchmark with server scenario
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
    <LLM_ENGINE_ARTIFACTS>
            A directory to cache LLM engine artifacts in

    <LOG_DIR>
            A directory to store MLPerf logs


``<ARTIFACT_PATH>`` is the path to the model artifacts, and ``<LOG_DIR>`` is the directory to store the MLPerf logs.
Once mlperf command is executed, it will generate the logs into a specified directory.
You can check the logs to see the results of the benchmark.

.. tip::

  The subcommand of furiosa-mlperf provides various options. So, you can use the ``--help`` option to see the detailed options.

  .. code-block:: sh

    furiosa-mlperf bert-offline --help


For example, you will be able to see the MLPerf results summary once you run the GPT-J 6B offline scenario.

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


MLPerf Configuration
-------------------------------------------------
You can configure the MLPerf benchmark by specifying ``--user-conf`` option with your configuration file.


.. code-block::

  cat << EOF > user.conf
  bert.Server.target_qps = 1900
  EOF

.. code-block::

  furiosa-mlperf bert-server ./mlperf-bert-large ./bert-server-result --user-conf ./user.conf


.. tip::

  You can find more information about the MLPerf configuration and examples at
  `mlcommons/inference/mlperf.conf <https://github.com/mlcommons/inference/blob/v4.1/mlperf.conf>`_.


Monitoring a running benchmark
-------------------------------------------------

Depending on model and scenario, the benchmark may take a long time to complete.
For example, Bert Large and GPT-J 6B take about 10 mins and 20 mins with 1 RNGD card respectively.
Llama 3.1 70B with 4 RNGD takes about 2 and half hours. So, we need a way to monitor the running benchmark.

You can check the status of the FuriosaAI NPUs using the :ref:`FuriosaSMICLI` command as following:

.. code-block:: sh

  furiosa-smi status


Output:

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


Running ``furiosa-mlperf`` in container environment
====================================================
FuriosaAI provides a containerized version of the ``furiosa-mlperf`` command.
With ``furiosa-mlperf`` container image, you can readily run ``furiosa-mlperf``.

.. note::

  The container version still requires :ref:`InstallingPrerequisites` step
  to install driver, firmware, pert on the host system.


To run the ``furiosa-mlperf`` container for GPT-J 6B offline scenario,
you can use the following command:

.. code-block:: sh

  # Please replace the path with the actual path to the model artifacts.
  ARTFIACTS_DIR=./mlperf-gpt-j-6b

  docker run -it --rm --privileged \
    -v $ARTFIACTS_DIR/:/model \
    -v `pwd`/gptj-result:/result \
    furiosaai/furiosa-mlperf:latest \
    gpt-j-offline --test-mode performance-only /model /result



.. warning::

  The above example uses the ``--privileged`` option for simplicity, but it is not recommended for security reasons.
  If you use Kubernetes, please refer to :ref:`CloudNativeToolkit` to learn more about the best practice.


Benchmark Examples
====================================================

BERT Large
---------------------------

The BERT benchmark can show the good performance with a single RNGD card.

Please run the follwing command for for the offline Scenario:

.. code-block:: sh

  furiosa-mlperf bert-offline ./mlperf-bert-large ./bert-offline-result \
    --devices "npu:0:*"


To run BERT-large server scenario, you need to specify the target QPS
in an user config file to get the expected performance.

.. code-block:: sh

  cat << EOF > user.conf
  bert.Server.target_qps = 1900
  EOF

.. note::

  The default target qps of MLPerf is ``1``, and too light-weight workload cannot
  show the full performance in the server scenario.

Then, you can run the following command with ``--user-conf`` option:

.. code-block:: sh

  furiosa-mlperf bert-server ./mlperf-bert-large ./bert-server-result \
    --devices "npu:0:*" --user-conf ./user.conf


.. tip::

  You can experience RNGD cards if you specify more device as the following.

  .. code-block:: sh

    furiosa-mlperf bert-offline ./mlperf-bert-large ./bert-offline-result \
      --devices "npu:0:*,npu:1:*" --user-conf ./user.conf


GPT-J 6B benchmark
----------------------------

The GPT-J benchmark also can show the good performance with a single RNGD.

To run GPT-J 6B serving inference benchmark, you can use the following command:

.. code-block:: sh

  furiosa-mlperf gpt-j-server ./mlperf-gpt-j-6b ./gpt-j-server-result

To run GPT-J 6B offline inference benchmark, you can use the following command:

.. code-block:: sh

  furiosa-mlperf gpt-j-offline ./mlperf-gpt-j-6b ./gpt-j-offline-result


Llama 3.1 70B benchmark
-----------------------------

Llama 3.1 70B can fit to at least 2 RNGDs. For the best performance, you will need 8 RNGD cards.

To run Llama 3.1 70B serving inference benchmark, you can use the following command:

.. code-block:: sh

  furiosa-mlperf llama-3.1-server ./Llama-3.1-70B-Instruct ./llama-3.1-server-result


To run Llama 3.1 70B offline inference benchmark, you can use the following command:

.. code-block:: sh

  furiosa-mlperf llama-3.1-offline ./Llama-3.1-70B-Instruct ./llama-3.1-offline-result
