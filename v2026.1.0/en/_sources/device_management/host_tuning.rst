.. _HostTuning:

****************************
Host PCI Optimization Tuning
****************************

This document describes a set of host-level tuning steps commonly used to improve
PCIe/DMA/P2P performance and reduce latency variance on Linux servers.


1. Hugepage Configuration
=========================

Allocate **64K (65536) of 2MB hugepages**.

Hugepages (i.e., "HugeTLB" pages) reduce TLB pressure and page-table walk
overhead by using larger page sizes. For workloads that frequently touch large
memory regions (e.g., DMA buffers, packet buffers, large pinned allocations),
hugepages can:

* Reduce CPU overhead from address translation (fewer TLB misses)
* Improve latency stability by reducing page management overhead
* Reduce fragmentation issues for large contiguous allocations (when allocated early)

One-time setting
----------------

To apply immediately (effective until reboot):

.. code-block:: bash

    echo 65536 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

Notes
^^^^^
- This adjusts the number of 2MB HugeTLB pages.
- The allocation can fail (partially or fully) if memory is fragmented; applying
  early after boot usually works best.

Persistent setting (every boot)
-------------------------------

Create a sysctl rule so it is applied at boot:

.. code-block:: bash

    sudo tee /etc/sysctl.d/90-hugepages.conf <<EOF
    vm.nr_hugepages = 65536
    EOF

Then apply it immediately without reboot:

.. code-block:: bash

    sudo sysctl --system


2. Disable PCI ACS (Access Control Services)
============================================

When to use this
----------------

On some systems(e.g., **NXT RNGD Server**), devices are connected behind a PCIe switch.
If peer-to-peer (P2P) data paths are expected,
**PCI ACS** enabled on the upstream switch port can prevent or penalize P2P traffic.

Example scenario:

* A device is located under a PCIe switch, and P2P throughput/latency matters.

Why ACS can impact P2P performance
----------------------------------

ACS provides isolation and routing control features (e.g., request/completion
redirection, upstream forwarding control) that improve security/isolation and
topology correctness in complex systems. However, when ACS enforces redirection,
it can force transactions to be routed upstream (toward the root complex) rather
than staying within the switch fabric. This can:

* Increase hop count / latency
* Reduce effective bandwidth
* Add contention on upstream links
* Prevent the most direct P2P path between endpoints under the same switch

Disabling ACS can therefore improve P2P performance by allowing more direct
routing within the switch.

.. warning::

    Disabling ACS reduces isolation between endpoints and may not be acceptable
    in multi-tenant / strict security environments. Apply only when your platform
    and use-case tolerate reduced PCIe isolation.

Supported Server and PCIe Switch Combinations
---------------------------------------------

The following table documents officially supported server configurations where
ACS disable is validated and supported.

.. list-table::
   :header-rows: 1
   :widths: 25 35 20 20

   * - Server Platform
     - PCIe Switch
     - Vendor / Device ID
     - ACS Control Offset
   * - NXT RNGD Server (Supermicro)
     - Broadcom / LSI PEX890xx PCIe Gen 5 Switch (rev b0)
     - ``0x1000`` / ``0xc030``
     - ``0x176``

For the configuration above, ACS is disabled on the **PCIe switch downstream
port connected to the RNGD device** in order to allow optimal PCIe P2P traffic
within the switch fabric.

Run the following command:

.. code-block:: bash

    sudo setpci -s ${PARENT_BDF} ${ACS_OFFSET}.W=0x0

* ``${PARENT_BDF}`` is the BDF of the PCIe switch port directly connected to the RNGD device (e.g., ``0000:02:03.0``).

.. code-block:: bash

    lspci -D -d 1ed2: -PP | head -n 1

    0000:00:01.1/01:00.0/02:03.0/06:00.0 Processing accelerators: FuriosaAI, Inc. Device 0001 (rev 01)

    lspci -D -s 0000:02:03.0 -nn

    0000:02:03.0 PCI bridge [0604]: Broadcom / LSI PEX890xx PCIe Gen 5 Switch [1000:c030] (rev b0)

Verify the result
-----------------

Run:

.. code-block:: bash

    lspci -vv -s ${PARENT_BDF}

After disabling, you should observe flags similar to:

.. code-block:: text

    Capabilities: [170 v1] Access Control Services
            ACSCap: SrcValid+ TransBlk+ ReqRedir+ CmpltRedir+ UpstreamFwd+ EgressCtrl- DirectTrans+
            ACSCtl: SrcValid- TransBlk- ReqRedir- CmpltRedir- UpstreamFwd- EgressCtrl- DirectTrans-

This indicates that the corresponding ACS control bits are cleared.

Auto-apply with a udev rule
---------------------------

You can use a udev rule to apply the ACS disable automatically when the device
(port) is enumerated. The following snippet provides a template. Adjust the
matching keys (vendor/device, BDF, or driver) for your environment.

Create a rules file such as ``/etc/udev/rules.d/99-furiosa.rules``:

.. code-block:: text

    # Disable ACS on a specific upstream PCIe port to improve P2P performance.
    SUBSYSTEM=="rngd_mgmt", ATTR{device_type}=="RNGD", ACTION=="add", ENV{BUSNAME}="$attr{busname}", RUN+="/usr/local/sbin/furiosa-acs-clear"

Create a script file such as ``/usr/local/sbin/furiosa-acs-clear``:

.. literalinclude:: ../../../examples/furiosa-acs-clear.sh
  :language: sh


3. tuned-adm: latency-performance Profile
=========================================

Overview
--------

The ``tuned`` daemon provides predefined system tuning profiles. The
``latency-performance`` profile generally targets lower latency and reduced
jitter by adjusting CPU governor, power management, kernel scheduler-related
settings, and other system knobs.

Install tuned
-------------

.. code-block:: bash

   sudo apt update
   sudo apt install -y tuned
   sudo systemctl enable --now tuned


Set the profile to latency-performance
--------------------------------------

.. code-block:: bash

    sudo tuned-adm profile latency-performance

Confirm active profile:

.. code-block:: bash

    sudo tuned-adm active

Expected effects
----------------

Common outcomes when using ``latency-performance`` include:

* Lower latency variance (reduced jitter) under load
* More consistent CPU frequency behavior (often favoring performance over power saving)
* Reduced impact from aggressive power management states
* Better tail latency for latency-sensitive PCIe/DMA-driven workloads

.. note::

    The exact changes depend on distribution and tuned version. Review the tuned
    profile contents under ``/usr/lib/tuned/latency-performance/`` if you need a
    precise list of applied knobs.
