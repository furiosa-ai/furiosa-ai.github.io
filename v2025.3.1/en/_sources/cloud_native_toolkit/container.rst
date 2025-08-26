.. _Container:

####################################
Container Support
####################################

Container Runtime Support
==========================================

We do support the following versions of container runtimes:

* Docker: v25.0.0 or later
* ContainerD: v1.7.0 or later
* CRI-O: v1.28.0 or later

.. note::

  The supported container runtimes above (Docker, ContainerD, CRI-O) are validated with the default low-level container runtime ``runc``.
  Other low-level runtimes or isolation technologies (such as ``crun``, ``gVisor``, or ``firecracker``) are not officially tested and may have compatibility limitations.

.. warning::

  Some container runtimes use default seccomp profiles that filter out the following ``io_uring`` syscalls due to `security considerations <https://github.com/moby/moby/pull/46762>`_:

  - io_uring_setup
  - io_uring_enter
  - io_uring_register

  RNGD driver relies on these syscalls. Therefore, users must either create and apply a custom seccomp profile explicitly permitting these syscalls or run containers with the unconfined seccomp setting.

Container Device Interface Support
===========================================

Container Device Interface (CDI) is a standard interface and specification for exposing devices to container runtimes.
It provides abstract mechanisms required to allocate and expose these devices to containers.
This abstraction simplifies device management and enhances user experience by hiding complex device integration details.

Installing ``furiosa-cdi`` CLI
-------------------------------------------

To install the ``furiosa-cdi`` CLI, install the ``furiosa-cdi`` package.

The minimum requirements for installing ``furiosa-cdi`` package are as follows:

* Ubuntu 22.04 LTS (Debian Bookworm) or later
* Linux Kernel 6.3 or later
* Administrator privileges on system (root)
* :ref:`AptSetup` and :ref:`InstallingPrerequisites`

Then, please install the ``furiosa-cdi`` package as follows:

.. code-block:: sh

  sudo apt update
  sudo apt install -y furiosa-cdi

This command installs packages ``furiosa-libsmi`` and ``furiosa-cdi``.

Generating CDI Specification
-------------------------------------------

Following command generates a CDI specification file in the ``/etc/cdi`` directory, enabling your container runtime to identify and manage Furiosa NPUs as CDI resources.

.. code-block:: sh

    sudo furiosa-cdi generate

Configuring Container Runtimes
-------------------------------------------

Depending on the container runtime and version, CDI feature may need to be explicitly enabled through configuration.
The section below provides detailed instructions for configuring CDI for each supported runtime.

Docker
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Docker currently supports CDI as an experimental feature.
Update the Docker daemon configuration file ``daemon.json`` to enable CDI feature and include ``/etc/cdi`` in the ``cdi-spec-dirs`` setting.
Below is an example of ``daemon.json`` configuration:

.. code-block:: json

  {
    "features": {
       "cdi": true
    },
    "cdi-spec-dirs": ["/etc/cdi/", "/var/run/cdi"]
  }

For more details, refer to the official Docker documentation on `enabling CDI devices <https://docs.docker.com/reference/cli/dockerd/#enable-cdi-devices>`_.

ContainerD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ContainerD supports CDI starting from version 2.0.0 by default.
For earlier versions, CDI must be explicitly enabled through the configuration file config.toml, typically located at ``/etc/containerd/config.toml``.

To enable CDI support, add the following configuration:

.. code-block:: toml

  [plugins."io.containerd.grpc.v1.cri"]
  enable_cdi = true
  cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]

After modifying the configuration, restart ContainerD to apply the changes:

.. code-block:: sh

  sudo systemctl restart containerd

For more details, refer to the official ContainerD documentation on `enabling CDI support <https://github.com/containerd/containerd/blob/main/docs/cri/config.md>`_.


CRI-O
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CRI-O enables CDI support by default, configured to use the standard CDI directories ``/etc/cdi`` and ``/var/run/cdi``.


Assign Furiosa NPUs to Containers using CDI
-------------------------------------------

After generating the CDI specification, use the furiosa-cdi list command to discover available NPU resources.
The special entry ``all`` aggregates all available NPUs for simultaneous allocation:

.. code-block:: sh

  furiosa-cdi list
  +----------------+------+
  |      KIND      | NAME |
  +----------------+------+
  |                | npu0 |
  |                | npu1 |
  |                | npu2 |
  |                | npu3 |
  | furiosa.ai/npu | npu4 |
  |                | npu5 |
  |                | npu6 |
  |                | npu7 |
  |                | all  |
  +----------------+------+

Below is an example of assigning ``npu0`` to a Docker container using CDI:

.. code-block:: sh

   docker run --security-opt seccomp=unconfined --device=furiosa.ai/npu=npu0 -it ubuntu:latest /bin/bash

To allocate multiple NPUs, repeat the ``--device`` option or use the ``all`` entry:

