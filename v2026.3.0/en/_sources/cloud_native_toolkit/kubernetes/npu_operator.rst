Installing Furiosa NPU Operator
================================

.. note::

   Furiosa NPU Operator is in **alpha** stage. Its features are experimental and may
   change in future releases.

   The operator may not be fully stable or feature-complete. Use caution when
   deploying it in production environments.

Furiosa NPU Operator
--------------------

The **Furiosa NPU Operator** is a Kubernetes operator that automates the deployment
and management of FuriosaAI NPU devices within a Kubernetes cluster. It deploys and
configures the drivers and cloud-native components required to use FuriosaAI NPUs in
containerized workloads.

Managed components
------------------

The following components are managed by the Furiosa NPU Operator:

- Furiosa Feature Discovery
- Furiosa Device Plugin
- Furiosa DRA Driver
- Furiosa Metrics Exporter
- Furiosa System Manager

Prerequisites
-------------

The Furiosa NPU Operator requires the following prerequisites:

- Deploying `Node Feature Discovery <https://github.com/kubernetes-sigs/node-feature-discovery>`_
  in the cluster is recommended to discover FuriosaAI NPU devices.

  - Nodes with FuriosaAI NPU devices must have the label
    ``feature.node.kubernetes.io/pci-1200_1ed2.present: "true"``.

- Kubernetes **1.34+** is required if you want to use Furiosa DRA Driver.

  - In this case, `CDI <https://github.com/cncf-tags/container-device-interface>`_
    must be enabled on each node.
  - Starting from `containerd <https://containerd.io/>`_ **2.0**, CDI is enabled by default.

Deploying Furiosa NPU Operator with Helm
----------------------------------------

The Furiosa NPU Operator Helm chart is available at:
https://github.com/furiosa-ai/helm-charts

To configure deployment, modify:
``charts/furiosa-npu-operator/values.yaml``

Deploy the operator:

.. code-block:: shell

   helm repo add furiosa https://furiosa-ai.github.io/helm-charts
   helm repo update
   helm install furiosa-npu-operator furiosa/furiosa-npu-operator \
     -n furiosa-system --create-namespace

After the Furiosa NPU Operator is successfully deployed, create a
``FuriosaClusterConfig`` Custom Resource to manage cloud-native components for
FuriosaAI NPU devices.

Example usage
-------------

The example below creates a ``FuriosaClusterConfig`` resource to deploy managed
components.

.. code-block:: yaml

   apiVersion: furiosa.ai/v1alpha1
   kind: FuriosaClusterConfig
   metadata:
     name: example
   spec:
     featureDiscovery:
       enabled: true
       registry: docker.io/furiosaai/furiosa-ai
       tag: 2026.1.0-rc0
       image: furiosa-feature-discovery
     devicePlugin:
       enabled: false
       registry: docker.io/furiosaai/furiosa-ai
       tag: 2026.1.0-rc0
       image: furiosa-device-plugin
     draDriver:
       enabled: true
       registry: docker.io/furiosaai/furiosa-ai
       tag: 2026.1.0-rc0
       image: furiosa-dra-driver
     metricsExporter:
       enabled: true
       registry: docker.io/furiosaai/furiosa-ai
       tag: 2026.1.0-rc0
       image: furiosa-metrics-exporter
       config:
         serviceType: ClusterIP
         servicePort: 6254
         enableScrapeAnnotations: true
         collectInterval: 10
     systemManager:
       enabled: true
       registry: docker.io/furiosaai/furiosa-ai
       tag: 2026.1.0-rc0
       installerImagePrefix: furiosa-system-manager-installer
       validatorImage: furiosa-system-manager-validator
       enableFirmwareInstall: false
       enableDriverInstall: true
       upgradePolicy:
         autoUpgrade: false

Creating the above ``FuriosaClusterConfig`` resource will deploy Furiosa Feature
Discovery, Furiosa DRA Driver, Furiosa Metrics Exporter, and Furiosa System
Manager on nodes where FuriosaAI NPUs are present.

Configuration for ``furiosa-metrics-exporter``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can configure ``furiosa-metrics-exporter`` by modifying the ``config`` field
under the ``metricsExporter`` section in ``FuriosaClusterConfig``.

Available options:

- ``serviceType``: Type of Kubernetes Service to create. Default: ``ClusterIP``.
- ``servicePort``: Port the metrics exporter listens on. Default: ``6254``.
- ``enableScrapeAnnotations``: If ``true``, adds Prometheus scrape annotations to
  the metrics exporter Pod. Default: ``true``.
- ``collectInterval``: Interval (in seconds) at which metrics are collected.
  Default: ``10``.

Selecting nodes for deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify which nodes to deploy the managed components to by setting
``nodeSelector`` in ``FuriosaClusterConfig``.

Example: deploy only on nodes labeled ``furiosa.ai/npu: "true"``:

.. code-block:: yaml

   apiVersion: furiosa.ai/v1alpha1
   kind: FuriosaClusterConfig
   metadata:
     name: example
   spec:
     nodeSelector:
       furiosa.ai/npu: "true"
     # ... other configurations ...

You can create multiple ``FuriosaClusterConfig`` resources, but each node can only
be managed by **one** ``FuriosaClusterConfig``.

