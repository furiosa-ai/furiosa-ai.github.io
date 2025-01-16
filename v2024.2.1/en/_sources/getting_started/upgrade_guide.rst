
.. _UpgradeGuide:

*****************************************
Upgrading Furiosa Software Stack
*****************************************
The upgrade workflow at high level is the following:

1. Upgrade the prerequisite packages (the driver and PE Runtime packages)
2. Upgrade the Firmware tool and image if there's new version.
3. Do cold reboot the system to apply the changes if the device firmware is upgraded.
4. Run ``furiosa-smi info`` CLI tool to check the new driver and firmware versions in all devices.
5. Upgrade Furiosa LLM if necessary.


Please follow the steps below to upgrade the Furiosa software stack:

.. note::

  Please make sure to stop all the running applications using NPUs before upgrading the SDK.
  Otherwise, the old driver kernel module won't be unloaded and the new driver cannot loaded.
  In that case, you need to reboot the system to load the new driver kernel module.

.. code-block:: sh

   sudo apt update && sudo apt install -y furiosa-driver-rngd furiosa-pert-rngd furiosa-smi

Usually, the major release of the Furiosa SDK includes the firmware upgrade for RNGD.
You can upgrade the firmware by running the following command:

.. code-block:: sh

   sudo apt update && sudo apt install -y furiosa-firmware-tools-rngd furiosa-firmware-image-rngd


.. note::

  Please do not forget to cold reboot the system after the firmware upgrade process is completed.
  If you learn more about the firmware upgrade process, please refer to :ref:`Upgrading Device Firmware <UpgradingDeviceFirmware>`.

Please check the new driver and firmware versions in all devices by running the following command:

.. code-block:: sh

   $ furiosa-smi info
   +------+--------+------------------+------------------+---------+---------+--------------+
   | Arch | Device | Firmware         | PERT             | Temp.   | Power   | PCI-BDF      |
   +------+--------+------------------+------------------+---------+---------+--------------+
   | rngd | npu0   | 2024.2.0+7a11888 | 2024.2.1+de20a9f | 30.62°C | 35.00 W | 0000:17:00.0 |
   +------+--------+------------------+------------------+---------+---------+--------------+
   | rngd | npu1   | 2024.2.0+7a11888 | 2024.2.1+de20a9f | 31.41°C | 37.00 W | 0000:2a:00.0 |
   +------+--------+------------------+------------------+---------+---------+--------------+

If you want to use the latest Furiosa LLM, you can install it via pip:

.. code-block::

   pip install --upgrade furiosa-llm
