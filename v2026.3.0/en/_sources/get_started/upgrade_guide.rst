
.. _UpgradeGuide:

******************************
Upgrading FuriosaAI's Software
******************************
The high-level upgrade workflow is as follows:

1. Upgrade the driver package.
2. Upgrade the firmware tool and image if a new version is available.
3. Perform a cold reboot of the system to apply the changes if the device firmware is upgraded.
4. Run the ``furiosa-smi info`` CLI tool to confirm the new driver and firmware versions in all devices.
5. Upgrade Furiosa-LLM if necessary.

.. warning::

  Make sure to stop all running applications using NPUs before upgrading the SDK.
  Otherwise, the new driver kernel module won't be loaded.
  In that case, you will need to reboot the system to load the new driver kernel
  module.

Upgrading the Driver, Firmware, and furiosa-smi
===============================================

Please follow the steps below to upgrade the Furiosa software stack:

.. tab-set::

  .. tab-item:: APT (Debian/Ubuntu)
    :sync: apt

    .. code-block:: sh

      sudo apt update && sudo apt install -y furiosa-driver-rngd furiosa-smi

  .. tab-item:: RPM (Rocky Linux)
    :sync: rpm-rocky

    .. code-block:: sh

      sudo dnf makecache && sudo dnf upgrade -y furiosa-driver-rngd furiosa-smi

  .. tab-item:: RPM (RHEL)
    :sync: rpm-rhel

    .. code-block:: sh

      sudo dnf makecache && sudo dnf upgrade -y furiosa-driver-rngd furiosa-smi

.. _UpgradingDeviceFirmware:

Usually, major releases of the Furiosa SDK include a firmware upgrade.
Before upgrading the firmware, install the firmware updater and firmware image packages:

.. tab-set::

  .. tab-item:: APT (Debian/Ubuntu)
    :sync: apt

    .. code-block:: sh

      sudo apt install -y furiosa-firmware-tools-rngd furiosa-firmware-image-rngd

  .. tab-item:: RPM (Rocky Linux)
    :sync: rpm-rocky

    .. code-block:: sh

      sudo dnf install -y furiosa-firmware-tools-rngd furiosa-firmware-image-rngd

  .. tab-item:: RPM (RHEL)
    :sync: rpm-rhel

    .. code-block:: sh

      sudo dnf install -y furiosa-firmware-tools-rngd furiosa-firmware-image-rngd

To upgrade the firmware on all RNGD devices, run:

.. code-block:: sh

   sudo furiosa_rngd_updater_all

Alternatively, you can upgrade the firmware on a specific RNGD by specifying its BDF and the firmware image:

.. code-block:: sh

   sudo furiosa_rngd_updater -b <BDF> -f <firmware image>
   # Example:
   # sudo furiosa_rngd_updater -b 0000:43:00.0 -f /usr/lib/firmware/furiosa-rngd/rngd_fw_v2026.3.0.bin

.. note::

  Before the 2026.3 release, the firmware updater was executed automatically when the firmware image package was installed.
  Starting with the 2026.3 release, you must run the firmware updater manually after installing the firmware image package.

.. note::

  Please remember to cold reboot the system after the firmware upgrade process is completed.
  If the firmware upgrade process is interrupted, the device may become unusable.
  In this case, please contact FuriosaAI support for assistance.


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


Upgrading Furiosa-LLM
=====================

To upgrade Furiosa-LLM, use ``uv pip``:

.. code-block:: sh

   uv pip install --upgrade --torch-backend=auto furiosa-llm
