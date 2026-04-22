.. _LLM-D:

################################
Deploying Furiosa-LLM with llm-d
################################

`llm-d <https://llm-d.ai/>`__ is a Kubernetes-native distributed inference framework that lets you serve LLM models across a cluster. Adopting llm-d as a distributed inference framework can provide the following benefits:

* **Intelligent Inference Scheduling**: llm-d provides a configurable load balancer and pluggable scorers to route serving requests to optimal pods. llm-d provides metrics-based scoring and prefix-cache-aware scorers.
* **Prefill/Decode Disaggregation**: llm-d selects optimal Prefill and Decode pods and relays KV Cache transfers between the designated pods.
* **Wide Expert-Parallelism**: llm-d supports wide expert parallelism to deploy large Mixture-of-Experts (MoE) models.

llm-d integration with Furiosa-LLM
----------------------------------

Furiosa-LLM can be integrated with llm-d to support distributed serving of LLM models using RNGDs.
Currently, the following integrations are supported:

* **Intelligent Inference Scheduling**
  Furiosa-LLM implements Model Server Protocol's `metrics reporting <https://github.com/kubernetes-sigs/gateway-api-inference-extension/tree/main/docs/proposals/003-model-server-protocol#metrics-reporting>`__ to support Intelligent Inference Scheduling. The corresponding metrics are as follows:

  .. list-table:: Model Server Protocol metrics
     :header-rows: 1
     :widths: 25 75

     * - Metric
       - Furiosa-LLM Metric
     * - TotalQueuedRequests
       - ``furiosa_llm_num_requests_waiting``
     * - TotalRunningRequests
       - ``furiosa_llm_num_requests_running``
     * - KVCacheUtilization
       - ``furiosa_llm_kv_cache_usage_percent``
     * - BlockSize
       - | Name: ``furiosa_llm_cache_config_info``
         | Label: ``block_size``
     * - NumGPUBlocks
       - | Name: ``furiosa_llm_cache_config_info``
         | Label: ``num_gpu_blocks``

The following integrations are not currently supported:

* **Precise Prefix-Cache-Aware Scoring**: Furiosa-LLM currently does not implement KV Cache events.
* **Prefill/Decode Disaggregation**: Furiosa-LLM currently does not support prefill/decode disaggregation.
* **Wide Expert-Parallelism**: Furiosa-LLM currently does not support wide expert parallelism.

Deploying Furiosa-LLM with llm-d
--------------------------------

This section describes how to deploy Furiosa-LLM with llm-d. The deployed llm-d will have Intelligent Inference Scheduling enabled, enabling metric-based request routing. This guide is based on llm-d's `Well-lit Path: Intelligent Inference Scheduling <https://github.com/llm-d/llm-d/blob/v0.5.0/guides/inference-scheduling/README.md>`__ guide.

Prerequisites
^^^^^^^^^^^^^

- A Kubernetes cluster equipped with two or more Furiosa RNGD devices.
- Hugging Face account and access token.
- A Kubernetes storage class which supports dynamic volume provisioning.

For detailed instructions on setting up an RNGD cluster, please refer to :ref:`Installing Prerequisites <InstallingPrerequisites>` and :ref:`Kubernetes Plugins <Kubernetes>`.

You will also install Gateway API and a GIE-compatible gateway (Istio) in the steps below.

llm-d-modelservice and Helm repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

llm-d provides the llm-d-modelservice Helm chart to simplify LLM deployment. We provide a fork of that chart to run Furiosa-LLM on RNGDs. Add the Furiosa Helm repository now so it is available when you deploy the model server in Step 5:

.. code-block:: bash

   helm repo add furiosa https://furiosa-ai.github.io/helm-charts
   helm repo update

Step 1: Set up Gateway API CRDs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

llm-d utilizes Kubernetes' `Gateway API Inference Extension (GIE) <https://gateway-api-inference-extension.sigs.k8s.io/>`__. Appropriate CRDs need to be installed in the cluster.

.. code-block:: bash

   # Install Gateway API CRDs
   kubectl apply -f \
       https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml
   # Install Gateway API Inference Extension CRDs
   kubectl apply -f \
       https://github.com/kubernetes-sigs/gateway-api-inference-extension/releases/download/v1.3.0/manifests.yaml

Step 2: Deploy GIE-compatible Gateway
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A list of GIE-compatible gateways can be found `here <https://gateway-api-inference-extension.sigs.k8s.io/implementations/gateways/>`__.
For this guide, we will deploy Istio as the GIE-compatible gateway. Install the base chart first, then the discovery chart (istiod) with GIE enabled.

Add the Istio Helm repository and install istio-base:

