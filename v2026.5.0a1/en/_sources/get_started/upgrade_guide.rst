
.. _UpgradeGuide:

******************************
Upgrading FuriosaAI's Software
******************************

.. note::

   Furiosa SDK 2026.4 requires the **2026.4.0 RNGD driver**. Upgrade
   ``furiosa-driver-rngd`` before running Furiosa-LLM 2026.4. Using a
   2026.3.x driver may cause unexpected issues. Also review the CLI and
   Python configuration migration table in :ref:`Release2026_4_0_BreakingChanges`.

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
   +------+--------+-------------------+---------+---------+--------------+
   | Arch | Device | Firmware          | Temp.   | Power   | PCI-BDF      |
   +------+--------+-------------------+---------+---------+--------------+
   | rngd | npu0   | 2026.3.0, 2d3f72a | 28.01°C | 37.00 W | 0000:83:00.0 |
   +------+--------+-------------------+---------+---------+--------------+
   | rngd | npu1   | 2026.3.0, 2d3f72a | 27.25°C | 37.00 W | 0000:84:00.0 |
   +------+--------+-------------------+---------+---------+--------------+
   | rngd | npu2   | 2026.3.0, 2d3f72a | 27.95°C | 37.00 W | 0000:c3:00.0 |
   +------+--------+-------------------+---------+---------+--------------+
   | rngd | npu3   | 2026.3.0, 2d3f72a | 29.12°C | 37.00 W | 0000:c4:00.0 |
   +------+--------+-------------------+---------+---------+--------------+


Upgrading Furiosa-LLM
=====================

To upgrade Furiosa-LLM, use ``uv pip``:

.. code-block:: sh

   uv pip install --upgrade --torch-backend=auto furiosa-llm


.. _FuriosaSMIPythonMigration:

Migrating the Furiosa SMI Python Package
========================================

When upgrading from **Furiosa SDK 2026.3.0 or earlier to 2026.4.0**, follow
this guide if your Python application uses ``furiosa-smi-py`` and you are
migrating it to ``furiosa-smi``.

Replace ``furiosa-smi-py`` with ``furiosa-smi`` in your application's
Python dependencies, then install the new package:

.. code-block:: sh

   pip install furiosa-smi

The packages have separate import names, so installing ``furiosa-smi`` does
not update existing ``furiosa_smi_py`` imports. Update your imports and API
calls as follows:

.. list-table:: Python API changes
   :header-rows: 1
   :widths: 50 50

   * - Previous API (``furiosa_smi_py``)
     - New API (``furiosa_smi``)
   * - ``import furiosa_smi_py``
     - ``import furiosa_smi``
   * - ``furiosa_smi_py.init()``
     - Remove the call; explicit initialization is no longer required.
   * - ``device.device_info().name()``
     - ``device.device_name()`` returns a name such as ``npu0``.
   * - ``device.device_info().uuid()``
     - ``device.uuid()``
   * - ``device.device_info().firmware_version()``
     - ``device.firmware_version()``
   * - ``device.device_info().serial()``
     - ``device.serial()``
   * - ``device.device_info().numa_node()``
     - ``device.numa_node()``
   * - ``device.core_status().pe_status()``
     - ``device.core_status().items()`` yields ``(core, status)`` pairs.

The following examples query the same device information and list available
cores before and after migration.

**Before:** ``furiosa-smi-py``

.. code-block:: python

   import furiosa_smi_py as smi

   smi.init()
   print("Driver:", smi.driver_info())
   for device in smi.list_devices():
       info = device.device_info()
       print(info.name().split("/")[-1], info.uuid(), info.firmware_version())
       for pe_status in device.core_status().pe_status():
           if pe_status.status() == smi.CoreStatus.Available:
               print("Available core:", pe_status.core())

**After:** ``furiosa-smi``

.. code-block:: python

   import furiosa_smi as smi

   print("Driver:", smi.driver_info())
   for device in smi.list_devices():
       print(device.device_name(), device.uuid(), device.firmware_version())
       for core, status in device.core_status().items():
           if status == smi.CoreStatus.Available:
               print("Available core:", core)

``core_status()`` now returns a dictionary mapping core indices to
``CoreStatus`` values. Compare these values with ``==``, as shown above;
``is`` compares Python object identity and should not be used here.
