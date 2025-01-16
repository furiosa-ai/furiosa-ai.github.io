****************************************************
FuriosaAI Developer Center
****************************************************

Welcome to the FuriosaAI Developer Center.
FuriosaAI provides an streamlined software stack for deep learning model inference on FuriosaAI NPUs.
This document provides a guide to easily perform the entire workflow of writing inference applications,
from starting with PyTorch model to model quantization, serving, and production deployment.

.. warning::

   This document is based on Furiosa SDK |release| (beta0) version,
   and the features and APIs described in this document may change in the future.

.. grid:: 3

    .. grid-item-card:: 📢 Latest Release |release|
      :link: /whatsnew/index
      :link-type: doc

      2024.2.1 is the latest SDK release for RNGD.
      This document provides an overview of the new features and changes in the latest release.
      +++
      :ref:`Learn more <Release2024_2_1>`

    .. grid-item-card:: 🚀 Quick Start with Furiosa LLM
      :link: /getting_started/furiosa_llm
      :link-type: doc

      Furiosa LLM is a high-performance inference engine for LLM models.
      This document explains how to install and use Furiosa LLM.
      +++
      :ref:`Learn more <GettingStartedFuriosaLLM>`

    .. grid-item-card:: 📊 Running MLPerf Benchmark
      :link: getting_started/furiosa_mlperf
      :link-type: doc

      This document describes how to reproduce the MLPerf™ Inference Benchmark
      using the FuriosaAI Software Stack.
      +++
      :ref:`Learn more <GettingStartedFuriosaMLPerf>`


Overview
-------------------------------------------------
* :ref:`RNGD`: RNGD Hardware Specification, and features
* :ref:`SoftwareStack`: An overview of the FuriosaAI software stack
* :ref:`SupportedModels`: A list of supported models
* :ref:`WhatsNew`: New features and changes in the latest release
* :ref:`RoadMap`: The future roadmap of FuriosaAI Software Stack

Getting Started
-------------------------------------------------
* :ref:`InstallingPrerequisites`: How to install the prerequisites for FuriosaAI Software Stack
* :ref:`UpgradeGuide`: How to upgrade the FuriosaAI Software Stack
* :ref:`GettingStartedFuriosaLLM`
* :ref:`GettingStartedFuriosaMLPerf`

Furiosa LLM
-------------------------------------------------
* :ref:`FuriosaLLM`: An introduction to Furiosa LLM
* :ref:`OpenAIServer`: More details about the OpenAI-compatible server and its features
* :ref:`ModelPreparationWorkflow`: How to quantize and compile Huggingface models
* :ref:`ModelParallelism`: Tensor/Pipeline/Data parallelism in Furiosa LLM
* :ref:`FuriosaLLMReference`: The Python API reference for Furiosa LLM

Cloud Native Toolkit
-------------------------------------------------
* :ref:`CloudNativeToolkit`: An overview of the Cloud Native Toolkit
* :ref:`Kubernetes`: An overview of the Kubernetes Support

Device Management
-------------------------------------------------
* :ref:`FuriosaSMICLI`: A command line utility for managing FuriosaAI NPUs
* :ref:`FuriosaSMILIB`: A library for managing FuriosaAI NPUs

Customer Support
--------------------------------------------------
* `FuriosaAI Forum <https://forums.furiosa.ai>`_
* `FuriosaAI Customer Portal <https://furiosa-ai.atlassian.net/servicedesk/customer/portals/>`_

Other Links
--------------------------------------------------
* `FuriosaAI Home <https://furiosa.ai>`_
* `FuriosaAI Warboy SDK Document <https://furiosa-ai.github.io/docs/latest/en/>`_



.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Overview

   /overview/rngd
   /overview/software_stack
   /overview/supported_models
   /whatsnew/index
   /overview/roadmap


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Getting Started

   /getting_started/prerequisites
   /getting_started/furiosa_llm
   /getting_started/furiosa_mlperf
   /getting_started/upgrade_guide

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Furiosa LLM

   /furiosa_llm/intro
   /furiosa_llm/furiosa-llm-serve
   /furiosa_llm/model-preparation-workflow
   /furiosa_llm/model-parallelism
   /furiosa_llm/references


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Cloud Native Toolkit

   /cloud_native_toolkit/intro
   /cloud_native_toolkit/kubernetes

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Device Management

   /device_management/system_management_interface


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Customer Support

   FuriosaAI Forum <https://forums.furiosa.ai>
   FuriosaAI Customer Portal <https://furiosa-ai.atlassian.net/servicedesk/customer/portals/>


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Other Links

   FuriosaAI Homepage <https://furiosa.ai>
   FuriosaAI Warboy SDK Document <https://furiosa-ai.github.io/docs/latest/en/>
