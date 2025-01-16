.. _RNGD:

**********************************
FuriosaAI RNGD
**********************************

FuriosaAI's second-generation Neural Processing Unit (NPU), RNGD, is a chip designed for deep learning inference,
supporting high-performance Large Language Models (LLM), Multi-Modal LLM, Vision models,
and other deep learning models.

.. figure:: ../_static/imgs/rngd_card.webp
  :alt: FuriosaAI RNGD
  :class: only-dark
  :width: 650px
  :align: center

.. figure:: ../_static/imgs/rngd_card.webp
  :alt: FuriosaAI RNGD
  :class: only-light
  :width: 650px
  :align: center


RNGD is based the Tensor Contraction Processor (TCP) architecture which
utilizes TSMC's 5nm process node, and operates at 1.0 GHz. It offers 512 TOPS and 1024 TOPS of INT8 and INT4
performance respectively. RNGD is configured with two HBM3 modules providing a memory bandwidth of 1.5 TB/s,
and supports PCIe Gen5 x16. For multi-tenant environments like Kubernetes and virtual environment,
a single RNGD chip can work as 2, 4, 8 individual NPUs, each fully isolated with its own cores and memory bandwidth.
RNGD supports Single Root IO Virtualization (SR-IOV) and virtualization for multi-instance NPUs.

Please refer to the followings to learn more about TCP architecture and RNGD:

* `TCP: A Tensor Contraction Processor for AI Workloads, ACM/IEEE ISCA 2024 <https://ieeexplore.ieee.org/document/10609575>`_ (`PDF <https://furiosa.ai/download/FuriosaAI-tensor-contraction-processor-isca24>`_)
* `FuriosaAI RNGD: A Tensor Contraction Processor for Sustainable AI Computing, Hotchips 2024 <https://hc2024.hotchips.org/#clip=8jnhm5vdlsow>`_
* `Tensor Contraction Processor: The first future-proof AI chip architecture <https://furiosa.ai/blog/tensor-contraction-processor-ai-chip-architecture>`_


.. list-table:: RNGD Hardware Specification
   :align: center
   :widths: 200 300

   * - Architecture
     - Tensor Contraction Processor
   * - Process Node
     - TSMC 5nm
   * - Frequency
     - 1.0GHz
   * - BF16
     - 256TFLOPS
   * - FP8
     - 512TFLOPS
   * - INT8
     - 512TOPS
   * - INT4
     - 1024TOPS
   * - Memory Bandwidth
     - HBM3 1.5TB/s
   * - Memory Capacity
     - HBM3 48GB
   * - On-Chip SRAM
     - 256MB
   * - Interconnect Interface
     - PCIe Gen5 x16
   * - Thermal Solution
     - Passive
   * - Thermal Design Power (TDP)
     - 150W
   * - Power Connector
     - 12VHPWR
   * - Form Factor
     - PCIe dual-slot full-height 3/4 Length
   * - Multi-Instance Support
     - 8
   * - Virtualization Support
     - Yes
   * - SR-IOV
     - 8 Virtual Functions
   * - ECC Memory Support
     - Yes
   * - Secure Boot with Root of Trust
     - Yes

