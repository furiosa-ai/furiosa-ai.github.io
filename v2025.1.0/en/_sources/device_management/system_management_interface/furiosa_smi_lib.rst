.. _FuriosaSMILIB:

****************************************************
Furiosa SMI Library
****************************************************

The ``Furiosa SMI`` library is a C-based programmatic interface for managing and monitoring FuriosaAI NPUs.

Installing ``Furiosa SMI`` library
==========================================================

To install ``Furiosa SMI`` library, install ``furiosa-libsmi`` package.

The minimum requirements for installing the library are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`

Then, please install the ``furiosa-libsmi`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-libsmi

This command installs ``furiosa-libsmi`` package.

API Modules
============

The library provides the following API modules, each designed to offer distinct functionalities for managing and monitoring NPU devices.
These modules enable developers to access essential hardware information, topology details, system-wide information, and performance metrics.

Each module provides the following features:

- **Device module** provides NPU device discovery and information including *device specification* and *liveness*.

- **Topology module** provides the topology status within a system including *device-to-device link type* and *p2p accessibility*.

- **System module** provides system-wide information of each NPU device, including *firmware version* and *driver version*.

- **Performance module** provides the device performance metrics including *power consumption*, *temperature*, *utilization*.

For further details on using these modules and available functions, please refer to the comprehensive `C API Documentation <https://furiosa-ai.github.io/furiosa-smi>`_.

API Bindings
============

The ``Furiosa SMI`` library officially supports bindings for Go, Rust, Python, allowing developers to integrate NPU management and monitoring capabilities into various applications, systems, and platforms.

Supported Language Documentation:
---------------------------------

- `Furiosa SMI - Go API <https://pkg.go.dev/github.com/furiosa-ai/furiosa-smi-go>`_
- `Furiosa SMI - Rust API <https://crates.io/crates/furiosa-smi-rs>`_
- `Furiosa SMI - Python API <https://pypi.org/project/furiosa-smi-py>`_

Each link provides language-specific examples and guides to help you get started quickly.
