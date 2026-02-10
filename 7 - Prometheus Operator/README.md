## Module 7: Prometheus Operator

In this module, we go beyond the installation of kube-prometheus and start exploring how Prometheus actually discovers targets, collects metrics, and triggers alerts inside a Kubernetes cluster.

The focus here is understanding and using the main Custom Resources provided by the Prometheus Operator:

- ServiceMonitor
- PodMonitor
- PrometheusRule

By the end of this module, we will:
- Expose application metrics
- Register new scrape targets dynamically
- Create custom alerts handled by Alertmanager

&nbsp;
### Understanding ServiceMonitor

A ServiceMonitor is a Custom Resource used by the Prometheus Operator to define **how Prometheus should scrape metrics from a Kubernetes Service**.

Instead of manually editing Prometheus configuration files, we describe:
- Which Services should be monitored
- Which ports and paths expose metrics
- How often metrics should be scraped

Prometheus automatically discovers these configurations through labels and selectors.

&nbsp;
### Use Case: Monitoring an Nginx Application

To demonstrate ServiceMonitor usage, we deployed:
- An Nginx application
- An Nginx Prometheus Exporter
- A Kubernetes Service exposing metrics
- A ServiceMonitor linked to that Service

This setup reflects a real-world scenario where applications expose metrics through `/metrics`.

&nbsp;
### Application Architecture

The monitoring flow works as follows:

- Nginx serves traffic
- Nginx Exporter exposes metrics on port `9113`
- Kubernetes Service exposes the exporter
- ServiceMonitor selects the Service
- Prometheus scrapes metrics automatically

No Prometheus configuration file changes are required.

&nbsp;
### Creating a ServiceMonitor

Now it is time to create our first **ServiceMonitor**, but before doing that, we need an application to monitor. For this purpose, we will deploy an **Nginx** application and use the **Nginx Exporter** to expose metrics for our service.

Additionally, we will create another pod to generate load against our application, allowing us to perform load testing and reach up to **1000 requests per second**. This will help us validate that metrics are being collected correctly under traffic.

&nbsp;
### Creating the Nginx ConfigMap

Before deploying Nginx, we need to create a **ConfigMap** containing the Nginx configuration we want to use.

This configuration is simple and focuses on:
- Exposing the `/nginx_status` endpoint to provide Nginx internal metrics
- Exposing the `/metrics` endpoint to provide metrics from the Nginx Exporter

To achieve this, we will create a ConfigMap with the following YAML file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```

&nbsp;
###  Understanding the YAML File

Let’s break down what is happening in this YAML file:

- `apiVersion`: Defines the Kubernetes API version being used
- `kind`: Specifies the type of resource being created, in this case a ConfigMap
- `metadata`: Contains metadata information about the resource
- `metadata.name`: The name of the ConfigMap
- `data`: Holds the data stored inside the ConfigMap
- `data.nginx.conf`: The Nginx configuration file content

This ConfigMap will later be mounted into the Nginx container so it can use this custom configuration.

&nbsp;
### Creating the ConfigMap

To create the ConfigMap in the cluster, run the following command:

```bash
kubectl apply -f nginx-config.yaml
```

Once the ConfigMap is created, we can verify that it exists by running:

```bash
kubectl get configmaps
```

&nbsp;
### Creating the Application Deployment

To create our application, we will use the following YAML file. This file defines a **Deployment** that runs Nginx together with the **Nginx Prometheus Exporter**, allowing Prometheus to scrape metrics from the application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-server
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '9113'
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: nginx.conf
        - name: nginx-exporter
          image: nginx/nginx-prometheus-exporter:0.11.0
          args:
            - '-nginx.scrape-uri=http://localhost/metrics'
          resources:
            limits:
              memory: 128Mi
              cpu: 0.3
          ports:
            - containerPort: 9113
              name: metrics
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
            defaultMode: 420
```

&nbsp;
### Understanding the Deployment YAML

Let’s break down what is happening in this YAML file:

