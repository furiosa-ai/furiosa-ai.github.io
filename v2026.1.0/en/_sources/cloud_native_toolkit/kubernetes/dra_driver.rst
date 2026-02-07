Installing Furiosa DRA Driver
=============================

.. note::

   Furiosa DRA Driver is in alpha stage, which means its features are experimental and may change in future releases.

   Users should be aware that the driver may not be fully stable or feature-complete, and they should exercise caution when deploying it in production environments.


Furiosa DRA Driver
------------------

The Furiosa DRA Driver implements the
`Kubernetes DRA <https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/>`_ interface for
FuriosaAI NPU devices, and its features are as follows:

- Discovering the FuriosaAI NPU devices and registering to a Kubernetes cluster as DRA resources.
- Tracking the health of the devices and reporting to a Kubernetes cluster.
- Running AI workload on the top of the Furiosa NPU devices within a Kubernetes cluster.

Prerequisites
-------------

The Furiosa DRA Driver requires the following prerequisites:

- Kubernetes **1.34+** is required.

  - In this case, `CDI <https://github.com/cncf-tags/container-device-interface>`_
    must be enabled on each node.
  - Starting from `containerd <https://containerd.io/>`_ **2.0**, CDI is enabled by default.

Deploying Furiosa DRA Driver with Helm
--------------------------------------

The Furiosa DRA Driver Helm chart is available at
https://github.com/furiosa-ai/helm-charts.

To configure deployment as you need, you can modify
``charts/furiosa-dra-driver/values.yaml``.

You can deploy the Furiosa DRA Driver by running the following commands:

.. code-block:: shell

   helm repo add furiosa https://furiosa-ai.github.io/helm-charts
   helm repo update
   helm install furiosa-dra-driver furiosa/furiosa-dra-driver -n furiosa-system

When DRA Driver is successfully deployed, you can see the registered FuriosaAI
NPU resources by running the following command:

.. code-block:: shell

   kubectl get resourceslices

.. code-block:: yaml

   - apiVersion: resource.k8s.io/v1
     kind: ResourceSlice
     metadata:
       creationTimestamp: "2026-01-12T02:57:12Z"
       generateName: cntk002-npu.furiosa.ai-
       generation: 1
       name: cntk002-npu.furiosa.ai-stczm
       ownerReferences:
         - apiVersion: v1
           controller: true
           kind: Node
           name: cntk002
           uid: 0a0a5366-cb39-4e8d-9aae-56a885709887
       resourceVersion: "8934931"
       uid: 8909a748-d798-4e98-b738-5c08d771e011
     spec:
       devices:
         - attributes:
             arch:
               string: rngd
             driverVersion:
               version: 2025.4.0-dev2+0e9e56a
             index:
               int: 0
             numaNode:
               int: 0
             pcieBusID:
               string: "0000:17:00.0"
             resource.kubernetes.io/pcieRoot:
               string: "pci0000:16"
             uuid:
               string: 09512C86-0709-4404-8943-414B404A4F44
           capacity:
             core:
               value: "8"
             memory:
               value: 48Gi
           name: npu0
         - attributes:
             arch:
               string: rngd
             driverVersion:
               version: 2025.4.0-dev2+0e9e56a
             index:
               int: 1
             numaNode:
               int: 0
             pcieBusID:
               string: "0000:43:00.0"
             resource.kubernetes.io/pcieRoot:
               string: "pci0000:42"
             uuid:
               string: B65D0C77-7333-4D90-AC82-3E22EE5F88EE
           capacity:
             core:
               value: "8"
             memory:
               value: 48Gi
           name: npu1
       driver: npu.furiosa.ai
       nodeName: cntk002
       pool:
         generation: 1
         name: cntk002
         resourceSliceCount: 1


Available attributes
--------------------

Below table lists available attributes for FuriosaAI NPU devices.

You need to attach ``npu.furiosa.ai`` at the front of each attribute key when you use them in ``ResourceClaimTemplate``.

- Only exception is ``resource.kubernetes.io/pcieRoot``, which is a standard attribute defined by Kubernetes.

.. list-table::
   :align: center
   :header-rows: 1
   :widths: 130 160 260

   * - Attribute
     - Type
     - Description
   * - ``index``
     - int
     - The index of the NPU device
   * - ``uuid``
     - string
     - The UUID of the NPU device
   * - ``arch``
     - string
     - The architecture of the NPU device
   * - ``driverVersion``
     - string
     - The driver version represented in semver
   * - ``numaNode``
     - int
     - The NUMA node value that NPU device belongs to
   * - ``pcieBusID``
     - string
     - The PCIe bus ID of the NPU device
   * - ``resource.kubernetes.io/pcieRoot``
     - string
     - The pcieRoot value of the NPU device


Example Usage
-------------

Request single NPU
~~~~~~~~~~~~~~~~~~

Below example shows how to request single FuriosaAI NPU in a Pod spec with ``ResourceClaimTemplate``.

First, create a ``ResourceClaimTemplate`` that defines a request for a single NPU device.
Then, create a Pod that references the template to claim the NPU resource.

.. code-block:: yaml

    apiVersion: resource.k8s.io/v1
    kind: ResourceClaimTemplate
    metadata:
     name: single-npu
    spec:
     spec:
       devices:
         requests:
           - name: npu
             exactly:
               count: 1
               deviceClassName: npu.furiosa.ai


.. code-block:: yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: furiosa-llm
    spec:
      containers:
        - name: furiosa-llm
          image: furiosaai/furiosa-llm:latest
          command:
            - sleep
            - "86400"
          env:
            - name: HF_TOKEN
              valueFrom:
                secretKeyRef:
                  key: token
                  name: hf-token-secret
          resources:
            claims:
              - name: npu
      resourceClaims:
        - name: npu
          resourceClaimTemplateName: single-npu
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: hf-token-secret
    type: Opaque
    data:
      token: <your_base64_encoded_hf_token>


Request quad NPU on same NUMA node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Similar to the previous example, below example shows how to request four FuriosaAI NPUs
that are located on the same NUMA node in a Pod spec.

First, create a ``ResourceClaimTemplate`` that defines a request for four FuriosaAI NPUs on the same NUMA node.
Then, create a Pod that references the template to claim the NPU resource.

.. code-block:: yaml

    apiVersion: resource.k8s.io/v1
    kind: ResourceClaimTemplate
    metadata:
      name: quad-npu-with-same-numa-node
    spec:
      spec:
        devices:
          constraints:
            - matchAttribute: npu.furiosa.ai/numaNode
          requests:
            - name: npu
              exactly:
                count: 4
                deviceClassName: npu.furiosa.ai


.. code-block:: yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: furiosa-llm
    spec:
      containers:
        - name: furiosa-llm
          image: furiosaai/furiosa-llm:latest
          command:
            - sleep
            - "86400"
          env:
            - name: HF_TOKEN
              valueFrom:
                secretKeyRef:
                  key: token
                  name: hf-token-secret
          resources:
            claims:
              - name: npu
      resourceClaims:
        - name: npu
          resourceClaimTemplateName: quad-npu-with-same-numa-node
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: hf-token-secret
    type: Opaque
    data:
      token: <your_base64_encoded_hf_token>

