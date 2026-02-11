## Module 9: Relabeling and Meta Labels

In this module, we will talk about **relabeling**, a powerful technique to make your metrics more organized and easier to query.

With relabeling, you can:

- Add new labels  
- Remove labels  
- Merge labels  
- Filter metrics  
- And much more  

After this module, you will look at metrics from a completely different perspective and better understand how they can be structured and optimized.

Basically, we will play with our `ServiceMonitor` / `PodMonitor`, labels, rules, and relabeling everywhere.

&nbsp;
### What is Relabeling?

Relabeling is one of the most powerful features of Prometheus. It allows you to modify target metadata before metrics are stored.

With relabeling, you can:

- Add labels  
- Remove labels  
- Rename labels  
- Modify label values  
- Filter targets  
- Filter metrics  

For example, you can rename a label, completely remove it, or create a new label with a specific value. You can also filter metrics based on label values.

&nbsp;
### How does Relabeling work?

Relabeling is done through rules applied to each target. These rules are defined inside the `relabelings` block in the configuration.

Example:

```yaml
relabelings:
  - sourceLabels: [__meta_kubernetes_service_label_team]
    regex: '(.*)'
    targetLabel: team
    replacement: '${1}'
```

Below is the full `ServiceMonitor` example:

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
      relabelings:
        - sourceLabels: [__meta_kubernetes_service_label_team]
          regex: '(.*)'
          targetLabel: team
          replacement: '${1}'
```

&nbsp;
### Explanation of each field:

- `sourceLabels`: original label used as input  
- `regex`: regular expression applied to the source label  
- `targetLabel`: label that will be created or modified  
- `replacement`: value assigned to the new label  

There are many other actions available — always check the official documentation.

&nbsp;
### Removing a metric based on a label

If you want to drop all metrics that contain a specific label:

``yaml
relabelings:
  - sourceLabels: [app]
    action: drop
``

All metrics containing the label `app` will be discarded.

* Use this carefully — you might drop important metrics.

&nbsp;
### Merging two labels into one

Example: merge `app` and `team` into a new label called `app_team`.

``yaml
relabelings:
  - sourceLabels: [app, team]
    targetLabel: app_team
    regex: (.*);(.*)
    replacement: ${1}_${2}
``

This creates a new label like:

`app_team="nginx_platform"`

&nbsp;
### Adding a new label

Suppose your cluster runs in `us-east-1` and you want to add a `region` label:

``yaml
relabelings:
  - sourceLabels: []
    targetLabel: region
    replacement: us-east-1
``

&nbsp;
### Storing only specific metrics

If you only want to store metrics where `app` is `nginx` or `redis`:

``yaml
relabelings:
  - sourceLabels: [app]
    regex: '(nginx|redis)'
    action: keep
``

This keeps only metrics matching those values.

&nbsp;
### Mapping all Kubernetes labels

To dynamically map all labels from a Service:

Example Service:

``yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
    team: platform-engineering
    environment: production
    version: 1.0.0
    type: web
spec:
  ports:
  - port: 9113
    name: metrics
  selector:
    app: nginx
``

Relabel rule:

``yaml
relabelings:
  - action: labelmap
    regex: __meta_kubernetes_service_label_(.+)
``

This maps all Service labels into metric labels.

&nbsp;
# Prometheus Meta Labels

Meta labels are automatically added by Prometheus during service discovery.

Example:

If your Service has:

``yaml
labels:
  team: platform-engineering
``

Prometheus automatically creates:

`__meta_kubernetes_service_label_team="platform-engineering"`

&nbsp;
### Common Kubernetes Meta Labels

- `__meta_kubernetes_pod_name`
- `__meta_kubernetes_pod_node_name`
- `__meta_kubernetes_pod_label_<labelname>`
- `__meta_kubernetes_pod_annotation_<annotationname>`
- `__meta_kubernetes_service_name`
- `__meta_kubernetes_service_label_<labelname>`
- `__meta_kubernetes_namespace`
- `__meta_kubernetes_node_name`

Full list:

https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config

&nbsp;
### Cloud Provider Meta Labels

Prometheus also supports cloud provider meta labels.

&nbsp;
### AWS
- `__meta_ec2_instance_id`
- `__meta_ec2_public_ip`

&nbsp;
### GCP
- `__meta_gce_instance_id`
- `__meta_gce_public_ip`

&nbsp;
### Azure
- `__meta_azure_machine_id`
- `__meta_azure_machine_public_ip`

Always check the official documentation for the full list:

https://prometheus.io/docs/prometheus/latest/configuration/configuration/