- apiVersion: Defines the Kubernetes API version being used
- kind: Specifies the type of resource being created, in this case a Deployment
- metadata: Contains information about the resource
- metadata.name: The name of the Deployment
- spec: Defines the desired state of the Deployment
- spec.selector: Selector used by the Deployment to manage its Pods
- spec.selector.matchLabels: Labels used to identify the Pods managed by the Deployment
- spec.selector.matchLabels.app: Application label used to match Pods
- spec.replicas: Number of replicas that the Deployment will create
- spec.template: Pod template used by the Deployment
- spec.template.metadata: Metadata applied to each Pod
- spec.template.metadata.labels: Labels added to the Pods
- spec.template.metadata.labels.app: Application label assigned to the Pods
- spec.template.metadata.annotations: Annotations added to the Pods
- spec.template.metadata.annotations.prometheus.io/scrape: Enables Prometheus scraping
- spec.template.metadata.annotations.prometheus.io/port: Defines the port Prometheus should scrape
- spec.template.spec: Pod specification
- spec.template.spec.containers: Containers created inside the Pod
- spec.template.spec.containers.name: Name of the container
- spec.template.spec.containers.image: Image used by the container
- spec.template.spec.containers.ports: Ports exposed by the container
- spec.template.spec.containers.ports.containerPort: Container port being exposed
- spec.template.spec.containers.volumeMounts: Volumes mounted into the container
- spec.template.spec.containers.volumeMounts.name: Name of the volume being mounted
- spec.template.spec.containers.volumeMounts.mountPath: Path where the volume is mounted
- spec.template.spec.containers.volumeMounts.subPath: Subpath used from the volume
- spec.template.spec.volumes: Volumes created for the Pod
- spec.template.spec.volumes.configMap: ConfigMap used to create the volume
- spec.template.spec.volumes.configMap.defaultMode: Default permission mode for the volume
- spec.template.spec.volumes.configMap.name: Name of the ConfigMap used
- spec.template.spec.volumes.name: Name of the volume

&nbsp;
### Deploying the Application

Now that we understand what is happening in our YAML file, let’s create the Deployment using the following command:

```bash
kubectl apply -f nginx-deployment.yaml
```

After the Deployment is created, we can verify that the Pods are running:

```bash
kubectl get pods
```

We can also check the Deployment we just created with:

```bash
kubectl get deployments
```

&nbsp;
### Creating the Service

Now we need to create a **Service** to expose our Deployment so that Prometheus can scrape the metrics from the Nginx Exporter.

We will create the Service using the following YAML file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  ports:
    - port: 9113
      name: metrics
  selector:
    app: nginx
```

&nbsp;
### Understanding the Service YAML

Let’s break down what is happening in this YAML file:

- apiVersion: Defines the Kubernetes API version being used
- kind: Specifies the type of resource being created, in this case a Service
- metadata: Contains information about the resource
- metadata.name: The name of the Service
- spec: Defines the desired state of the Service
- spec.selector: Selector used by the Service to find the Pods it should expose
- spec.selector.app: Label used by the Service to match the Pods
- spec.ports: Defines the ports exposed by the Service
- spec.ports.port: Port exposed by the Service
- spec.ports.name: Name of the exposed port

&nbsp;
### Creating the Service

To create the Service in the cluster, run the following command:

```bash
kubectl apply -f nginx-service.yaml
```

Once the Service is created, we can verify it with:

```bash
kubectl get services
```

&nbsp;
### Verifying Nginx and Exposed Metrics

First, let’s verify that our Nginx service is running by executing the following command:

```bash
curl http://<SERVICE-EXTERNAL-IP>:80
```

If everything is working correctly, you should receive a response from Nginx.

&nbsp;
### Verifying Nginx Metrics Endpoint

Now, let’s check if the Nginx metrics are being exposed correctly:

```bash
curl http://<SERVICE-EXTERNAL-IP>:80/nginx_status
```

This endpoint should return basic Nginx statistics such as active connections, requests, and handled connections.

&nbsp;
### Verifying Nginx Exporter Metrics

Next, let’s verify whether the Nginx Exporter is exposing metrics in Prometheus format:

```bash
curl http://<SERVICE-EXTERNAL-IP>:80/metrics
```

If successful, you should see metrics formatted for Prometheus scraping.

At this point, you already know how to create a Kubernetes Service and expose metrics from both Nginx and the Nginx Exporter.

&nbsp;
### Creating a ServiceMonitor

Now, let’s create a **ServiceMonitor** so that Prometheus can automatically discover and scrape metrics from our Nginx Service.

Create the following YAML file:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-servicemonitor
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
    - interval: 10s
      path: /metrics
      targetPort: 9113
```

&nbsp;
### Understanding the ServiceMonitor Configuration

Let’s break down what each part of this YAML file does:

