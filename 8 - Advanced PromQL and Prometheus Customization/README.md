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

The `Prometheus` CRD allows us to configure the Prometheus instance running in the cluster.

Example configuration snippet to define resource requests and limits:

```yaml
resources:
  requests:
    memory: 400Mi
    cpu: 500m
  limits:
    memory: 1Gi
    cpu: 900m
```

Apply changes:

```bash
kubectl apply -f prometheus.yaml
```

Verify:

```bash
kubectl get pods -n monitoring prometheus-k8s-0 -o yaml | grep -A 10 resources:
```

&nbsp;
### Customizing Alertmanager

The `Alertmanager` CRD allows configuration of the Alertmanager instance.

Apply updates:

```bash
kubectl apply -f alertmanager-alertmanager.yaml
```

Always refer to the official documentation to explore all available configuration options.