.. code-block:: bash

   helm repo add istio https://istio-release.storage.googleapis.com/charts
   helm repo update
   helm install istio-base istio/base -n istio-system \
       --set defaultRevision=default --create-namespace

Install the Istio discovery chart with GIE support:

.. code-block:: yaml
   :caption: istiod.yaml

   meshConfig:
     defaultConfig:
       proxyMetadata:
         ENABLE_GATEWAY_API_INFERENCE_EXTENSION: "true"
   pilot:
     env:
       ENABLE_GATEWAY_API_INFERENCE_EXTENSION: "true"

.. code-block:: bash

   helm install istiod istio/istiod -f /path/to/istiod.yaml -n istio-system --wait

Step 3: Prepare Kubernetes Secret
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will use ``llm-d`` namespace to deploy llm-d with llm-d-modelservice Helm chart.
Create a Kubernetes secret for your Hugging Face token:

.. code-block:: yaml
   :caption: hf-secret.yaml

   apiVersion: v1
   kind: Secret
   metadata:
     name: hf-token-secret
   type: Opaque
   data:
     HF_TOKEN: <your_base64_encoded_hf_token>

Encode your token using the following command:

.. code-block:: bash

   echo -n '<your_HF_TOKEN>' | base64

Then apply the secret:

.. code-block:: bash
   
   kubectl create namespace llm-d
   kubectl apply -f /path/to/hf-secret.yaml -n llm-d

Step 4: Download the Target LLM Model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Furiosa-LLM deployed using llm-d-modelservice can utilize PVCs to pre-download Hugging Face models.
In this guide, we will pre-download the ``furiosa-ai/Llama-3.1-8B-Instruct`` model from Hugging Face Hub.

Create a PVC to store the model:

.. code-block:: yaml
   :caption: model-pvc.yaml

   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: model-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 20Gi
     # The storage class "default" is an example, use an appropriate one for your cluster.
     storageClassName: default


.. code-block:: bash

   kubectl apply -f /path/to/model-pvc.yaml -n llm-d

Create a pod to download the model:

.. code-block:: yaml
   :caption: model-downloader.yaml

   apiVersion: v1
   kind: Pod
   metadata:
     name: model-downloader
   spec:
     volumes:
       - name: model-storage
         persistentVolumeClaim:
           claimName: model-pvc
   
     initContainers:
       - name: download-model
         image: python:3.10-slim
         env:
           - name: HF_TOKEN
             valueFrom:
               secretKeyRef:
                 name: hf-token-secret
                 key: HF_TOKEN
         command:
           - /bin/bash
           - -c
           - >
             pip install --no-cache-dir huggingface_hub &&
             python -c "from huggingface_hub import snapshot_download; snapshot_download(repo_id='furiosa-ai/Llama-3.1-8B-Instruct', cache_dir='/models')"

         volumeMounts:
           - name: model-storage
             mountPath: /models
   
     containers:
       - name: model-consumer
         image: ubuntu
         command: ["sleep", "infinity"]
         volumeMounts:
           - name: model-storage
             mountPath: /models
   
     restartPolicy: Never

.. code-block:: bash

   kubectl apply -f /path/to/model-downloader.yaml -n llm-d

Wait for the model download to complete before proceeding (e.g. check the pod status with ``kubectl get pods -n llm-d`` and logs with ``kubectl logs model-downloader -n llm-d -c download-model``).
For detailed instructions on how to pre-download a model on PVCs, please refer to the `upstream documentation <https://github.com/llm-d-incubation/llm-d-modelservice/blob/llm-d-modelservice-v0.3.18/examples/pvc/README.md>`__.

Step 5: Deploy llm-d
^^^^^^^^^^^^^^^^^^^^

First, we need to deploy the llm-d Inference Scheduler to the namespace using the Helm chart. Prior to deploying the Inference Scheduler, the llm-d-infra Helm chart must be deployed.

.. code-block:: yaml
   :caption: llm-d-infra.yaml

   gateway:
     gatewayClassName: istio

.. code-block:: bash

   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo add llm-d-infra https://llm-d-incubation.github.io/llm-d-infra/
   helm repo update
   helm install llm-d-infra llm-d-infra/llm-d-infra -n llm-d -f /path/to/llm-d-infra.yaml

Then, deploy the llm-d Inference Scheduler using the Helm chart. Set the flags to the respective Model Server Protocol metrics of Furiosa-LLM to enable metric-based request routing.

