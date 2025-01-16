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

The following table shows the available resource strategy:


.. list-table:: Resource Strategy
   :align: center
   :widths: 200 200 200
   :header-rows: 1

   * - NPU Configuration
     - Resource Name
     - Resource Count Per Card
   * - legacy
     - beta.furiosa.ai/npu
     - 1
   * - generic
     - furiosa.ai/rngd
     - 1

The helm chart of Furiosa device plugin is available at https://github.com/furiosa-ai/helm-charts.

Following shows default values of the helm chart.

.. code-block:: yaml

  config:
    resourceStrategy: generic
    debugMode: false
    disabledDeviceUUIDListMap:

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

