Enable Hubble
==============

Hubble is a fully distributed networking and security observability platform
for cloud native workloads. It is built on top of Cilium and eBPF to enable
deep visibility into the communication and behavior of services as well as the
networking infrastructure in a completely transparent manner.

First, specify the namespace in which Cilium is installed as ``CILIUM_NAMESPACE``
environment variable. Subsequent commands reference this environment variable.

.. parsed-literal::

    export CILIUM_NAMESPACE=kube-system

.. note::

    Cilium is typically installed in ``kube-system`` namespace, but there are
    a few exceptions such as GKE which install Cilium in a dedicated ``cilium``
    namespace.

Deploy Hubble using Helm:

.. parsed-literal::

   helm upgrade cilium |CHART_RELEASE| \\
      --namespace $CILIUM_NAMESPACE \\
      --set global.hubble.enabled=true \\
      --set global.hubble.listenAddress=":4244" \\
      --set global.hubble.metricsServer=":54321" \\
      --set global.hubble.metrics.enabled="{dns,drop,tcp,flow,port-distribution,icmp,http}" \\
      --set global.hubble.relay.enabled=true \\
      --set global.hubble.ui.enabled=true

Restart the Cilium daemonset to allow Cilium agent to pick up the ConfigMap changes:

.. parsed-literal::

    kubectl rollout restart -n $CILIUM_NAMESPACE ds/cilium

To pick one Cilium instance and validate that Hubble server is properly configured to listen on
a UNIX domain socket:

.. parsed-literal::

    kubectl exec -n $CILIUM_NAMESPACE -t ds/cilium -- hubble observe

To validate that Hubble UI is properly configured, set up a port forwarding for ``hubble-ui`` service:

.. parsed-literal::

    kubectl port-forward -n $CILIUM_NAMESPACE svc/hubble-ui 12000

and then open http://localhost:12000/.
