.. _K8sFuriosaLLMDeployment:

****************************************************
Deploying Furiosa-LLM on Kubernetes
****************************************************

This guide describes how to deploy Furiosa-LLM—an OpenAI-compatible server optimized for Furiosa AI RNGDs—on Kubernetes.

Prerequisites
-------------

- A Kubernetes cluster equipped with Furiosa RNGD devices.
- Hugging Face account and access token.
- A Kubernetes storage class which supports dynamic volume provisioning.

For detailed instructions on setting up an RNGD cluster, please refer to :ref:`Installing Prerequisites<InstallingPrerequisites>` and :ref:`Kubernetes Plugins<Kubernetes>`.

Step 1: Prepare Kubernetes Secret
---------------------------------

Create a Kubernetes secret for your Hugging Face token:

.. code-block:: yaml

   apiVersion: v1
   kind: Secret
   metadata:
     name: hf-token-secret
   type: Opaque
   data:
     token: <your_base64_encoded_hf_token>

Encode your token using the following command:

.. code-block:: bash

   echo -n '<your_HF_TOKEN>' | base64

Then apply the secret:

.. code-block:: bash

   kubectl apply -f secret.yaml


Step 2: Create Persistent Volume Claim (PVC)
--------------------------------------------

.. note::

   Using ephemeral storage inside a Pod for large files, such as LLM model caches, may result in eviction due to disk pressure.
   Thus, using a Persistent Volume Claim (PVC) is highly recommended to ensure data durability and stability.

Create a PVC manifest to store models.
The storage class specified below ("default") is an example, and the actual available storage class in your Kubernetes environment may vary.
Please verify and specify the appropriate storage class for your cluster:

.. code-block:: yaml

   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: llama-storage
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
     # The storage class "default" is an example, use an appropriate one for your cluster.
     storageClassName: default

Apply the PVC:

.. code-block:: bash

   kubectl apply -f pvc.yaml


Step 3: Deploy Furiosa-LLM Server
---------------------------------

Create a Deployment manifest for serving the ``furiosa-ai/Llama-3.1-8B-Instruct-FP8`` model with Furiosa-LLM.
For detailed information about the ``furiosa-ai/Llama-3.1-8B-Instruct-FP8`` model, please see the `Hugging Face model page <https://huggingface.co/furiosa-ai/Llama-3.1-8B-Instruct-FP8>`_.

.. code-block:: yaml

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: llama-3-8b
     labels:
       app: llama-3-8b
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: llama-3-8b
     template:
       metadata:
         labels:
           app: llama-3-8b
       spec:
         containers:
           - name: llama-3-8b
             image: furiosaai/furiosa-llm:latest
             args:
               - "serve"
               - "furiosa-ai/Llama-3.1-8B-Instruct-FP8"
             ports:
               - containerPort: 8000
             resources:
               # Recommended resources for one RNGD card: 10 CPU cores and 100GB memory
               limits:
                 cpu: 10
                 memory: 100Gi
                 furiosa.ai/rngd: "1"
               requests:
                 cpu: 10
                 memory: 100Gi
                 furiosa.ai/rngd: "1"
             env:
               - name: HF_TOKEN
                 valueFrom:
                   secretKeyRef:
                     name: hf-token-secret
                     key: token
             volumeMounts:
               - name: model-storage
                 mountPath: /root/.cache/huggingface
             securityContext:
               capabilities:
                 drop:
                   - ALL
               seccompProfile:
                 type: Unconfined
             # Increase initialDelaySeconds of livenessProbe and readinessProbe if larger models take longer to load
             livenessProbe:
               httpGet:
                 path: /health
                 port: 8000
               initialDelaySeconds: 180
               periodSeconds: 10
             readinessProbe:
               httpGet:
                 path: /health
                 port: 8000
               initialDelaySeconds: 180
               periodSeconds: 5
         volumes:
           # If dynamic PVC provisioning isn’t possible in your cluster, consider using a hostPath volume.
           - name: model-storage
             persistentVolumeClaim:
               claimName: llama-storage

Apply the deployment:

.. code-block:: bash

   kubectl apply -f deployment.yaml


Step 4: Expose the Deployment as a Service
------------------------------------------

Expose the Furiosa-LLM server using a Kubernetes Service:

.. code-block:: yaml

   apiVersion: v1
   kind: Service
   metadata:
     name: llama-3-8b
   spec:
     selector:
       app: llama-3-8b
     ports:
       - port: 8000
         targetPort: 8000
     type: ClusterIP

Apply the service:

.. code-block:: bash

   kubectl apply -f service.yaml


Step 5: Test the Deployment
---------------------------

Confirm the server is running by inspecting the logs:

.. code-block:: bash

   kubectl logs deployment/llama-3-8b

You should see output similar to:

.. code-block::

   INFO:     Started server process [1]
   INFO:     Waiting for application startup.
   INFO:     Application startup complete.
   INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)

Test the inference endpoint:

.. code-block:: bash

   kubectl port-forward svc/llama-3-8b 8000:8000

Then, issue a test request:

.. code-block:: bash

   curl http://localhost:8000/v1/completions \
     -H "Content-Type: application/json" \
     -d '{
           "model": "furiosa-ai/Llama-3.1-8B-Instruct-FP8",
           "prompt": "San Francisco is a",
           "max_tokens": 7,
           "temperature": 0
         }'

You should receive a valid response from the Furiosa-LLM server, similar to:

.. code-block:: json

   {
     "id": "cmpl-69102e5d78c94e29b74660eaadbe39db",
     "object": "text_completion",
     "created": 1748675715,
     "model": "meta-llama/Llama-3.1-8B-Instruct",
     "choices": [
       {
         "index": 0,
         "text": " city that is known for its vibrant",
         "logprobs": null,
         "finish_reason": "length",
         "prompt_logprobs": null
       }
     ],
     "usage": {
       "prompt_tokens": 5,
       "total_tokens": 12,
       "completion_tokens": 7,
       "completion_tokens_details": null
     }
   }

Conclusion
----------

Deploying Furiosa-LLM with Kubernetes leverages the efficiency and scalability of Furiosa RNGD accelerators.
This guide provides a straightforward approach to setting up your Kubernetes cluster to serve inference workloads efficiently.