* **`apiVersion`**: Kubernetes API version used for the ServiceMonitor resource.
* **`kind`**: The type of resource being created (`ServiceMonitor`).
* **`metadata`**: Metadata about the resource.
* **`metadata.name`**: Name of the ServiceMonitor.
* **`metadata.labels`**: Labels used to identify and organize the resource.
* **`spec`**: Specification of the ServiceMonitor.
* **`spec.selector`**: Selector used to identify which Service will be monitored.
* **`spec.selector.matchLabels`**: Matches Services labeled with `app: nginx`.
* **`spec.endpoints`**: List of endpoints that Prometheus will scrape.
* **`spec.endpoints.interval`**: How often Prometheus scrapes metrics (every 10 seconds).
* **`spec.endpoints.path`**: HTTP path used to scrape metrics (`/metrics`).
* **`spec.endpoints.targetPort`**: Port where the exporter exposes metrics (`9113`).

&nbsp;
### Applying the ServiceMonitor

Create the ServiceMonitor by running:

```bash
kubectl apply -f nginx-service-monitor.yaml
```

Then, verify that the ServiceMonitor was successfully created:

```bash
kubectl get servicemonitors
```

&nbsp;
### Verifying Metrics in Prometheus

Now that Nginx is running, metrics are exposed, and the ServiceMonitor is configured, the final step is to verify that Prometheus is successfully scraping metrics from both **Nginx** and the **Nginx Exporter**.

You can do this by accessing the Prometheus UI and checking the **Targets** page or querying metrics directly using PromQL.

&nbsp;
### Accessing Prometheus Locally

Let’s create a port-forward to access Prometheus locally:

```bash
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

Now, let’s use `curl` to verify if Prometheus is scraping metrics from Nginx and the Nginx Exporter:

```bash
curl http://localhost:39090/api/v1/targets
```

Done! Now you know how to create a Service in Kubernetes, expose metrics from Nginx and the Nginx Exporter, and create a ServiceMonitor so your Service is monitored by Prometheus.

It’s very important to understand that Prometheus does not scrape metrics automatically. A ServiceMonitor is required for Prometheus to collect metrics.

&nbsp;
### PodMonitors

What if our workload is not a Service? What if our workload is a Pod? How can we monitor a Pod?

There are scenarios where we don’t have a Service in front of our Pods, such as when using CronJobs, Jobs, DaemonSets, and similar resources. I’ve also seen cases where teams use PodMonitors to monitor non-HTTP Pods, for example Pods that expose metrics from RabbitMQ, Redis, Kafka, and others.

&nbsp;
### Creating a PodMonitor

To create a PodMonitor, there are very few changes compared to what we learned when creating a ServiceMonitor. Let’s create our PodMonitor using the following YAML file called ``nginx-pod-monitor.yaml``:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: nginx-podmonitor
  labels:
    app: nginx-pod
spec:
  namespaceSelector:
    matchNames:
      - default
  selector:
    matchLabels:
      app: nginx-pod
  podMetricsEndpoints:
    - interval: 10s
      path: /metrics
      targetPort: 9113
```

As you can see, we are using almost the same options as the ServiceMonitor. The main difference is the use of ``podMetricsEndpoints`` to define which endpoints will be monitored.

Another new concept here is the ``namespaceSelector``, which is used to select the namespaces that will be monitored. In this case, we are monitoring the ``default`` namespace, where our Nginx Pod will be running.

Before deploying the PodMonitor, let’s create our Nginx Pod using the following YAML file called ``nginx-pod.yaml``:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
          name: http
      volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
    - name: nginx-exporter
      image: nginx/nginx-prometheus-exporter:0.11.0
      args:
        - -nginx.scrape-uri=http://localhost/metrics
      resources:
        limits:
          memory: 128Mi
          cpu: 0.3
      ports:
        - containerPort: 9113
          name: metrics
  volumes:
    - configMap:
        defaultMode: 420
        name: nginx-config
      name: nginx-config
```

Once the Pod is created, let’s apply the PodMonitor using the YAML file we created earlier:

```bash
kubectl apply -f nginx-pod-monitor.yaml
```

Now, let’s list the PodMonitors created in the cluster:

```bash
kubectl get podmonitors
```

If you want to see more details about a PodMonitor, run:

```bash
kubectl describe podmonitors nginx-podmonitor
```

You can do the same for a ServiceMonitor:

```bash
kubectl describe servicemonitors nginx-servicemonitor
```

Below is an example of the output from describing our PodMonitor:

```bash
Name:         nginx-podmonitor
Namespace:    default
Labels:       app=nginx-pod
Annotations:  <none>
API Version:  monitoring.coreos.com/v1
Kind:         PodMonitor
Metadata:
  Creation Timestamp:  2026-02-10T14:39:35Z
  Generation:          1
  Resource Version:    75731
  UID:                 057f2548-e39c-46a5-abaa-d4fbb0b836a9
