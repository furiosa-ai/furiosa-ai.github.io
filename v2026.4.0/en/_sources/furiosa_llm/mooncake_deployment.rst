.. _K8sMooncakeDeployment:

****************************************************
Using Mooncake with Furiosa-LLM
****************************************************

This guide describes how to deploy and utilize Mooncake, a cluster-wide distributed KV cache pool, on Kubernetes, enabling KV cache sharing among Furiosa-LLM pods located in the same cluster.

Prerequisites
-------------

- A Kubernetes cluster equipped with Furiosa RNGD devices.
- Prebuilt Mooncake Library

For detailed instructions on setting up an RNGD cluster, please refer to :ref:`Installing Prerequisites<InstallingPrerequisites>` and :ref:`Kubernetes Plugins<Kubernetes>`.

Building Mooncake Library
-------------------------

Furiosa-LLM uses Mooncake libraries built from commit `c989668 <https://github.com/kvcache-ai/Mooncake/tree/c9896684fbd7b85ca207c643056a645ab6be3bad>`__.
Refer to the `Mooncake Build Guide <https://kvcache-ai.github.io/Mooncake/getting_started/build.html>`__ for build instructions.
The library is built with the following configuration:

.. code-block:: bash

   cmake .. \
       -DBUILD_UNIT_TESTS=OFF \
       -DUSE_HTTP=ON \
       -DUSE_ETCD=ON \
       -DUSE_CUDA=OFF \
       -DWITH_EP=OFF \
       -DSTORE_USE_ETCD=ON \
       -DCMAKE_BUILD_TYPE=Release \
       -DBUILD_SHARED_LIBS=ON \
       -DBUILD_EXAMPLES=OFF \
       -DBUILD_BENCHMARK=OFF \
       -DWITH_STORE_GO=OFF \
       -DENABLE_DEBUG_SYMBOLS=OFF

This produces ``libmooncake_store.so`` (and its dependencies such as
``libtransfer_engine.so``). Furiosa-LLM loads this library at runtime rather than
linking it, so it is **not** bundled into the ``furiosa-llm`` wheel; you make it
available to Furiosa-LLM by putting its directory on ``LD_LIBRARY_PATH`` as
described in :ref:`Step 4 <ConnectFuriosaToMooncake>` below.

Step 1: Deploy Mooncake Master
------------------------------

Mooncake Master Service coordinates data transfer and KV cache allocation policies among Mooncake Storages.
The following is a recommended Mooncake Master deployment, which enables Host DRAM and SSDs to be utilized as a distributed memory pool.
Currently, Furiosa-LLM supports TCP-based Mooncake Master Service.

.. code-block:: yaml

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mooncake-master
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mooncake-master
     template:
       metadata:
         labels:
           app: mooncake-master
       spec:
         containers:
         - name: mooncake-master
           image: furiosaai/mooncake:c9896684-no-cuda
           command:
           - mooncake_master
           args:
           - --rpc_address
           - $(POD_IP)
           - --rpc_port
           - "50051"
           - --rpc_thread_num=32
           - --http_metadata_server_host
           - $(POD_IP)
           - --http_metadata_server_port
           - "8080"
           - --enable_http_metadata_server
           - --enable_metric_reporting=true
           - --metrics_port
           - "9003"
           - --enable_offload=true
           - --offload_on_evict=true
           - --promotion_on_hit=true
           - --offloading_queue_limit=500000
           - --offload_cap_ratio=0.8
           - --default_kv_lease_ttl=300000
           - --enable_metadata_cleanup_on_timeout=true
           env:
           - name: POD_IP
             valueFrom:
               fieldRef:
                 apiVersion: v1
                 fieldPath: status.podIP
           - name: MC_TCP_ENABLE_CONNECTION_POOL
             value: "1"
           - name: MC_TE_METRIC
             value: "1"
           - name: MC_TE_METRIC_INTERVAL_SECONDS
             value: "10"
           ports:
           - containerPort: 8080
             name: http
             protocol: TCP
           - containerPort: 9003
             name: metrics
             protocol: TCP
           - containerPort: 50051
             name: rpc
             protocol: TCP
           readinessProbe:
             failureThreshold: 3
             initialDelaySeconds: 10
             periodSeconds: 10
             successThreshold: 1
             tcpSocket:
               port: 8080
             timeoutSeconds: 1

Apply the deployment:

.. code-block:: bash

   kubectl apply -f deployment.yaml

Refer to the `Mooncake Tuning Guide <https://kvcache-ai.github.io/Mooncake/deployment/mooncake-store-deployment-guide.html>`__ for available parameters and tuning guides.

Step 2: Expose Mooncake Master as Service
-----------------------------------------

To expose Mooncake Master Service to the Kubernetes cluster and to enable FQDN resolution, expose the Mooncake Master using a Kubernetes Service:

.. code-block:: yaml

   apiVersion: v1
   kind: Service
   metadata:
     name: mooncake-master
   spec:
     ports:
     - name: http
       port: 8080
       protocol: TCP
       targetPort: 8080
     - name: metrics
       port: 9003
       protocol: TCP
       targetPort: 9003
     - name: rpc
       port: 50051
       protocol: TCP
       targetPort: 50051
     selector:
       app: mooncake-master

