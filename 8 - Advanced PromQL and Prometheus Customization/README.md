## Module 8: Advanced PromQL and Prometheus Customization

In this module, we will explore the metrics we are collecting from our Kubernetes cluster. The goal is to dive deeper into PromQL and extract as much value as possible from the collected metrics.

When using Kube-Prometheus, we already have dozens of metrics available that provide detailed insights into our cluster's behavior.

Another important topic in this module is understanding where and how we can modify the configuration of Prometheus and Alertmanager.

By installing Kube-Prometheus (and consequently the Prometheus Operator), we extend Kubernetes with new capabilities. Some of these include custom resources such as:

- PodMonitor  
- ServiceMonitor  
- PrometheusRule  

In this module, we will also explore two additional resources provided by the Prometheus Operator:

- Prometheus  
- Alertmanager  

&nbsp;
### Exploring Kubernetes Metrics

At this point, we already have our Kubernetes cluster running with Kube-Prometheus installed.

Now it’s time to explore the metrics and extract useful information about cluster health and performance.

&nbsp;
### What Can We Learn About Our Cluster Nodes?

Some questions we can answer:

- How many nodes are in the cluster?
- How much CPU and memory does each node have?
- Is the node ready to receive new pods?
- How much network traffic is each node handling?
- How many pods are running on each node?

&nbsp;
### How Many Nodes Are in the Cluster?

```promql
count(kube_node_info)
```

&nbsp;
### CPU and Memory Per Node

```promql
kube_node_status_allocatable{resource="cpu"}
kube_node_status_allocatable{resource="memory"}
```

Convert memory to GB:

```promql
kube_node_status_allocatable{resource="memory"} / 1024 / 1024 / 1024
```

&nbsp;
### Is the Node Ready?

```promql
kube_node_status_condition{condition="Ready", status="true"}
```

If the value is `1`, the node is ready.  
If `0`, the node is not ready.

&nbsp;
### Network Traffic Per Node

```promql
sum by (instance) (node_network_receive_bytes_total)
sum by (instance) (node_network_transmit_bytes_total)
```

Convert to MB:

```promql
sum by (instance) (node_network_receive_bytes_total) / 1024 / 1024
sum by (instance) (node_network_transmit_bytes_total) / 1024 / 1024
```

&nbsp;
### How Many Pods Are Running Per Node?

```promql
count by (node) (kube_pod_info)
```

&nbsp;
### Detecting Cluster Issues

### Total Running Pods

```promql
count(kube_pod_info)
```

&nbsp;
### Pods in Failed State

```promql
kube_pod_status_phase{phase="Failed"}
```

Count failed pods:

```promql
count(kube_pod_status_phase{phase="Failed"})
```

By namespace:

```promql
count by (namespace) (kube_pod_status_phase{phase="Failed"})
```

&nbsp;
### Pod Resource Limits

```promql
kube_pod_container_resource_limits{resource="memory"}
kube_pod_container_resource_limits{resource="cpu"}
```

&nbsp;
### Disk Pressure

```promql
kube_node_status_condition{condition="DiskPressure", status="true"}
```

&nbsp;
### Memory Pressure

```promql
kube_node_status_condition{condition="MemoryPressure", status="true"}
```

&nbsp;
### Deployment Monitoring

### Number of Deployments

```promql
kube_deployment_status_replicas
```

&nbsp;
### Unavailable Replicas

```promql
kube_deployment_status_replicas_unavailable
```

&nbsp;
### Deployment Health Status

```promql
kube_deployment_status_condition{condition="Available", status="false"}
```

&nbsp;
### Service Monitoring

### List Services

```promql
kube_service_info
```

&nbsp;
### Service Endpoints

```promql
kube_endpoint_address
```

Not ready endpoints:

```promql
kube_endpoint_address{ready="false"}
```

&nbsp;
### Customizing Prometheus

