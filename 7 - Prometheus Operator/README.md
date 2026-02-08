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


---

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
