.. _Kubernetes:

####################################
Kubernetes Plugins
####################################

.. _KubernetesVersionInfo:

Kubernetes Support
=========================================

We do support the following versions of Kubernetes and CRI runtime:

* Kubernetes: v1.24.0 or later
* helm v3.0.0 or later
* CRI Runtime: `containerd <https://github.com/containerd/containerd>`_ or `CRI-O <https://github.com/cri-o/cri-o>`_

.. note::

  Docker is officially deprecated as a container runtime in Kubernetes.
  It is recommended to use containerd or CRI-O as a container runtime.
  Otherwise you may face unexpected issues with NPU plugins.
  For more information, see `here <https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/>`_.


NPU Plugins Deployment Guide
=========================================

As an administrator, you have to install :ref:`prerequisites <InstallingPrerequisites>` such as driver, firmware on NPU nodes.
If prerequisites are not installed, the NPU plugins will return an error.

We recommend to deploy NPU plugins using Helm charts. Since it is difficult to control deployment of each plugin individually.
If NPU plugins are deployed on non-NPU nodes, the plugins will return not supported errors and the pods will be in a CrashLoopBackOff state.

The helm chart deploy the Node Feature Discovery (NFD) by default.
Once NFD is deployed, NPU nodes are automatically labelled with PCI IDs and capabilities.
Here is an example of the labels that are attached to the NPU node:

.. code-block:: sh

  feature.node.kubernetes.io/pci-1200_1ed2.present: "true"
  feature.node.kubernetes.io/pci-1200_1ed2.sriov.capable: "true"

The NPU Plugins helm chart set nodeAffinity like below to control the deployment of NPU plugins only on NPU nodes.

.. code-block:: sh

    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: feature.node.kubernetes.io/pci-1200_1ed2.present
              operator: In
              values:
                - "true"

.. note::

  If you already have NFD deployed in your cluster, you can configure helm-chart to skip NFD deployment.
  However, you need to ensure that NFD has default label policy. Default label policy includes PCI class and Vendor ID fields in the label.

  .. code-block:: sh

    feature.node.kubernetes.io/pci_<CLASS_ID>_<VENDOR_ID>.present


  In some cases, NFD is configured to omit PCI class field in the label.

  .. code-block:: sh

    feature.node.kubernetes.io/pci_<VENDOR_ID>.present

  You have two options to resolve this issue: either deploy two NFD instances in the cluster with different label policies,
  managing them using taint and toleration,or modify the NPU plugins Helm chart to use the correct label.


See the following links for more information and configurations of each plugin:

.. toctree::
   :maxdepth: 1

   /cloud_native_toolkit/kubernetes/feature_discovery
   /cloud_native_toolkit/kubernetes/device_plugin
   /cloud_native_toolkit/kubernetes/dra_driver
   /cloud_native_toolkit/kubernetes/metrics_exporter
   /cloud_native_toolkit/kubernetes/npu_operator

..


Once you have installed the NPU plugins, your cluster exposes Furiosa NPUs as schedulable resources, such as ``furiosa.ai/rngd``.

To ensure your node is ready, you can examine Capacity and/or Allocatable field of ``v1.node`` object.
Here is an example of node that has 2 RNGD NPUs:

.. code-block:: yaml

  ...
  status:
    ...
    allocatable:
      cpu: "20"
      ephemeral-storage: "1770585791219"
      furiosa.ai/rngd: "2"
      hugepages-1Gi: "0"
      hugepages-2Mi: "0"
      memory: 527727860Ki
      pods: "110"
    capacity:
      cpu: "20"
      ephemeral-storage: 1921208544Ki
      furiosa.ai/rngd: "2"
      hugepages-1Gi: "0"
      hugepages-2Mi: "0"
      memory: 527830260Ki
      pods: "110"
  ...

The following command should show the ``Capacity`` field of each node in the Kubernetes cluster.

.. code-block:: sh

  kubectl get nodes -o json | jq -r '.items[] | .metadata.name as $name | .status.capacity | to_entries | map("    \(.key): \(.value)") | $name + ":\n  capacity:\n" + join("\n")'

Requesting NPUs
====================================================

You can consume NPUs from your containers in a Pod by requesting NPU resources, the same way you request CPU or memory.

However, since NPUs are exposed as a custom resource, there are some limitations you should be aware of when requesting NPU resources:

- You can specify NPU ``limits`` without specifying ``requests``, because kubernetes will use limit as request if request is not specified.
- You can specify NPU in both ``limits`` and ``requests`` but these two values must be equal.
- You cannot specify NPU ``request`` without specifying ``limits``.

Here is an example manifest for a Pod that requests 2 RNGD NPUs:

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: example-npu-request
  spec:
    containers:
    - name: furiosa
      image: furiosaai/furiosa-smi:latest
      imagePullPolicy: IfNotPresent
      command: ["sleep"]
      args: ["120"]
      resources:
        limits:
          furiosa.ai/rngd: 2


Scheduling NPUs With Specific Requirements
====================================================

In certain cases, user may need to schedule NPU workload on node that meet specific hardware or software requirements, such as particular driver versions.
If the :ref:`Furiosa Feature Discovery<FeatureDiscovery>` is deployed in your cluster, nodes are automatically labelled based on their hardware and software configurations including driver version.
This allows user to schedule Pod on nodes that meet specific requirements.

Following example shows how to use affinity to schedule a Pod that request 2 RNGD NPUs with specific driver version:

.. code-block:: yaml

  apiVersion: v1
  kind: Pod
  metadata:
    name: example-npu-scheduling-with-affinity
  spec:
    containers:
    - name: furiosa
      image: furiosaai/furiosa-smi:latest
      imagePullPolicy: IfNotPresent
      command: ["sleep"]
      args: ["120"]
      resources:
        limits:
          furiosa.ai/rngd: 2
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: furiosa.ai/driver-version
              operator: In
              values:
              - "1.0.12"

Known Issues
=========================================

* A pod that uses an external device, like an NPU, may not restart and will end up in an UnexpectedAdmissionError when the node is restarted. This happens because the kubelet has a race condition when allocating external resources to the restarting pod. The issue is tracked in `kubernetes/kubernetes#128043 <https://github.com/kubernetes/kubernetes/issues/128043>`_