When we installed kube-prometheus, several Custom Resource Definitions (CRDs) were created in our cluster. We have already seen some of them, such as `ServiceMonitor` and `PrometheusRule`. However, our focus now will be on two other CRDs: `Prometheus` and `Alertmanager`.

The `Prometheus` resource allows us to configure the Prometheus instance running in our cluster. The `Alertmanager` resource allows us to configure the Alertmanager instance.

Let’s start by focusing on the `Prometheus` resource.

&nbsp;
### Defining Our Prometheus

As mentioned earlier, the `Prometheus` resource allows us to configure and customize Prometheus according to our needs.

The file below is the default configuration created when installing kube-prometheus. It is located inside the `manifests/` directory of the kube-prometheus repository and is called `prometheus-prometheus.yaml`.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 3.9.1
  name: k8s
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
    - apiVersion: v2
      name: alertmanager-main
      namespace: monitoring
      port: web
  enableFeatures: []
  externalLabels: {}
  image: quay.io/prometheus/prometheus:v3.9.1
  nodeSelector:
    kubernetes.io/os: linux
  podMetadata:
    labels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
      app.kubernetes.io/version: 3.9.1
  podMonitorNamespaceSelector: {}
  podMonitorSelector: {}
  probeNamespaceSelector: {}
  probeSelector: {}
  replicas: 2
  resources:
    requests:
      memory: 400Mi
  ruleNamespaceSelector: {}
  ruleSelector: {}
  scrapeConfigNamespaceSelector: {}
  scrapeConfigSelector: {}
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-k8s
  serviceDiscoveryRole: EndpointSlice
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector: {}
  version: 3.9.1
```

This is the default file created during kube-prometheus installation, located in the `manifests/` directory.

In the example above, comments were added to explain what each section does.

If you want to modify the running Prometheus instance, simply edit this file and apply the changes to the cluster.

&nbsp;
### Adding CPU and Memory Limits

For example, if you want to define CPU and memory limits for Prometheus, add the following under the `spec:` section (maintaining the correct indentation):

```yaml
resources:
  requests:
    memory: 400Mi
    cpu: 500m
  limits:
    memory: 1Gi
    cpu: 900m
```

Now apply the changes:

```bash
kubectl apply -f prometheus.yaml
```

Verify that the configuration has been updated:

```bash
kubectl get pods -n monitoring prometheus-k8s-0 -o yaml | grep -A 10 resources:
```

If everything is correct, you should see the updated CPU and memory requests/limits.

There are many other configurations you can modify in Prometheus, such as adding alert rules, adjusting selectors for monitored resources, and more.

The best way to learn is by testing and exploring the available options.

&nbsp;
### Defining Our Alertmanager

Just like Prometheus, Alertmanager is also defined using a CRD.

The default configuration file created during kube-prometheus installation is located in the `manifests/` directory and is called `alertmanager-alertmanager.yaml`.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  labels:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 0.30.1
  name: main
  namespace: monitoring
spec:
  alertmanagerConfigSelector: {}
  image: quay.io/prometheus/alertmanager:v0.30.1
  nodeSelector:
    kubernetes.io/os: linux
  podMetadata:
    labels:
      app.kubernetes.io/component: alert-router
      app.kubernetes.io/instance: main
      app.kubernetes.io/name: alertmanager
      app.kubernetes.io/part-of: kube-prometheus
      app.kubernetes.io/version: 0.30.1
  replicas: 3
  resources:
    limits:
      cpu: 100m
      memory: 100Mi
    requests:
      cpu: 4m
      memory: 100Mi
  secrets: []
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: alertmanager-main
  version: 0.30.1
```

As with Prometheus, comments were added to explain each section of the configuration.

If you need to modify the running Alertmanager instance, edit the file and apply the changes:

```bash
kubectl apply -f alertmanager-alertmanager.yaml
```

After applying the changes, your Alertmanager will be updated.

Always refer to the official documentation to fully understand all available configuration options.
