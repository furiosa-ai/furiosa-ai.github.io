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
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`

Then, please install the ``furiosa-smi`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-smi


Synopsis
--------------------------------

.. code-block:: sh

    furiosa-smi <sub command> [option] ..


``furiosa-smi info``
---------------------------------------------
After installing the kernel driver, you can use the ``furiosa-smi`` command to check whether the NPU device is recognized.
Currently, this command provides the ``furiosa-smi info`` command to output temperature, power consumption and PCI information of the NPU device.
If the device is not visible with this command after mounting it on the machine, please install the driver.
If you add the ``--format full`` option to the ``info`` command, you can see the device's UUID, serial number, power limit, and core and TCP clocks together.


.. code-block:: sh

  $ furiosa-smi info
  +------+--------+---------------------+---------+-------+--------------+
  | Arch | Device | Firmware            | Temp.   | Power | PCI-BDF      |
  +------+--------+---------------------+---------+-------+--------------+
  | RNGD | npu0   | 1.7.0, 4dfed1852e7b | 40.98°C |  38 W | 0000:03:00.0 |
  +------+--------+---------------------+---------+-------+--------------+
  | RNGD | npu1   | 1.7.0, 4dfed1852e7b | 41.34°C |  37 W | 0000:04:00.0 |
  +------+--------+---------------------+---------+-------+--------------+
  | RNGD | npu2   | 1.7.0, 4dfed1852e7b | 40.43°C |  37 W | 0000:44:00.0 |
  +------+--------+---------------------+---------+-------+--------------+
  | RNGD | npu3   | 1.7.0, 4dfed1852e7b | 46.57°C |  39 W | 0000:45:00.0 |
  +------+--------+---------------------+---------+-------+--------------+


  $ furiosa-smi info --format full
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+
  | Arch | Device | UUID                                 | S/N        | Firmware            | Temp.   | Power        | Core Clock | TCP Clock | Governor    | PCI-BDF      | PCI-DEV |
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+
  | RNGD | npu0   | 007741DA-70B7-4996-AAC0-7E8E46FFC817 | RNGD000206 | 1.7.0, 4dfed1852e7b | 40.80°C | 38 W / 300 W |   2000 MHz |  1000 MHz | Performance | 0000:03:00.0 | 236:40  |
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+
  | RNGD | npu1   | A2216AE7-8D79-4086-A3B1-200AB4CE230A | RNGD000068 | 1.7.0, 4dfed1852e7b | 40.97°C | 37 W / 300 W |   2000 MHz |  1000 MHz | Performance | 0000:04:00.0 | 236:60  |
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+
  | RNGD | npu2   | B1B02CFC-ADAE-4683-BA8E-08CBA0D5F907 | RNGD000198 | 1.7.0, 4dfed1852e7b | 40.43°C | 37 W / 300 W |   2000 MHz |  1000 MHz | Performance | 0000:44:00.0 | 236:0   |
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+
  | RNGD | npu3   | 09512C86-0702-4305-824D-44454143444F | RNGD000145 | 1.7.0, 4dfed1852e7b | 46.57°C | 39 W / 300 W |   2000 MHz |  1000 MHz | Performance | 0000:45:00.0 | 236:20  |
  +------+--------+--------------------------------------+------------+---------------------+---------+--------------+------------+-----------+-------------+--------------+---------+

``furiosa-smi status``
---------------------------------------------
The ``status`` subcommand provides information about NPU status.
Use ``--format full`` to check whether each core present in the NPU is in use or idle.
When NPU is not being used, core utilization is displayed as ``-``.

