.. _InstallingPrerequisites:

************************
Installing Prerequisites
************************

This article describes how to install the prerequisite packages necessary
for the FuriosaAI software stack, namely the device driver, and firmware.
These packages are available for Debian/Ubuntu systems (via APT) and for
Rocky Linux / RHEL systems (via RPM).

Requirements
====================================

The minimum requirements are as follows:

* A supported Linux distribution:

  * Ubuntu 22.04 LTS (or Debian Bookworm) or later, or
  * Rocky Linux 10 / RHEL 10

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

Setting up the Package Repository
=================================

FuriosaAI distributes its packages through both an APT repository (for
Debian/Ubuntu) and an RPM repository (for Rocky Linux / RHEL). Select the tab
matching your Linux distribution.

.. tab-set::

  .. tab-item:: APT (Debian/Ubuntu)
    :sync: apt

    To use the APT server provided by FuriosaAI, follow the steps below:

    1. Install the required packages and register the signing key.

    .. code-block:: sh

      sudo apt update && sudo apt install -y curl gnupg
      curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/cloud.google.gpg

    2. Configure the APT server for your Linux distribution by executing the following command:

    .. code-block:: sh

      echo "deb [arch=$(dpkg --print-architecture)] http://asia-northeast3-apt.pkg.dev/projects/furiosa-ai $(. /etc/os-release && echo "$VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/furiosa.list

  .. tab-item:: RPM (Rocky Linux)
    :sync: rpm-rocky

    FuriosaAI provides an RPM repository for installing the same packages.
    Register it by creating the following repository file:

    .. code-block:: sh

      sudo tee /etc/yum.repos.d/furiosa.repo > /dev/null << EOF
      [furiosa]
      name=furiosa
      baseurl=https://asia-northeast3-yum.pkg.dev/projects/furiosa-ai/el10
      enabled=1
      repo_gpgcheck=0
      gpgcheck=0
      EOF

  .. tab-item:: RPM (RHEL)
    :sync: rpm-rhel

    FuriosaAI provides an RPM repository for installing the same packages.
    Register it by creating the following repository file:

    .. code-block:: sh

      sudo tee /etc/yum.repos.d/furiosa.repo > /dev/null << EOF
      [furiosa]
      name=furiosa
      baseurl=https://asia-northeast3-yum.pkg.dev/projects/furiosa-ai/el10
      enabled=1
      repo_gpgcheck=0
      gpgcheck=0
      EOF


Installing Prerequisite Packages
================================

After registering FuriosaAI's package repository as described above, you can now
install the required packages: the device driver, the PE Runtime, and optionally
the device control/information tool (:ref:`furiosa-smi <FuriosaSMICLI>`). On RPM
systems, the commands also install the EPEL repository, which provides the
kernel-build dependencies.

.. tab-set::

  .. tab-item:: APT (Debian/Ubuntu)
    :sync: apt

    .. code-block:: sh

      sudo apt update
      sudo apt install build-essential linux-modules-extra-$(uname -r) linux-headers-$(uname -r)
      sudo apt install furiosa-driver-rngd furiosa-smi

  .. tab-item:: RPM (Rocky Linux)
    :sync: rpm-rocky

    .. code-block:: sh

      sudo dnf makecache
      sudo dnf install -y epel-release
      sudo dnf install -y dkms gcc make kernel-devel-$(uname -r) kernel-headers-$(uname -r)
      sudo dnf install furiosa-driver-rngd furiosa-smi

  .. tab-item:: RPM (RHEL)
    :sync: rpm-rhel

    .. code-block:: sh

      sudo dnf makecache
      sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
      sudo dnf install -y dkms gcc make kernel-devel-$(uname -r) kernel-headers-$(uname -r)
      sudo dnf install furiosa-driver-rngd furiosa-smi


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


Upgrading the Software Stack
============================

To upgrade an existing installation — the driver, firmware, and Furiosa-LLM — see the
:ref:`UpgradeGuide`.

