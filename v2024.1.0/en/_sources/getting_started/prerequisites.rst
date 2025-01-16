.. _InstallingPrerequisites:

**********************************
Installing Prerequisites
**********************************

We will explain how to install the prerequisite packages necessary for FuriosaAI software stack.
The prerequisite packages include device driver, firmware, and PE Runtime.
These packages are available in package format for installation on Debian and Ubuntu systems.

Requirements
====================================

The minimum requirements are as follows:

* Ubuntu 22.04 LTS (or Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)


Verifying if the system has devices
====================================

You can verify the proper installation of FuriosaAI's devices on your machine by running the following commands:

.. code-block:: sh

  lspci -nn | grep FuriosaAI

If the device is properly installed, you should see the PCI information as shown below.

.. code-block:: sh

  4e:00.0 Processing accelerators [1200]: FuriosaAI, Inc. Device [1ed2:0001] (rev 01)

If the ``lspci`` command is not available, please install the following packages and run the commands to update PCIe ID database:

.. code-block:: sh

  sudo apt update && sudo apt install -y pciutils
  sudo update-pciids


.. _AptSetup:

Setting up APT server
====================================

To use the APT server provided by FuriosaAI, you must configure it on Ubuntu or Debian Linux as outlined below.

1. Install the required packages and register the signing key.

.. code-block:: sh

  sudo apt update && sudo apt install -y curl gnupg
  curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/cloud.google.gpg

2. Configure the APT server according to the instructions provided for the Linux distribution versions.

.. code-block:: sh

  echo "deb [arch=$(dpkg --print-architecture)] http://asia-northeast3-apt.pkg.dev/projects/furiosa-ai $(. /etc/os-release && echo "$VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/furiosa.list

Installing Pre-requisite Packages
====================================

If you have registered the APT server as described above, you will be able to install the required packages: the device driver and PE Runtime.

.. code-block:: sh

  sudo apt update && sudo apt install furiosa-driver-rngd furiosa-pert-rngd


:ref:`furiosa-smi <FuriosaSMI>` is an useful CLI tool for listing and managing FuriosaAI NPUs.

.. code-block:: sh

  sudo apt install furiosa-smi

Checking NPU devices
====================================

Once the device driver and :ref:`furiosa-smi <FuriosaSMI>` are successfully installed,
you can check the list of NPU devices as following command:

.. code-block:: sh

  furiosa-smi info

Output:

.. code-block::

  +------+--------+----------------+---------+---------+--------------+
  | Arch | Device | Firmware       | Temp.   | Power   | PCI-BDF      |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu0   | 0.0.16+b4a67ca | 28.88°C | 38.00 W | 0000:4e:00.0 |
  +------+--------+----------------+---------+---------+--------------+

Please refer to :ref:`FuriosaSMI` to learn more about ``furiosa-smi`` command.


Upgrading Device Firmware
====================================

Upgrading firmware versions can improve the performance and stability of the devices.
If there is newer firmware in the latest release, you can upgrade them using the following methods:

.. code-block:: sh

  sudo apt install furiosa-firmware-tools-rngd
  sudo apt install furiosa-firmware-image-rngd

Installing the ``furiosa-firmware-image-rngd`` package will automatically upgrade the firmware.
The process takes approximately 3 to 5 minutes per device to complete.
