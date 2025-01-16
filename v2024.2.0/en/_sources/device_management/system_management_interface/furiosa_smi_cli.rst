.. _FuriosaSMICLI:

****************************************************
Furiosa SMI CLI
****************************************************

The ``furiosa-smi`` CLI provides a variety of subcommands and has the ability to obtain information or control the device.


Installing ``furiosa-smi`` CLI
=========================================

To install the ``furiosa-smi`` CLI, install the ``furiosa-smi`` package.

The minimum requirements for installing ``furiosa-smi`` package are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`

Then, please install the ``furiosa-smi`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-smi

This command installs packages ``furiosa-libsmi`` and ``furiosa-smi``.


Synopsis
--------------------------------

.. code-block:: sh

    furiosa-smi <sub command> [option] ..


``furiosa-smi info``
---------------------------------------------
After installing the kernel driver, you can use the ``furiosa-smi`` command to check whether the NPU device is recognized.
Currently, this command provides the ``furiosa-smi info`` command to output temperature, power consumption and PCI information of the NPU device.
If the device is not visible with this command after mounting it on the machine, please install the driver.
If you add the ``--full`` option to the ``info`` command, you can see the device's UUID and serial number information together.


.. code-block:: sh

  $ furiosa-smi info
  +------+--------+----------------+---------+---------+--------------+
  | Arch | Device | Firmware       | Temp.   | Power   | PCI-BDF      |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu0   | 0.0.21+d6cada3 | 29.81°C | 35.00 W | 0000:30:00.0 |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu1   | 0.0.21+d6cada3 | 32.97°C | 36.00 W | 0000:3f:00.0 |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu2   | 0.0.21+d6cada3 | 32.04°C | 36.00 W | 0000:40:00.0 |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu3   | 0.0.21+d6cada3 | 31.48°C | 36.00 W | 0000:41:00.0 |
  +------+--------+----------------+---------+---------+--------------+


  $ furiosa-smi info --format full
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | Arch | Device | UUID                                 | S/N        | Firmware       | Temp.   | Power   | Clock | PCI-BDF      | PCI-DEV |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu0   | 8EAE95DA-2625-40D0-8838-68EA772193A8 | RNGD000002 | 0.0.21+d6cada3 | 30.00°C | 35.00 W | N/A   | 0000:30:00.0 | 510:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu1   | 50D4EF2E-E7B8-4438-8C69-829B76CEF6D0 | RNGD000126 | 0.0.21+d6cada3 | 32.97°C | 36.00 W | N/A   | 0000:3f:00.0 | 506:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu2   | B45DF4F9-AA6C-447B-85AF-146DB0B9B31B | RNGD000128 | 0.0.21+d6cada3 | 32.60°C | 36.00 W | N/A   | 0000:40:00.0 | 504:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu3   | 2D56DDB6-B67E-42A9-9DC5-8EA2EBDA055F | RNGD000123 | 0.0.21+d6cada3 | 31.67°C | 36.00 W | N/A   | 0000:41:00.0 | 502:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+

``furiosa-smi status``
---------------------------------------------
The ``status`` subcommand provides information about the device files available on the NPU device.
You can also check whether each core present in the NPU is in use or idle.

