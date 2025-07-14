.. _QuickStartWithFuriosaTorch:

===================================
Quick Start with Furiosa-Torch
===================================

Installing FuriosaAI RNGD Extension
====================================
This section describes how to install the prerequisite packages
and the FuriosaAI RNGD extension for PyTorch.

The minimum requirements are as follows:

* Ubuntu 22.04 LTS (or Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges (root)

Please follow the instructions in the official FuriosaAI documentation to install the necessary prerequisites at
`Furiosa Docs - Installing Prerequisites <https://developer.furiosa.ai/latest/en/getting_started/prerequisites.html#installing-prerequisites>`_.

After installing the prerequisites, you can install the FuriosaAI RNGD extension for PyTorch using the following command:

.. code-block:: bash

    PIP_EXTRA_INDEX_URL=https://pypi.corp.furiosa.ai/simple pip install furiosa-torch
    pip list | grep furiosa-torch
    # shows
    # furiosa-torch                 x.x.x.yyyzzzzzz
    # furiosa-torch-cpp-ext         x.x.x.yyyzzzzzz


Verifying the Installation
====================================

To verify that the pre-requisites and the FuriosaAI RNGD extension for PyTorch are installed correctly, you can run the following command:

.. code-block:: sh

    $ furiosa-smi info

    +-------+------+--------+---------------+---------------+---------+---------+--------------+
    | Index | Arch | Device | Firmware      | PERT          | Temp.   | Power   | PCI-BDF      |
    +-------+------+--------+---------------+---------------+---------+---------+--------------+
    |   0   | rngd | npu0   | 1.2.0+25b24d3 | 0.1.0+52e5705 | 34.41°C | 36.48 W | 0000:17:00.0 |
    +-------+------+--------+---------------+---------------+---------+---------+--------------+
    |   1   | rngd | npu1   | 1.2.0+25b24d3 | 0.1.0+52e5705 | 32.59°C | 37.44 W | 0000:2a:00.0 |
    +-------+------+--------+---------------+---------------+---------+---------+--------------+
    ...

    $ python -c "from furiosa import native_torch;print(native_torch.version())"
    0.1.0-dev


Perform a basic calculation on RNGD
====================================

To perform a basic calculation using the FuriosaAI RNGD extension for PyTorch, you can use the following example code.
Note that ``torch.device("rngd:0")`` is used to define the RNGD device, and
``a.to(device)`` is used to move the tensor to the RNGD device.

.. literalinclude:: ../../furiosa-torch/tests/python/docs/torch/quckstart_tensor_op.py
   :language: Python


Compile a simple PyTorch model to RNGD
=========================================

To compile a PyTorch model for RNGD and run on RNGD, you can use the following example code.
Here, ``furiosa.torch.backend`` allows you to compile the model for RNGD.

.. literalinclude:: ../../furiosa-torch/tests/python/docs/torch/quckstart_compile.py
   :language: Python