Spec:
  Namespace Selector:
    Match Names:
      default
  Pod Metrics Endpoints:
    Interval:     10s
    Path:         /metrics
    Target Port:  9113
  Selector:
    Match Labels:
      App:  nginx-pod
Events:     <none>
```

As we can see, the PodMonitor was created successfully.

Now let’s check if it appears as a target in Prometheus. To do that, access Prometheus locally using ``kubectl port-forward``:

```bash
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

Now go ahead and check your brand-new target and the metrics being collected.

It’s also worth remembering that you can access the container using ``kubectl exec`` to verify whether the exporter is working correctly or to inspect which metrics are being exposed to Prometheus:

```bash
kubectl exec -it nginx-pod -c nginx-exporter -- bash
```

Now let’s use ``curl`` to verify that the exporter is working properly:

```bash
curl localhost:9113/metrics
```

If everything is working as expected, you should see an output similar to the following:

```bash
# HELP nginx_connections_accepted Accepted client connections
# TYPE nginx_connections_accepted counter
nginx_connections_accepted 1
# HELP nginx_connections_active Active client connections
# TYPE nginx_connections_active gauge
nginx_connections_active 1
# HELP nginx_connections_handled Handled client connections
# TYPE nginx_connections_handled counter
nginx_connections_handled 1
# HELP nginx_connections_reading Connections where NGINX is reading the request header
# TYPE nginx_connections_reading gauge
nginx_connections_reading 0
# HELP nginx_connections_waiting Idle client connections
# TYPE nginx_connections_waiting gauge
nginx_connections_waiting 0
# HELP nginx_connections_writing Connections where NGINX is writing the response back to the client
# TYPE nginx_connections_writing gauge
nginx_connections_writing 1
# HELP nginx_http_requests_total Total http requests
# TYPE nginx_http_requests_total counter
nginx_http_requests_total 39
# HELP nginx_up Status of the last metric scrape
# TYPE nginx_up gauge
nginx_up 1
# HELP nginxexporter_build_info Exporter build information
# TYPE nginxexporter_build_info gauge
```

Remember that all these metrics can be queried directly in Prometheus.

Let’s switch topics a bit — time to talk about alerts!

&nbsp;
### Creating our first alert

Now that we already have Kube-Prometheus installed, let’s configure Prometheus to monitor our EKS cluster. To do that, we’ll use ``kubectl port-forward`` to access Prometheus locally. Just run the following command:

