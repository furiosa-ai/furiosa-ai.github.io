.. _FuriosaSMI:

****************************************************
furiosa-smi
****************************************************

The ``furiosa-smi`` command provides a variety of subcommands and has the ability to obtain information or control the device.


Installing ``furiosa-smi`` command
=========================================

To install the ``furiosa-smi`` command, you need to install ``furiosa-smi`` as following:

The minimum requirements for ``furiosa-smi`` are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* Configuring the APT server and installing device drivers (:ref:`AptSetup`)

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
  | rngd | npu0   | 0.0.15+af1daaa | 30.18°C | 53.00 W | 0000:17:00.0 |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu1   | 0.0.15+af1daaa | 29.25°C | 53.00 W | 0000:2a:00.0 |
  +------+--------+----------------+---------+---------+--------------+


  $ furiosa-smi info --format full
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | Arch | Device | UUID                                 | S/N        | Firmware       | Temp.   | Power   | Clock | PCI-BDF      | PCI-DEV |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu0   | 3E93AE7C-E8EA-4C62-BED6-AD2EC0461AE8 | RNGDXXXXXX | 0.0.15+af1daaa | 30.18°C | 53.00 W |   N/A | 0000:17:00.0 | 508:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+
  | rngd | npu1   | 176DAD0F-1510-475C-91D8-5F79551CF718 | RNGDXXXXXY | 0.0.15+af1daaa | 29.44°C | 53.00 W |   N/A | 0000:2a:00.0 | 506:0   |
  +------+--------+--------------------------------------+------------+----------------+---------+---------+-------+--------------+---------+

``furiosa-smi status``
---------------------------------------------
The ``status`` subcommand provides information about the device files available on the NPU device.
You can also check whether each core present in the NPU is in use or idle.

.. code-block:: sh

  $ furiosa-smi status
  +------+--------+---------------+------------------+
  | Arch | Device | Cores         | Core Utilization |
  +------+--------+---------------+------------------+
  |      |        | 0 (occupied), | Core 0: 0.00%,   |
  |      |        | 1 (occupied), | Core 1: 0.00%,   |
  |      |        | 2 (occupied), | Core 2: 0.00%,   |
  | rngd | npu0   | 3 (occupied), | Core 3: 0.00%,   |
  |      |        | 4 (occupied), | Core 4: 0.00%,   |
  |      |        | 5 (occupied), | Core 5: 0.00%,   |
  |      |        | 6 (occupied), | Core 6: 0.00%,   |
  |      |        | 7 (occupied)  | Core 7: 0.00%    |
  +------+--------+---------------+------------------+
  |      |        | 0 (occupied), | Core 0: 0.00%,   |
  |      |        | 1 (occupied), | Core 1: 0.00%,   |
  |      |        | 2 (occupied), | Core 2: 0.00%,   |
  | rngd | npu1   | 3 (occupied), | Core 3: 0.00%,   |
  |      |        | 4 (occupied), | Core 4: 0.00%,   |
  |      |        | 5 (occupied), | Core 5: 0.00%,   |
  |      |        | 6 (occupied), | Core 6: 0.00%,   |
  |      |        | 7 (occupied)  | Core 7: 0.00%    |
  +------+--------+---------------+------------------+

``furiosa-smi ps``
---------------------------------------------
The ``ps`` subcommand prints information about the OS process currently occupying the NPU device.

.. code-block:: sh

    $ furiosa-smi ps
    +-----------+--------+------------------------------------------------------------+
    | NPU       | PID    | CMD                                                        |
    +-----------+--------+------------------------------------------------------------+
    | npu0pe0-3 | 132529 | /usr/bin/python3 /usr/local/bin/uvicorn gptj:app           |
    +-----------+--------+------------------------------------------------------------+


``furiosa-smi topo``
---------------------------------------------
The ``topo`` subcommand shows the topology of the NPU device and its NUMA node.

.. code-block:: sh

  $ furiosa-smi topo
  +--------+--------------+--------------+-----------+
  | Device | npu0         | npu1         | NUMA node |
  +--------+--------------+--------------+-----------+
  | npu0   | Noc          | Interconnect | 0         |
  +--------+--------------+--------------+-----------+
  | npu1   | Interconnect | Noc          | 0         |
  +--------+--------------+--------------+-----------+

  Legend:

    Noc          = Connection within the same npu chip
    Bridge       = Devices communicating via one or more PCIe switches
    Cpu          = Devices communicating exclusively within a single CPU socket
    Interconnect = Devices communicating via inter-socket links (e.g., QPI, GMI)
    Unknown      = Connection type is unidentified
