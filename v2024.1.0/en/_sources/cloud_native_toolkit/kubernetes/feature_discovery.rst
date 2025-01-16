.. _FeatureDiscovery:

####################################
Installing Furiosa Feature Discovery
####################################


Furiosa Feature Discovery and NFD
================================================================

The Furiosa Feature Discovery automatically labels Kubernetes nodes with information
about FuriosaAI NPU properties, such as the NPU family, count, and driver versions.
Using these labels, you can schedule your Kubernetes workloads based on specific NPU requirements.

The Furiosa Feature Discovery leverage NFD(Node Feature Discovery) which is a tool that detects
hardware features and labels Kubernetes nodes. It is recommended to use NFD and
Furiosa Feature Discovery to ensure that the Cloud Native Toolkit is deployed only on nodes
equipped with FuriosaAI NPUs.


Labels
-----------------------------

The followings are the labels that the Furiosa Feature Discovery attaches and what they mean.

.. list-table:: Labels
   :align: center
   :header-rows: 1
   :widths: 130 160 260

   * - Label
     - Value
     - Description
   * - furiosa.ai/npu.count
     - n
     - # of NPU devices
   * - furiosa.ai/npu.family
     - warboy, rngd
     - Chip family
   * - furiosa.ai/npu.product
     - warboy, rngd, rngd-s, rngd-max
     - Chip product name
   * - furiosa.ai/npu.driver.version
     - x.y.z
     - NPU device driver version
   * - furiosa.ai/npu.driver.version.major
     - x
     - NPU device driver version major part
   * - furiosa.ai/npu.driver.version.minor
     - y
     - NPU device driver version minor part
   * - furiosa.ai/npu.driver.version.patch
     - z
     - NPU device driver version patch part
   * - furiosa.ai/npu.driver.version.metadata
     - abcxyz
     - NPU device driver version metadata


Deploying Furiosa Feature Discovery with Helm
----------------------------------------------
With the helm chart you can easily install Furiosa feature discovery and NFD into your Kubernetes cluster.
Following command shows how to install them.
The Furiosa device plugin helm chart is available at https://github.com/furiosa-ai/helm-charts. To configure deployment as you need, you can modify ``charts/furiosa-feature-discovery/values.yaml``.

.. code-block:: sh

  helm repo add furiosa https://furiosa-ai.github.io/helm-charts
  helm repo update
  helm install furiosa-feature-discovery furiosa/furiosa-feature-discovery -n kube-system
