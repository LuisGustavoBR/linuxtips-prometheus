## Module 6: Prometheus in Kubernetes

This repository documents **Module 6 – Prometheus in Kubernetes**, covering the installation and configuration of **kube-prometheus** on a Kubernetes cluster running on **Talos Linux**.

The goal of this module is to understand how Prometheus, Alertmanager, and Grafana work together inside Kubernetes, using Ingress to expose services in a lab environment.

&nbsp;
### Environment Overview

- Kubernetes distribution: Talos Linux
- Observability stack: kube-prometheus
  - Prometheus
  - Alertmanager
  - Grafana
  - node-exporter
  - kube-state-metrics
- Ingress Controller: ingress-nginx
- Access method: Ingress + NodePort
- Local DNS resolution via `/etc/hosts`

&nbsp;
### Installing kube-prometheus

#### Clone the official repository

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
```

&nbsp;
#### Create CRDs and namespace

Before deploying any component, Kubernetes must be aware of the **custom resources** used by the Prometheus Operator.

```bash
kubectl apply -f manifests/setup
```

This step creates:

- CustomResourceDefinitions (Prometheus, ServiceMonitor, Alertmanager, etc)
- The ``monitoring`` namespace

&nbsp;
#### Deploy the monitoring stack

Once CRDs exist, the remaining manifests can be applied.

```bash
kubectl apply -f manifests
```

This step deploys:

- Prometheus instances
- Alertmanager
- Grafana
- Exporters and supporting services

Verify that all pods are running:

```bash
kubectl get pods -n monitoring
```

&nbsp;
#### How Prometheus Discovers Targets

Prometheus does not scrape services automatically.  
Target discovery is controlled by Kubernetes resources such as:

- ``ServiceMonitor``
- ``PodMonitor``

These resources define **what** should be scraped and **how**.

The Prometheus Operator continuously reconciles these objects and updates the Prometheus configuration automatically.

&nbsp;
### Exposing Grafana

#### Why Ingress is Required

Grafana runs inside the cluster and is only accessible through a ClusterIP service by default.

To access it externally without exposing multiple NodePorts, an **Ingress Controller** is used to route HTTP traffic.

&nbsp;
#### Grafana Ingress Configuration

An Ingress resource is created to route requests for ``grafana.local`` to the Grafana service.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
```

This configuration means:

- Incoming HTTP requests for ``grafana.local``
- Are handled by the NGINX Ingress Controller
- And forwarded to the Grafana service on port ``3000``

&nbsp;
#### Local DNS Resolution

Since no external DNS exists in this lab, the hostname is resolved locally.

```txt
192.168.0.109 grafana.local
```

This entry maps the hostname to the IP address of the Kubernetes node running the Ingress Controller.

&nbsp;
#### NodePort Exposure

Because the cluster does not have a cloud LoadBalancer, the Ingress Controller service is exposed using ``NodePort``.

Access pattern:

```
http://grafana.local:<NODEPORT>
```

The NodePort can be identified using:

```bash
kubectl get svc -n ingress-nginx
```

&nbsp;
### Grafana Access and Authentication

Grafana is now accessible through the browser.

Default credentials:

``username: admin``
``password: admin``

If needed, the password can be retrieved from Kubernetes secrets:

```bash
kubectl get secret grafana-admin -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d
```
