.. _FuriosaSMILIB:

****************************************************
Furiosa SMI Library
****************************************************

.. note::

   In **Furiosa SDK 2026.4.0**, the Python package changed from
   ``furiosa-smi-py`` to ``furiosa-smi``. Follow the
   :ref:`migration guide <FuriosaSMIPythonMigration>` to update your application.

Furiosa SMI provides programmatic interfaces for managing and monitoring
FuriosaAI NPUs, including device discovery, driver and firmware information,
and performance metrics.

Python API
==========

Use the ``furiosa-smi`` Python package to query devices and their status from
Python applications.

Installation
------------

The Python package requires Python 3.10 or later and is installed as a
Furiosa-LLM dependency (see :ref:`InstallingFuriosaLLM`). To install it
separately in your Python environment:

.. code-block:: sh

   python -m pip install --upgrade "furiosa-smi==2026.4.*"

The host must have the NPU driver installed; see :ref:`InstallingPrerequisites`.
For the system-installed command-line tool, see :ref:`FuriosaSMICLI`.

Basic usage
-----------

Import ``furiosa_smi`` and query devices directly. No explicit ``init()``
call is needed:

.. code-block:: python

   import furiosa_smi

   print("Driver:", furiosa_smi.driver_info())
   for device in furiosa_smi.list_devices():
       print("Device:", device.device_name())
       print("UUID:", device.uuid())
       print("Firmware:", device.firmware_version())
       print("Temperature:", device.device_temperature().ambient())
       print("Power (W):", device.power_consumption())

C API
=====

The C API is provided by the ``furiosa-libsmi`` system library.

Installation
------------

Install the ``furiosa-libsmi`` system package to use the C API.

The minimum requirements for installing the library are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`

Then, please install the ``furiosa-libsmi`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-libsmi

API modules
-----------

The library provides the following API modules, each designed to offer distinct functionalities for managing and monitoring NPU devices.
These modules enable developers to access essential hardware information, topology details, system-wide information, and performance metrics.

Each module provides the following features:

- **Initialize module** handles initialization of the Furiosa SMI library.

- **Device module** provides NPU device discovery and information including *device specification* and *liveness*.

- **Topology module** provides the topology status within a system including *device-to-device link type* and *p2p accessibility*.

- **System module** provides system-wide information of each NPU device, including *firmware version* and *driver version*.

- **Performance module** provides the device performance metrics including *power consumption*, *temperature*, *utilization*.

For further details on using these modules and available functions, please refer to the comprehensive `C API Documentation <https://furiosa-ai.github.io/furiosa-smi>`_.

Other language bindings
-----------------------

For Go and Rust examples and API details, see:

* `Furiosa SMI - Go API <https://pkg.go.dev/github.com/furiosa-ai/furiosa-smi-go>`_
* `Furiosa SMI - Rust API <https://crates.io/crates/furiosa-smi-rs>`_
