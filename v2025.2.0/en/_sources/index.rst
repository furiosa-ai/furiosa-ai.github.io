****************************************************
Welcome to Furiosa Docs
****************************************************

Welcome!
FuriosaAI offers a streamlined software stack designed for deep learning model
inference on FuriosaAI NPUs.
This guide covers the entire workflow for creating inference applications,
starting from a PyTorch model, through model quantization, and model serving and
deployment.

.. warning::

   This document is based on the Furiosa SDK |release| (beta 1) version.
   The features and APIs described herein are subject to change in the future.

.. grid:: 2
   :padding: 0


   .. grid-item-card:: 📢 Latest Release |release|
      :link: /whatsnew/index
      :link-type: doc
      :columns: 6

      Stay up to date with the newest features, improvements, and fixes in the latest release.


   .. grid-item-card:: 🚀 Quick Start with Furiosa-LLM
      :link: /getting_started/furiosa_llm
      :link-type: doc
      :columns: 6

      Furiosa-LLM is a high-performance inference engine for LLM models.
      This document explains how to install and use Furiosa-LLM.


.. grid:: 2
   :padding: 0

   .. grid-item-card:: 📋 Roadmap Overview
      :link: overview/roadmap
      :link-type: doc
      :columns: 6

      See what's ahead for FuriosaAI with our planned releases and upcoming features.
      Stay informed on development progress and key milestones.

   .. grid-item-card:: 🤗 Hugging Face Hub
      :link: https://huggingface.co/furiosa-ai
      :link-alt: https://huggingface.co/furiosa-ai
      :columns: 6

      Pre-optimized and pre-compiled models for FuriosaAI NPUs are available on the Hugging Face Hub.
      Check out the latest models and their capabilities.



Overview
-------------------------------------------------

.. figure:: _static/imgs/SW_stack_3D.png
   :alt: FuriosaAI Software Stack
   :width: 960px
   :align: center

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

Furiosa-LLM
-------------------------------------------------
* :ref:`FuriosaLLM`: An introduction to Furiosa-LLM
* :ref:`OpenAIServer`: More details about the OpenAI-compatible server and its features
* :ref:`ModelPreparation`: How to prepare LLM models to be served by Furiosa-LLM
* :ref:`ModelParallelism`: Tensor/Pipeline/Data parallelism in Furiosa-LLM
* :ref:`FuriosaLLMBuildCommand`: Command line tool for building model artifacts
* :ref:`FuriosaLLMReference`: The Python API reference for Furiosa-LLM
* :ref:`FuriosaLLMExamples`: Examples of using Furiosa-LLM

Cloud Native Toolkit
-------------------------------------------------
* :ref:`CloudNativeToolkit`: An overview of the Cloud Native Toolkit
* :ref:`Container`: An overview of Container Support
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
   :caption: Furiosa-LLM

   /furiosa_llm/intro
   /furiosa_llm/furiosa-llm-serve
   /furiosa_llm/model-preparation
   /furiosa_llm/build-artifacts-by-examples
   /furiosa_llm/model-parallelism
   /furiosa_llm/reference
   /furiosa_llm/examples


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Cloud Native Toolkit

   /cloud_native_toolkit/intro
   /cloud_native_toolkit/container
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

   Forums <https://forums.furiosa.ai>
   Customer Support <https://furiosa-ai.atlassian.net/servicedesk/customer/portals/>


.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Other Links

   FuriosaAI Homepage <https://furiosa.ai>
   Furiosa Gen 1 NPU SDK Doc <https://furiosa-ai.github.io/docs/latest/en/>
