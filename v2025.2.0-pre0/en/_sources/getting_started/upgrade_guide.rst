
.. _UpgradeGuide:

******************************
Upgrading FuriosaAI's Software
******************************
The high-level upgrade workflow is as follows:

1. Upgrade the prerequisite packages (the driver and PE Runtime packages).
2. Upgrade the firmware tool and image if a new version is available.
3. Perform a cold reboot of the system to apply the changes if the device firmware is upgraded.
4. Run the ``furiosa-smi info`` CLI tool to confirm the new driver and firmware versions in all devices.
5. Upgrade Furiosa-LLM if necessary.

.. warning::

  Make sure to stop all running applications using NPUs before upgrading the SDK.
  Otherwise, the new driver kernel module won't be loaded.
  In that case, you will need to reboot the system to load the new driver kernel
  module.

Please follow the steps below to upgrade the Furiosa software stack:

.. code-block:: sh

   sudo apt update && sudo apt install -y furiosa-driver-rngd furiosa-pert-rngd furiosa-smi

Usually, major releases of the Furiosa SDK include a firmware upgrade.
The firmware can be upgraded by running the following command:

.. code-block:: sh

   sudo apt install -y furiosa-firmware-tools-rngd furiosa-firmware-image-rngd


.. note::

  Please remember to cold reboot the system after the firmware upgrade process is completed.
  For more information about the firmware upgrade process, please refer to
  :ref:`Upgrading Device Firmware <UpgradingDeviceFirmware>`.


To check if the new driver and firmware were installed successfully in all
devices, run the following command:

.. code-block:: sh

   $ furiosa-smi info
   +------+--------+------------------+------------------+---------+---------+--------------+
   | Arch | Device | Firmware         | PERT             | Temp.   | Power   | PCI-BDF      |
   +------+--------+------------------+------------------+---------+---------+--------------+
   | rngd | npu0   | 2024.2.0+7a11888 | 2024.2.1+de20a9f | 30.62°C | 35.00 W | 0000:17:00.0 |
   +------+--------+------------------+------------------+---------+---------+--------------+
   | rngd | npu1   | 2024.2.0+7a11888 | 2024.2.1+de20a9f | 31.41°C | 37.00 W | 0000:2a:00.0 |
   +------+--------+------------------+------------------+---------+---------+--------------+


Upgrading Furiosa-LLM is done via pip:

.. code-block::

   pip install --upgrade furiosa-llm