.. code-block:: yaml
   :caption: inference-scheduler.yaml

   inferenceExtension:
     replicas: 1
     flags:
       cache-info-metric: "furiosa_llm_cache_config_info"
       kv-cache-usage-percentage-metric: "furiosa_llm_kv_cache_usage_percent"
       lora-info-metric: ""
       total-queued-requests-metric: "furiosa_llm_num_requests_waiting"
       total-running-requests-metric: "furiosa_llm_num_requests_running"
     image:
       name: llm-d-inference-scheduler
       hub: ghcr.io/llm-d
       tag: v0.5.0
       pullPolicy: Always
     extProcPort: 9002
     pluginsConfigFile: "default-plugins.yaml"
     monitoring:
       interval: "10s"
       # Prometheus ServiceMonitor will be created when enabled for EPP metrics collection
       secret:
         name: inference-scheduling-gateway-sa-metrics-reader-secret
       prometheus:
         enabled: true
         auth:
           # To allow unauthenticated /metrics access (e.g., for debugging with curl), set to false
           enabled: true
   inferencePool:
     targetPorts:
       - number: 8000
     modelServerType: vllm
     modelServers:
       matchLabels:
         llm-d.ai/inferenceServing: "true"
   provider:
     name: istio
     istio:
       destinationRule:
         host: "gaie-epp.llm-d.svc.cluster.local"

.. code-block:: bash

   helm install gaie \
       oci://registry.k8s.io/gateway-api-inference-extension/charts/inferencepool \
       --version v1.3.0 \
       -n llm-d \
       -f /path/to/inference-scheduler.yaml

llm-d's Inference Scheduler is deployed using GIE's InferencePool helm chart. For further information on customizing the deployment, please refer to the `InferencePool helm chart documentation <https://github.com/kubernetes-sigs/gateway-api-inference-extension/tree/v1.3.0/config/charts/inferencepool>`__.

Finally, deploy the llm-d-modelservice Helm chart to run the Furiosa-LLM model server.

.. code-block:: yaml
   :caption: llm-d-modelservice.yaml

   multinode: false

   modelArtifacts:
     name: "furiosa-ai/Llama-3.1-8B-Instruct"
     uri: "pvc+hf://model-pvc/furiosa-ai/Llama-3.1-8B-Instruct"
     size: 20Gi
     authSecretName: "hf-token-secret"
     labels:
       llm-d.ai/inferenceServing: "true"
       llm-d.ai/model: "Llama-3.1-8B-Instruct"

   accelerator:
     type: "furiosa"

   routing:
     proxy:
       enabled: false
       targetPort: 8000

   prefill:
     create: false
   
   decode:
     parallelism:
       tensor: 8
     replicas: 2
     containers:
       - name: "furiosa-llm"
         image: furiosaai/furiosa-llm:latest
         modelCommand: furiosaLLMServe
         args:
           - --enable-prefix-caching
           - --disable-uvicorn-access-log
         ports:
           - containerPort: 8000
             name: furiosa-llm
             protocol: TCP
         mountModelVolume: true
         startupProbe:
           httpGet:
             path: /v1/models
             port: furiosa-llm
           initialDelaySeconds: 15
           periodSeconds: 30
           timeoutSeconds: 5
           failureThreshold: 60
         livenessProbe:
           httpGet:
             path: /health
             port: furiosa-llm
           periodSeconds: 10
           timeoutSeconds: 5
           failureThreshold: 3
         readinessProbe:
           httpGet:
             path: /v1/models
             port: furiosa-llm
           periodSeconds: 5
           timeoutSeconds: 2
           failureThreshold: 3
   
.. code-block:: bash

   helm install ms furiosa/llm-d-modelservice -n llm-d -f /path/to/llm-d-modelservice.yaml

llm-d-modelservice Helm chart provides more configuration knobs other than the ones shown in the example above. Please refer to the `llm-d-modelservice Helm chart documentation <https://github.com/furiosa-ai/helm-charts/blob/llm-d-modelservice-2026.1.0/charts/llm-d-modelservice/README.md>`__ for details.

Step 6: Expose the service with an HTTPRoute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create an HTTPRoute so that inference requests are routed to the llm-d Inference Scheduler:

.. code-block:: yaml
   :caption: httproute.yaml

   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: llm-d
   spec:
     parentRefs:
       - group: gateway.networking.k8s.io
         kind: Gateway
         name: llm-d-infra-inference-gateway
     rules:
       - backendRefs:
           - group: inference.networking.k8s.io
             kind: InferencePool
             name: gaie
             port: 8000
             weight: 1
         timeouts:
           backendRequest: 0s
           request: 0s
         matches:
           - path:
               type: PathPrefix
               value: /

.. code-block:: bash
   
   kubectl apply -f /path/to/httproute.yaml -n llm-d

After the route is applied, you can send inference requests to the Gateway address (e.g. via the Istio ingress). For details on how to obtain the Gateway URL and call the API, see the llm-d `Inference Against llm-d <https://github.com/llm-d/llm-d/blob/v0.5.0/docs/getting-started-inferencing.md>`__ guide.