.. _DevicePlugin:

################################
Installing Furiosa Device Plugin
################################


Furiosa Device Plugin
================================================================
The Furiosa device plugin implements the `Kubernetes Device Plugin <https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/>`_
interface for FuriosaAI NPU devices, and its features are as follows:

* Discovering the Furiosa NPU devices and registeriing to a Kubernetes cluster.
* Tracking the health of the devices and reporting to a Kubernetes cluster.
* Running AI workload on the top of the Furiosa NPU devices within a Kubernetes cluster.

Configuration
----------------------------------------------
The Furiosa NPU can be integrated into the Kubernetes cluster in various configurations.
A single NPU card can either be exposed as a single resource or partitioned into multiple resources.
Partitioning into multiple resources allows for more granular control.

The configuration structure is as follows:

.. code-block:: yaml

  config:
    resourceStrategy: generic
    debugMode: false
    disabledDeviceUUIDListMap:

`resourceStrategy` defines the resource unit of NPU scheduling in the cluster. The following table shows the available resource strategy:

.. list-table::
   :align: center
   :widths: 200 200 200
   :header-rows: 1

   * - NPU Configuration
     - Resource Name
     - Resource Count Per Card
   * - generic
     - furiosa.ai/rngd
     - 1
   * - single-core
     - furiosa.ai/rngd-1core.6gb
     - 8
   * - dual-core
     - furiosa.ai/rngd-2core.12gb
     - 4
   * - quad-core
     - furiosa.ai/rngd-4core.24gb
     - 2

`debugMode` enables or disables debug mode. The default value is `false`.

`disabledDeviceUUIDListMap` allows disabling specific devices on a per-node basis. This is structured as follows:

.. code-block:: yaml

  disabledDeviceUUIDListMap:
    node_a:
      - "uuid1"
      - "uuid2"
    node_b:
      - "uuid3"
      - "uuid4"

If `disabledDeviceUUIDListMap` is not configured, all devices are enabled by default.


Deploying Furiosa Device Plugin with Helm
-----------------------------------------

The Furiosa device plugin helm chart is available at https://github.com/furiosa-ai/helm-charts. To configure deployment as you need, you can modify ``charts/furiosa-device-plugin/values.yaml``.

* If resourceStrategy is not specified, the default value is ``"generic"``.
* If debugMode is not specified, the default value is ``false``.
* If disabledDeviceUUIDListMap is not specified, the default value is empty list ``[]``.

You can deploy the Furiosa Device Plugin by running the following commands:

.. code-block:: sh

  helm repo add furiosa https://furiosa-ai.github.io/helm-charts
  helm repo update
  helm install furiosa-device-plugin furiosa/furiosa-device-plugin -n kube-system

