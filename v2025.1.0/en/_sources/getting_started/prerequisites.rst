.. _InstallingPrerequisites:

************************
Installing Prerequisites
************************

This article describes how to install the prerequisite packages necessary
for the FuriosaAI software stack, namely the device driver, firmware,
and PE Runtime.
These packages are available for Debian and Ubuntu systems.

Requirements
====================================

The minimum requirements are as follows:

* Ubuntu 22.04 LTS (or Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges (root)


Checking for Installed Devices
==============================

To confirm that FuriosaAI devices are correctly installed on your system, run the following command:

.. code-block:: sh

  lspci -nn | grep FuriosaAI

If the device is properly installed, the output will be similar to the following:

.. code-block:: sh

  4e:00.0 Processing accelerators [1200]: FuriosaAI, Inc. Device [1ed2:0001] (rev 01)

If the ``lspci`` command is not available, you can install it and update the
PCIe ID database using the following commands:

.. code-block:: sh

  sudo apt update && sudo apt install -y pciutils
  sudo update-pciids


.. _AptSetup:

Setting up APT
==============

To use the APT server provided by FuriosaAI, follow the steps below:

1. Install the required packages and register the signing key.

.. code-block:: sh

  sudo apt update && sudo apt install -y curl gnupg
  curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/cloud.google.gpg

2. Configure the APT server for your Linux distribution by executing the following command:

.. code-block:: sh

  echo "deb [arch=$(dpkg --print-architecture)] http://asia-northeast3-apt.pkg.dev/projects/furiosa-ai $(. /etc/os-release && echo "$VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/furiosa.list


Installing Prerequisite Packages
================================

After registering FuriosaAI's APT server as described above, you can now
install the required packages: the device driver, the PE Runtime, and optionally
the device control/information tool (:ref:`furiosa-smi <FuriosaSMICLI>`).

.. code-block:: sh

  sudo apt update
  sudo apt install build-essential linux-modules-extra-$(uname -r) linux-headers-$(uname -r)
  sudo apt install furiosa-driver-rngd furiosa-pert-rngd furiosa-smi


Adding a user to the furiosa Group
==================================
Linux is a multi-user operating system that enables file and device access for
both the owner and users within a specific group. The RNGD device driver creates
a group called ``furiosa`` and restricts access to RNGD devices exclusively to users
who are members of the ``furiosa`` group. To add a user to a member of furiosa group,
please run as follows:

.. code-block:: sh

  sudo usermod -aG friosa <username>

Replace <username> with the name of the user you want to add to the furiosa group.
For example, in order to add the current user (i.e., $USER) to the furiosa group, you can run as follows:

.. code-block:: sh

  sudo usermod -aG furiosa $USER

Upon logging out and logging back in, the change to the group membership will take effect.


Verifying NPU Device Installation
=================================

Once the device driver and :ref:`furiosa-smi <FuriosaSMICLI>` are successfully installed,
you can check the list of NPU devices by running the following command:

.. code-block:: sh

  furiosa-smi info

Example output:

.. code-block::

  +------+--------+----------------+---------+---------+--------------+
  | Arch | Device | Firmware       | Temp.   | Power   | PCI-BDF      |
  +------+--------+----------------+---------+---------+--------------+
  | rngd | npu0   | 0.0.16+b4a67ca | 28.88°C | 38.00 W | 0000:4e:00.0 |
  +------+--------+----------------+---------+---------+--------------+

Please refer to :ref:`FuriosaSMICLI` to learn more about the ``furiosa-smi``
command.


.. _UpgradingDeviceFirmware:

Upgrading Device Firmware
====================================

Upgrading the firmware can improve the performance and stability of the devices.
If there is newer firmware in the latest release, you can upgrade it using the
following commands:

.. code-block:: sh

  sudo apt install furiosa-firmware-tools-rngd furiosa-firmware-image-rngd

Installing the ``furiosa-firmware-image-rngd`` package will automatically upgrade the firmware.
The process takes 3 to 5 minutes per device to complete.
The firmware upgrade process may require a reboot to complete the installation.

.. note::

  Please remember to cold reboot the system after the firmware upgrade process is completed.
  For more information about the firmware upgrade process. If the firmware upgrade process is interrupted,
  the device may become unusable. In this case, please contact FuriosaAI support for assistance.

