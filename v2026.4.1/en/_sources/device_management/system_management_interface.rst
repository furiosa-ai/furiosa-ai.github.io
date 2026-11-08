.. _FuriosaSMI:

****************************************************
Furiosa SMI
****************************************************

Furiosa System Management Interface (furiosa-smi) provides the capability to manage and monitor FuriosaAI NPU products.

The ``furiosa-smi`` command-line interface (CLI) no longer requires the legacy ``furiosa-libsmi`` system package.
The CLI tool allows users and/or administrators to access npu information, system topology, utilization, and performance data.
Visit :ref:`Furiosa SMI CLI <FuriosaSMICLI>` page for more information.

Furiosa SMI provides a Python API through the ``furiosa-smi`` package; the legacy C API is provided by ``furiosa-libsmi``.
It is designed for developers to integrate the Furiosa NPU into their applications, platforms and infrastructure.
Visit :ref:`Furiosa SMI Library <FuriosaSMILIB>` page for more information.

.. toctree::
   :maxdepth: 1

   /device_management/system_management_interface/furiosa_smi_cli
   /device_management/system_management_interface/furiosa_smi_lib