```bash
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

If you want to access Alertmanager, run:

```bash
kubectl port-forward -n monitoring svc/alertmanager-main 39093:9093
```

Done! Now you know how to access Prometheus, Alertmanager, and Grafana locally.

Remember that you can access Prometheus and Alertmanager through your browser using the following URLs:

* Prometheus: ``http://localhost:39090``
* Alertmanager: ``http://localhost:39093``

Simple as that!

Of course, you can expose these services to the internet or to a private VPC, but that’s a topic to discuss with your team.

Before defining a new alert, we need to understand how alerts work now, since we no longer have a standalone alert file like we used when installing Prometheus on a Linux server.

At this point, it’s important to understand that a large part of Prometheus’ configuration lives inside ConfigMaps. ConfigMaps are Kubernetes resources used to store configuration data as key-value pairs and are widely used to manage application configurations.

To list the ConfigMaps in the monitoring namespace, run:

```bash
kubectl get configmaps -n monitoring
```

The output should look similar to this:

```bash
NAME                                                  DATA   AGE
adapter-config                                        1      9d
blackbox-exporter-configuration                       1      9d
grafana-dashboard-alertmanager-overview               1      9d
grafana-dashboard-apiserver                           1      9d
grafana-dashboard-cluster-total                       1      9d
grafana-dashboard-controller-manager                  1      9d
grafana-dashboard-grafana-overview                    1      9d
grafana-dashboard-k8s-resources-cluster               1      9d
grafana-dashboard-k8s-resources-multicluster          1      9d
grafana-dashboard-k8s-resources-namespace             1      9d
grafana-dashboard-k8s-resources-node                  1      9d
grafana-dashboard-k8s-resources-pod                   1      9d
grafana-dashboard-k8s-resources-windows-cluster       1      9d
grafana-dashboard-k8s-resources-windows-namespace     1      9d
grafana-dashboard-k8s-resources-windows-pod           1      9d
grafana-dashboard-k8s-resources-workload              1      9d
grafana-dashboard-k8s-resources-workloads-namespace   1      9d
grafana-dashboard-k8s-windows-cluster-rsrc-use        1      9d
grafana-dashboard-k8s-windows-node-rsrc-use           1      9d
grafana-dashboard-kubelet                             1      9d
grafana-dashboard-namespace-by-pod                    1      9d
grafana-dashboard-namespace-by-workload               1      9d
grafana-dashboard-node-cluster-rsrc-use               1      9d
grafana-dashboard-node-rsrc-use                       1      9d
grafana-dashboard-nodes                               1      9d
grafana-dashboard-nodes-aix                           1      9d
grafana-dashboard-nodes-darwin                        1      9d
grafana-dashboard-persistentvolumesusage              1      9d
grafana-dashboard-pod-total                           1      9d
grafana-dashboard-prometheus                          1      9d
grafana-dashboard-prometheus-remote-write             1      9d
grafana-dashboard-proxy                               1      9d
grafana-dashboard-scheduler                           1      9d
grafana-dashboard-workload-total                      1      9d
grafana-dashboards                                    1      9d
kube-root-ca.crt                                      1      9d
prometheus-k8s-rulefiles-0                            8      9d
```

As you can see, there are several ConfigMaps containing configurations for Prometheus, Alertmanager, and Grafana. We’ll focus on the ``prometheus-k8s-rulefiles-0`` ConfigMap, which stores Prometheus alert rules.

To inspect its contents, run:

```bash
kubectl get configmap prometheus-k8s-rulefiles-0 -n monitoring -o yaml
```

The output is quite large, so here’s a small snippet showing an example alert:

```bash
- alert: KubeMemoryOvercommit
  annotations:
    description: Cluster {{ $labels.cluster }} has overcommitted memory resource
      requests for Pods by {{ $value | humanize }} bytes and cannot tolerate node
      failure.
    runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubememoryovercommit
    summary: Cluster has overcommitted memory resource requests.
  expr: |
    # Non-HA clusters.
    (
      (
        sum by(cluster) (namespace_memory:kube_pod_container_resource_requests:sum{})
        -
        sum by(cluster) (kube_node_status_allocatable{job="kube-state-metrics",resource="memory"}) > 0
      )
      and
      count by (cluster) (max by (cluster, node) (kube_node_role{job="kube-state-metrics", role="control-plane"})) < 3
    )
    or
    # HA clusters.
    (
      sum by(cluster) (namespace_memory:kube_pod_container_resource_requests:sum{})
      -
      (
        # Skip clusters with only one allocatable node.
        (
          sum by (cluster) (kube_node_status_allocatable{job="kube-state-metrics",resource="memory"})
          -
          max by (cluster) (kube_node_status_allocatable{job="kube-state-metrics",resource="memory"})
        ) > 0
      ) > 0
    )
  for: 10m
  labels:
    severity: warning
```

This alert is called ``KubeMemoryOvercommit`` and it triggers when the cluster has more memory requested by Pods than what is available on the nodes. Its definition is the same as the alert rules we used on a standalone Prometheus installation.

&nbsp;
### Creating a new alert

Now that we know where alert rules are stored, let’s create a new alert to monitor Nginx.

Before that, we need to understand what a PrometheusRule resource is.

&nbsp;
### What is a PrometheusRule?

A PrometheusRule is a Kubernetes resource installed when we applied the kube-prometheus CRDs. It allows you to define alert rules for Prometheus, very similar to the alert files used in a traditional Prometheus setup — but now managed as Kubernetes resources.

&nbsp;
### Creating a PrometheusRule

Create a file called ``nginx-prometheus-rule.yaml`` with the following content:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nginx-prometheus-rule
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
spec:
  groups:
  - name: nginx-prometheus-rule
    rules:
    - alert: NginxDown
      expr: up{job="nginx"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Nginx is down"
        description: "Nginx is down for more than 1 minute. Pod name: {{ $labels.pod }}"
```

Apply the PrometheusRule:

```bash
kubectl apply -f nginx-prometheus-rule.yaml
```

Verify that it was created successfully:

```bash
kubectl get prometheusrules -n monitoring
```

Now we have a new alert configured in Prometheus. Since Prometheus is integrated with Alertmanager, when the alert is triggered it will be sent to Alertmanager, which can forward notifications to Slack, email, or any other configured receiver.

At this point, we can confidently say that we understand how to create new targets and alerts in Prometheus running on Kubernetes with the Prometheus Operator.

That’s a wrap for today!
Review everything you learned, practice it, and keep experimenting with new alerts and services in your Kubernetes cluster. We’ll see more cool stuff in the next module!