.. code-block:: sh

  # Allocating multiple NPUs
  docker run --security-opt seccomp=unconfined \
    --device=furiosa.ai/npu=npu0 \
    --device=furiosa.ai/npu=npu1 \
    -it ubuntu:latest /bin/bash

  # Allocating all NPUs at once
  docker run --security-opt seccomp=unconfined \
    --device=furiosa.ai/npu=all \
    -it ubuntu:latest /bin/bash


Similarly, for ContainerD (ctr/nerdctl) and CRI-O, use their respective CDI syntax to reference Furiosa NPU devices.


Assign Furiosa NPUs to Containers Manually
===========================================

In environments where CDI support is unavailable or if manual allocation of Furiosa NPUs is preferred, this section provides guidance for manually assigning Furiosa NPUs to containers.

To use a Furiosa NPU inside a container, the corresponding device nodes must be mapped into the container.
Each Furiosa card has 35 required device nodes. The device node paths follow the format below, where ``{index}`` represents the NPU device number (e.g., ``npu0``, ``npu1``):

.. code-block:: text

  /dev/rngd/npu{index}bar0
  /dev/rngd/npu{index}bar2
  /dev/rngd/npu{index}bar4
  /dev/rngd/npu{index}ch0
  /dev/rngd/npu{index}ch0r
  /dev/rngd/npu{index}ch1
  /dev/rngd/npu{index}ch1r
  /dev/rngd/npu{index}ch2
  /dev/rngd/npu{index}ch2r
  /dev/rngd/npu{index}ch3
  /dev/rngd/npu{index}ch3r
  /dev/rngd/npu{index}ch4
  /dev/rngd/npu{index}ch4r
  /dev/rngd/npu{index}ch5
  /dev/rngd/npu{index}ch5r
  /dev/rngd/npu{index}ch6
  /dev/rngd/npu{index}ch6r
  /dev/rngd/npu{index}ch7
  /dev/rngd/npu{index}ch7r
  /dev/rngd/npu{index}dmar
  /dev/rngd/npu{index}mgmt
  /dev/rngd/npu{index}pe0
  /dev/rngd/npu{index}pe0-1
  /dev/rngd/npu{index}pe0-3
  /dev/rngd/npu{index}pe1
  /dev/rngd/npu{index}pe2
  /dev/rngd/npu{index}pe2-3
  /dev/rngd/npu{index}pe3
  /dev/rngd/npu{index}pe4
  /dev/rngd/npu{index}pe4-5
  /dev/rngd/npu{index}pe4-7
  /dev/rngd/npu{index}pe5
  /dev/rngd/npu{index}pe6
  /dev/rngd/npu{index}pe6-7
  /dev/rngd/npu{index}pe7

Below is an example of how to assign the device nodes for ``npu0`` to a container using Docker.
For ContainerD (ctr/nerdctl) and CRI-O, pass the device nodes listed above and configure the required seccomp settings.

.. code-block:: sh

  docker run --security-opt seccomp=unconfined -it \
    --device /dev/rngd/npu0bar0:rw \
    --device /dev/rngd/npu0bar2:rw \
    --device /dev/rngd/npu0bar4:rw \
    --device /dev/rngd/npu0ch0:rw \
    --device /dev/rngd/npu0ch0r:rw \
    --device /dev/rngd/npu0ch1:rw \
    --device /dev/rngd/npu0ch1r:rw \
    --device /dev/rngd/npu0ch2:rw \
    --device /dev/rngd/npu0ch2r:rw \
    --device /dev/rngd/npu0ch3:rw \
    --device /dev/rngd/npu0ch3r:rw \
    --device /dev/rngd/npu0ch4:rw \
    --device /dev/rngd/npu0ch4r:rw \
    --device /dev/rngd/npu0ch5:rw \
    --device /dev/rngd/npu0ch5r:rw \
    --device /dev/rngd/npu0ch6:rw \
    --device /dev/rngd/npu0ch6r:rw \
    --device /dev/rngd/npu0ch7:rw \
    --device /dev/rngd/npu0ch7r:rw \
    --device /dev/rngd/npu0dmar:rw \
    --device /dev/rngd/npu0mgmt:rw \
    --device /dev/rngd/npu0pe0:rw \
    --device /dev/rngd/npu0pe0-1:rw \
    --device /dev/rngd/npu0pe0-3:rw \
    --device /dev/rngd/npu0pe1:rw \
    --device /dev/rngd/npu0pe2:rw \
    --device /dev/rngd/npu0pe2-3:rw \
    --device /dev/rngd/npu0pe3:rw \
    --device /dev/rngd/npu0pe4:rw \
    --device /dev/rngd/npu0pe4-5:rw \
    --device /dev/rngd/npu0pe4-7:rw \
    --device /dev/rngd/npu0pe5:rw \
    --device /dev/rngd/npu0pe6:rw \
    --device /dev/rngd/npu0pe6-7:rw \
    --device /dev/rngd/npu0pe7:rw \
    ubuntu:latest /bin/bash
