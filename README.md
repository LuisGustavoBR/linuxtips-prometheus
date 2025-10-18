# LinuxTips Prometheus Learning Path

Welcome to the repository for the **LinuxTips Prometheus Learning Path**!  
This repository contains practical labs, notes, and configurations based on the *Descomplicando Prometheus* training by LinuxTips.  
The goal is to consolidate my hands-on experience with Prometheus, demonstrating a solid understanding of monitoring and observability concepts.

## Overview

Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability.  
It collects metrics, stores them efficiently, and provides powerful query capabilities through PromQL.  
This repository follows a modular structure, covering everything from core concepts to advanced integrations with Grafana and Kubernetes.

## Contents

The repository is organized into modules, each focused on a specific Prometheus topic.  

### Module 1: Introduction to Prometheus
- What is Prometheus?
- Architecture and main components
- Installation and configuration
- Scrape targets and basic metrics exploration

### Module 2: PromQL and Data Model
- Understanding the Prometheus data model
- Working with metric types
- Querying data with PromQL
- Using functions, operators, and aggregations

### Module 3: Exporters and Metrics Collection
- What are exporters?
- Node Exporter setup
- Custom metrics and instrumentation
- Pushgateway and ephemeral jobs

### Module 4: Grafana Integration and Dashboards
- Installing Grafana
- Connecting Prometheus as a data source
- Building and customizing dashboards
- Visualizing metrics effectively

### Module 5: Alerting and Alertmanager
- Alerting rules and thresholds
- Setting up Alertmanager
- Notification channels (Slack, email, etc.)
- Alert routing and grouping

### Module 6: Prometheus in Kubernetes
- Service discovery in Kubernetes
- Monitoring pods, nodes, and services
- kube-prometheus stack overview
- Best practices for observability in clusters

### Module 7: Advanced Topics and Scaling
- High availability and federation
- Remote storage integrations
- Relabeling and metric optimization
- Performance tuning and troubleshooting

## Usage

Each module folder contains its own README with objectives, configuration files, and step-by-step exercises.  
You can clone the repository and follow along:

```bash
git clone https://github.com/LuisGustavoBR/linuxtips-prometheus.git
```

Follow the instructions in each module to run Prometheus locally (via Docker or binaries) and explore the monitoring stack.

## How to Contribute

Contributions are welcome!  
If you find issues or have improvements, feel free to open a pull request.  
Please maintain consistent formatting and clear explanations in your submissions.

## Disclaimer

This repository is for educational purposes only.  
While it follows best practices, it may not reflect production-grade configurations.  
Always validate your setup and consult the official Prometheus documentation for deployment-critical environments.

## Credits

Original training by **LinuxTips**  
Hands-on notes and exercises compiled by **[Luis Gustavo Bordon]**

