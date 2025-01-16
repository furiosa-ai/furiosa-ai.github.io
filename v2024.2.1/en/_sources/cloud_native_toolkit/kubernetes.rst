.. _Kubernetes:

####################################
Kubernetes Support
####################################

We do support the following versions of Kubernetes and CRI runtime:

* Kubernetes: v1.24.0 or later
* helm v3.0.0 or later
* CRI Runtime: `containerd <https://github.com/containerd/containerd>`_ or `CRI-O <https://github.com/cri-o/cri-o>`_

.. note::

  Docker is officially deprecated as a container runtime in Kubernetes.
  It is recommended to use containerd or CRI-O as a container runtime.
  Otherwise you may face unexpected issues with the device plugin.
  For more information, see `here <https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/>`_.


.. toctree::
   :maxdepth: 1
   :caption: Kubernetes Support

   /cloud_native_toolkit/kubernetes/feature_discovery
   /cloud_native_toolkit/kubernetes/device_plugin
   /cloud_native_toolkit/kubernetes/metrics_exporter
   /cloud_native_toolkit/kubernetes/scheduling_npus
