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