Apply the service:

.. code-block:: bash

   kubectl apply -f service.yaml

Step 3: Deploy Mooncake Store (Optional)
----------------------------------------

While Furiosa-LLM launches its own Mooncake Store when L3 cache is enabled, it is recommended to launch a dedicated Mooncake Store for stability and a larger memory pool.
The following configuration deploys a TCP-based Mooncake Store, which contributes 64GB of Host DRAM and 256GB of SSD to the memory pool.
Replace the ``MOONCAKE_MASTER_SERVICE_FQDN`` with the FQDN obtained from the service created in Step 2.

.. code-block:: yaml

   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mooncake-store
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mooncake-store
     template:
       metadata:
         labels:
           app: mooncake-store
       spec:
         containers:
         - name: mooncake-store
           image: furiosaai/mooncake:c9896684-no-cuda
           command:
           - mooncake_client
           args:
           - --master_server_address=(MOONCAKE_MASTER_SERVICE_FQDN):50051
           - --metadata_server=http://(MOONCAKE_MASTER_SERVICE_FQDN):8080/metadata
           - --host=$(POD_IP)
           - --protocol=tcp
           - --global_segment_size=64GB
           - --enable_offload=true
           - --threads=32
           - --port=8088
           - --enable_http_server=true
           env:
           - name: POD_IP
             valueFrom:
               fieldRef:
                 apiVersion: v1
                 fieldPath: status.podIP
           - name: MOONCAKE_OFFLOAD_FILE_STORAGE_PATH
             value: /mnt/ssd_offload
           - name: MOONCAKE_OFFLOAD_LOCAL_BUFFER_SIZE_BYTES
             value: "8589934592"
           - name: MOONCAKE_OFFLOAD_TOTAL_SIZE_LIMIT_BYTES
             value: "274877906944"
           - name: MOONCAKE_OFFLOAD_BUCKET_MAX_TOTAL_SIZE
             value: "247390116249"
           - name: MOONCAKE_OFFLOAD_HEARTBEAT_INTERVAL_SECONDS
             value: "2"
           - name: MOONCAKE_OFFLOAD_USE_URING
             value: "true"
           - name: MC_TCP_ENABLE_CONNECTION_POOL
             value: "1"
           - name: MC_TE_METRIC
             value: "1"
           - name: MC_TE_METRIC_INTERVAL_SECONDS
             value: "10"
           ports:
           - containerPort: 8088
             name: mooncake
             protocol: TCP
           - containerPort: 9300
             name: metrics
             protocol: TCP
           readinessProbe:
             failureThreshold: 3
             initialDelaySeconds: 10
             periodSeconds: 10
             successThreshold: 1
             tcpSocket:
               port: 8088
             timeoutSeconds: 1
           resources:
             limits:
               ephemeral-storage: 256Gi
               memory: 80Gi
             requests:
               ephemeral-storage: 256Gi
               memory: 80Gi
           terminationMessagePath: /dev/termination-log
           terminationMessagePolicy: File
           volumeMounts:
           - mountPath: /mnt/ssd_offload
             name: ssd-offload
         dnsPolicy: ClusterFirst
         imagePullSecrets:
         - name: gcr-credential
         restartPolicy: Always
         schedulerName: default-scheduler
         securityContext: {}
         terminationGracePeriodSeconds: 30
         volumes:
         - emptyDir: {}
           name: ssd-offload

Apply the deployment:

.. code-block:: bash

   kubectl apply -f deployment.yaml


.. _ConnectFuriosaToMooncake:

Step 4: Connect Furiosa-LLM to Mooncake
---------------------------------------

Set the following environment variables on Furiosa-LLM Pods to utilize Mooncake.
The following environment variables enable the TCP-based Mooncake transfer protocol for Furiosa-LLM, with 6.25 GiB of Host DRAM contributing to the distributed memory pool.

Furiosa-LLM does not bundle Mooncake: the ``furiosa-llm`` wheel links no Mooncake
library and stays lean. When L3 is enabled, Furiosa-LLM loads
``libmooncake_store.so`` by soname on first use (``dlopen``), so put the
directory holding the library you built above on ``LD_LIBRARY_PATH`` — the same
directory its own dependencies (``libtransfer_engine.so``, ``libetcd_wrapper.so``,
glog, …) are resolved from. If the library cannot be loaded, Furiosa-LLM logs a
warning and runs L2-only instead of failing.

.. code-block:: bash

   export L3_ENABLED=1
   export LD_LIBRARY_PATH=/opt/mooncake/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}
   export MOONCAKE_MASTER=(MOONCAKE_MASTER_SERVICE_FQDN):50051
   export MOONCAKE_TE_META_DATA_SERVER=http://(MOONCAKE_MASTER_SERVICE_FQDN):8080/metadata
   export MOONCAKE_PROTOCOL=tcp
   export MOONCAKE_LOCAL_HOSTNAME=$(hostname -i)
   export MC_TCP_ENABLE_CONNECTION_POOL=1
   export L3_GLOBAL_SEGMENT_SIZE=6710886400
   export L3_LOCAL_BUFFER_SIZE=2147483648
   export L3_NAMESPACE=(Desired Mooncake Namespace)