.. code-block:: sh

  $ furiosa-smi status
  +------+--------+----------+----------------+------------------+
  | Arch | Device | Liveness | Cores          | Core Utilization |
  +------+--------+----------+----------------+------------------+
  |      |        |          | 0 (available), | Core 0: 0.00%,   |
  |      |        |          | 1 (available), | Core 1: 0.00%,   |
  |      |        |          | 2 (available), | Core 2: 0.00%,   |
  | rngd | npu0   | alive    | 3 (available), | Core 3: 0.00%,   |
  |      |        |          | 4 (available), | Core 4: 0.00%,   |
  |      |        |          | 5 (available), | Core 5: 0.00%,   |
  |      |        |          | 6 (available), | Core 6: 0.00%,   |
  |      |        |          | 7 (available)  | Core 7: 0.00%    |
  +------+--------+----------+----------------+------------------+
  |      |        |          | 0 (available), | Core 0: 0.00%,   |
  |      |        |          | 1 (available), | Core 1: 0.00%,   |
  |      |        |          | 2 (available), | Core 2: 0.00%,   |
  | rngd | npu1   | alive    | 3 (available), | Core 3: 0.00%,   |
  |      |        |          | 4 (available), | Core 4: 0.00%,   |
  |      |        |          | 5 (available), | Core 5: 0.00%,   |
  |      |        |          | 6 (available), | Core 6: 0.00%,   |
  |      |        |          | 7 (available)  | Core 7: 0.00%    |
  +------+--------+----------+----------------+------------------+
  |      |        |          | 0 (available), | Core 0: 0.00%,   |
  |      |        |          | 1 (available), | Core 1: 0.00%,   |
  |      |        |          | 2 (available), | Core 2: 0.00%,   |
  | rngd | npu2   | alive    | 3 (available), | Core 3: 0.00%,   |
  |      |        |          | 4 (available), | Core 4: 0.00%,   |
  |      |        |          | 5 (available), | Core 5: 0.00%,   |
  |      |        |          | 6 (available), | Core 6: 0.00%,   |
  |      |        |          | 7 (available)  | Core 7: 0.00%    |
  +------+--------+----------+----------------+------------------+
  |      |        |          | 0 (available), | Core 0: 0.00%,   |
  |      |        |          | 1 (available), | Core 1: 0.00%,   |
  |      |        |          | 2 (available), | Core 2: 0.00%,   |
  | rngd | npu3   | alive    | 3 (available), | Core 3: 0.00%,   |
  |      |        |          | 4 (available), | Core 4: 0.00%,   |
  |      |        |          | 5 (available), | Core 5: 0.00%,   |
  |      |        |          | 6 (available), | Core 6: 0.00%,   |
  |      |        |          | 7 (available)  | Core 7: 0.00%    |
  +------+--------+----------+----------------+------------------+

``furiosa-smi ps``
---------------------------------------------
The ``ps`` subcommand prints information about the OS process currently occupying the NPU device.

.. code-block:: sh

  $ furiosa-smi ps
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+
  | PID  | NPU core                                           | CMD                                                                           |
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+
  | 3283 | npu0:[0, 7], npu1:[0, 7], npu2:[0, 7], npu3:[0, 7] | furiosa-mlperf llama-3.1-offline ./ --devices npu:0:*,npu:1:*,npu:2:*,npu:3:* |
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+


``furiosa-smi topo``
---------------------------------------------
The ``topo`` subcommand shows the topology of the NPU device and its NUMA node.

.. code-block:: sh

  $ furiosa-smi topo
  +--------+------+--------+--------+--------+-----------+
  | Device | npu0 | npu1   | npu2   | npu3   | NUMA node |
  +--------+------+--------+--------+--------+-----------+
  | npu0   | Noc  | Cpu    | Cpu    | Cpu    | 0         |
  +--------+------+--------+--------+--------+-----------+
  | npu1   | Cpu  | Noc    | Bridge | Bridge | 0         |
  +--------+------+--------+--------+--------+-----------+
  | npu2   | Cpu  | Bridge | Noc    | Bridge | 0         |
  +--------+------+--------+--------+--------+-----------+
  | npu3   | Cpu  | Bridge | Bridge | Noc    | 0         |
  +--------+------+--------+--------+--------+-----------+

  Legend:

    Noc          = Connection within the same npu chip
    Bridge       = Devices communicating via one or more PCIe switches
    Cpu          = Devices communicating exclusively within a single CPU socket
    Interconnect = Devices communicating via inter-socket links (e.g., QPI, GMI)
    Unknown      = Connection type is unidentified
