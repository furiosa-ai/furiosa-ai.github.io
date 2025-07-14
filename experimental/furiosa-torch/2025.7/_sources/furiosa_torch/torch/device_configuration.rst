.. _DeviceConfiguration:

Device Configuration
=====================
This section explains how to configure the device setup for FuriosaAI’s RNGD NPU.
Proper configuration ensures optimal use of the hardware resources, such as physical
and software fusion, multi-chip mesh settings, and available device discovery.

The RNGD NPU consists of 8 Processing Elements (PEs) and supports both physical and software fusion.
You can configure the number of fused PEs globally using set_fusion(). Additionally, set_mesh_kind()
allows you to define how multiple chips (NPUs) should be logically grouped.

Below is an example demonstrating how to:

*  Set the fusion level (e.g., number of fused PEs).
*  Set the mesh kind (e.g., single NPU or multi-chip mesh).
*  Retrieve the list of available devices.
*  Check the current device configuration.
*  Handle exceptions when trying to change configuration after it’s been locked.


.. note::

   These settings must be defined before any tensor operations are performed. Once set, they cannot be changed during runtime.

.. literalinclude:: ../../../../furiosa-torch/tests/python/docs/torch/device_configuration.py
   :language: Python
