.. _SchedulingNpus:

###############
Scheduling NPUs
###############


This page describes how administrator prepares node and user can consume NPU in Kubernetes.


Preparing Node
====================================================

As an administrator, you have to install :ref:`prerequisites <InstallingPrerequisites>` such as driver, firmware on nodes and deploy :ref:`Furiosa Device Plugin <DevicePlugin>`.
Once you have installed it, your cluster exposes Furiosa NPUs as schedulable resources, such as ``furiosa.ai/rngd``.

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