.. code-block:: sh

  $ furiosa-smi status
  +------+--------+---------------------------+-------------+
  | Arch | Device | Memory                    | Utilization |
  +------+--------+---------------------------+-------------+
  | RNGD | npu0   |  0.00/47.50 GiB   (0.00%) |    0.00%    |
  +------+--------+---------------------------+-------------+
  | RNGD | npu1   |  0.00/47.50 GiB   (0.00%) |    0.00%    |
  +------+--------+---------------------------+-------------+
  | RNGD | npu2   |  0.00/47.50 GiB   (0.00%) |      -      |
  +------+--------+---------------------------+-------------+
  | RNGD | npu3   |  0.00/47.50 GiB   (0.00%) |      -      |
  +------+--------+---------------------------+-------------+

  $ furiosa-smi status --format full
  +------+--------+-----------------+------------------+
  | Arch | Device | Memory          | Core Utilization |
  +------+--------+-----------------+------------------+
  |      |        |                 | Core 0: 0.00%    |
  |      |        |                 | Core 1: 0.00%    |
  |      |        |                 | Core 2: 0.00%    |
  | RNGD | npu0   |  0.00/47.50 GiB | Core 3: 0.00%    |
  |      |        |     (0.00%)     | Core 4: 0.00%    |
  |      |        |                 | Core 5: 0.00%    |
  |      |        |                 | Core 6: 0.00%    |
  |      |        |                 | Core 7: 0.00%    |
  +------+--------+-----------------+------------------+
  |      |        |                 | Core 0: 0.00%    |
  |      |        |                 | Core 1: 0.00%    |
  |      |        |                 | Core 2: 0.00%    |
  | RNGD | npu1   |  0.00/47.50 GiB | Core 3: 0.00%    |
  |      |        |     (0.00%)     | Core 4: 0.00%    |
  |      |        |                 | Core 5: 0.00%    |
  |      |        |                 | Core 6: 0.00%    |
  |      |        |                 | Core 7: 0.00%    |
  +------+--------+-----------------+------------------+
  |      |        |                 | Core 0: -        |
  |      |        |                 | Core 1: -        |
  |      |        |                 | Core 2: -        |
  | RNGD | npu2   |  0.00/47.50 GiB | Core 3: -        |
  |      |        |     (0.00%)     | Core 4: -        |
  |      |        |                 | Core 5: -        |
  |      |        |                 | Core 6: -        |
  |      |        |                 | Core 7: -        |
  +------+--------+-----------------+------------------+
  |      |        |                 | Core 0: -        |
  |      |        |                 | Core 1: -        |
  |      |        |                 | Core 2: -        |
  | RNGD | npu3   |  0.00/47.50 GiB | Core 3: -        |
  |      |        |     (0.00%)     | Core 4: -        |
  |      |        |                 | Core 5: -        |
  |      |        |                 | Core 6: -        |
  |      |        |                 | Core 7: -        |
  +------+--------+-----------------+------------------+

``furiosa-smi ps``
---------------------------------------------
The ``ps`` subcommand prints information about the OS process currently occupying the NPU device.

.. code-block:: sh

  $ furiosa-smi ps
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+
  | PID  | NPU core                                           | CMD                                                                           |
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+
  | 3283 | npu0:[0, 7], npu1:[0, 7], npu2:[0, 7], npu3:[0, 7] | furiosa-mlperf llama-3.1-offline ./ --devices npu:0,npu:1,npu:2,npu:3 |
  +------+----------------------------------------------------+-------------------------------------------------------------------------------+


``furiosa-smi version``
---------------------------------------------
The ``version`` subcommand displays the CLI and device driver versions.

.. code-block:: sh

  $ furiosa-smi version
  - furiosa-smi
    2026.3.0-dev

  - device driver
    2026.4.0, 32ffd4e


``furiosa-smi topo``
---------------------------------------------
The ``topo`` subcommand shows the topology of the NPU device and its NUMA node.

.. code-block:: sh

  $ furiosa-smi topo
  +--------+---------+--------+--------+--------+-----------+
  | Device | npu0    | npu1   | npu2   | npu3   | NUMA node |
  +--------+---------+--------+--------+--------+-----------+
  | npu0   | Noc     | Bridge | Cpu    | Cpu    | 0         |
  +--------+---------+--------+--------+--------+-----------+
  | npu1   | Bridge  | Noc    | Cpu    | Cpu    | 0         |
  +--------+---------+--------+--------+--------+-----------+
  | npu2   | Cpu     | Cpu    | Noc    | Bridge | 0         |
  +--------+---------+--------+--------+--------+-----------+
  | npu3   | Cpu     | Cpu    | Bridge | Noc    | 0         |
  +--------+---------+--------+--------+--------+-----------+

  Legend:

    Noc          = Connection within the same npu chip
    Bridge       = Devices communicating via one or more PCIe switches
    Cpu          = Devices communicating exclusively within a single CPU socket
    Interconnect = Devices communicating via inter-socket links (e.g., QPI, GMI)
    Unknown      = Connection type is unidentified


``furiosa-smi governor``
---------------------------------------------
The ``governor`` subcommand configures the device governor profile (requires root privileges).

Governor Profiles:

* Performance : Sets NPU frequency statically to the highest
* PowerSave   : Sets NPU frequency statically to the lowest

.. code-block:: console

  $ sudo furiosa-smi governor -p powersave -i 0,1,2
  npu0's governor profile is set to PowerSave
  npu1's governor profile is set to PowerSave
  npu2's governor profile is set to PowerSave


``furiosa-smi drain``
---------------------------------------------
The ``drain`` subcommand disables/enables the NPU device (requires root privileges).

.. code-block:: sh

  $ sudo furiosa-smi drain set 0000:17:00.0
  Successfully drained device with BDF: 0000:17:00.0

  $ furiosa-smi drain list
  Drained device BDF: 0000:17:00.0

  $ sudo furiosa-smi drain unset 0000:17:00.0
  Successfully undrained device with BDF: 0000:17:00.0


``furiosa-smi power``
---------------------------------------------
The ``power`` subcommand configures the device power limit in watts (requires root privileges).
Omitting ``--indices`` applies the limit to all devices.

.. code-block:: console

  $ sudo furiosa-smi power --limit 250.5 --indices 0,1,2
  npu0's power limit is set to 250.5 W
  npu1's power limit is set to 250.5 W
  npu2's power limit is set to 250.5 W