If multiple ``FuriosaClusterConfig`` resources match the same node, they will all
be marked with the ``nodeCoverageCollision`` condition and reconciliation will stop.

Affinity for managed components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can set ``affinity`` for managed components in ``FuriosaClusterConfig``.

- The ``affinity`` setting is applied to all managed components.

Default ``affinity``:

.. code-block:: yaml

   apiVersion: furiosa.ai/v1alpha1
   kind: FuriosaClusterConfig
   metadata:
     name: example
   spec:
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: feature.node.kubernetes.io/pci-1200_1ed2.present
                   operator: In
                   values:
                     - "true"
     # ... other configurations ...

If you do not set ``affinity``, the operator applies the default configuration above.

To override ``affinity``, set your own configuration. To clear affinity entirely,
set it to an empty object:

.. code-block:: yaml

   apiVersion: furiosa.ai/v1alpha1
   kind: FuriosaClusterConfig
   metadata:
     name: example
   spec:
     affinity: {}
     # ... other configurations ...

Common configuration for managed components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can configure common options for these managed components:

- ``featureDiscovery``
- ``devicePlugin``
- ``draDriver``
- ``metricsExporter``

Common options:

- ``enabled``: Enable/disable the component. Default: ``false``.
- ``registry``: Container image registry. Default: ``docker.io/furiosaai``.
- ``image``: Container image name. Default depends on the component
  (e.g., ``furiosa-feature-discovery``).
- ``tag``: Container image tag. **Required.**
- ``imagePullPolicy``: Image pull policy. Default: ``IfNotPresent``.
- ``imagePullSecrets``: Image pull secrets.
- ``resources``: Resource requests and limits.
- ``tolerations``: Tolerations.
- ``updateStrategy``: DaemonSet update strategy.

Furiosa System Manager configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure Furiosa System Manager under the ``systemManager`` section.

Available options:

- ``enabled``: Enable/disable the component. Default: ``false``.
- ``registry``: Container image registry. Default: ``docker.io/furiosaai``.
- ``installerImagePrefix``: Installer image name prefix.
  Default: ``furiosa-system-manager-installer``.
- ``validatorImage``: Validator image name.
  Default: ``furiosa-system-manager-validator``.
- ``tag``: Container image tag. **Required.**
- ``enableDriverInstall``: Enable driver installation. Default: ``true``.
- ``enableFirmwareInstall``: Enable firmware installation. Default: ``false``.
- ``imagePullPolicy``: Image pull policy. Default: ``IfNotPresent``.
- ``imagePullSecrets``: Image pull secrets.
- ``resources``: Resource requests and limits.
- ``tolerations``: Tolerations.
- ``upgradePolicy``: Upgrade policy.

  - ``autoUpgrade``: Enable automatic upgrade. Default: ``false``.
  - ``maxParallelUpgrades``: Max nodes upgraded in parallel. Default: ``1``.
  - ``maxUnavailable``: Max unavailable during upgrade. Default: ``25%``.


Important note: Using Device Plugin and DRA Driver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

FuriosaAI NPUs can be managed using either the Device Plugin or the DRA Driver.

However, using both simultaneously on the same node is not supported and may lead to conflicts.

If both are enabled in ``FuriosaClusterConfig``, the operator will set the
``invalidSpec`` condition on the resource and stop reconciliation.

Important note: ``DeviceClass`` secondary resource
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you enable the DRA Driver in ``FuriosaClusterConfig``, a ``DeviceClass`` resource
is created with a fixed name: ``npu.furiosa.ai``.

Example:

.. code-block:: yaml

   apiVersion: resource.k8s.io/v1
   kind: DeviceClass
   metadata:
     name: npu.furiosa.ai
     labels:
       app.kubernetes.io/instance: npu.furiosa.ai
       app.kubernetes.io/managed-by: furiosa-npu-operator
       app.kubernetes.io/name: furiosa-dra-driver
     ownerReferences:
       - apiVersion: furiosa.ai/v1alpha1
         kind: FuriosaClusterConfig
         name: example
         uid: 78ddc742-2c01-4201-bdca-306d8f8931d9
   spec:
     selectors:
       - cel:
           expression: device.driver == 'npu.furiosa.ai'

If multiple ``FuriosaClusterConfig`` resources enable the DRA Driver, the existing
``DeviceClass`` is not modified except for ``metadata.ownerReferences``.

Example with two owners:

.. code-block:: yaml

   apiVersion: resource.k8s.io/v1
   kind: DeviceClass
   metadata:
     name: npu.furiosa.ai
     labels:
       app.kubernetes.io/instance: npu.furiosa.ai
       app.kubernetes.io/managed-by: furiosa-npu-operator
       app.kubernetes.io/name: furiosa-dra-driver
     ownerReferences:
       - apiVersion: furiosa.ai/v1alpha1
         kind: FuriosaClusterConfig
         name: example
         uid: 31553189-c4be-4865-be51-0f5ea29387f4
       - apiVersion: furiosa.ai/v1alpha1
         kind: FuriosaClusterConfig
         name: example2
         uid: eeecf263-5468-4b5f-b298-ad2419023009
   spec:
     selectors:
       - cel:
           expression: device.driver == 'npu.furiosa.ai'
