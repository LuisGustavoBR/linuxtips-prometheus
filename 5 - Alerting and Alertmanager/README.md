## Module 5: Alerting and Alertmanager

### What Is an Alert?

An alert is a signal that something is not behaving as expected in your system.

In practice, an alert is a rule that continuously evaluates metrics and triggers a notification when a specific condition is met. These conditions are usually based on thresholds, trends, or anomalies observed in metrics collected by systems like Prometheus.

Alerts exist to answer a very simple question:

**“Do I need to take action right now?”**

If the answer is yes, then an alert should fire.

&nbsp;
### Why Alerts Are Important

In modern and distributed systems, it is impossible to manually monitor everything all the time. Metrics alone are not enough — you need automated mechanisms to tell you when something goes wrong.

Alerts help you:

- Detect incidents before users notice them
- Reduce downtime and mean time to recovery (MTTR)
- Identify abnormal behavior in applications and infrastructure
- Take action based on real signals, not guesswork

&nbsp;
### Alert vs Monitoring

This is a very common confusion, so it’s important to clarify.

- **Monitoring** tells you what is happening
- **Alerting** tells you when you need to act

You can have thousands of dashboards and metrics, but if nobody is watching them at the right moment, they don’t help much. Alerts are the bridge between monitoring and action.

&nbsp;
### What Makes a Good Alert?

A good alert should be:

- **Actionable** – someone knows exactly what to do when it fires
- **Clear** – the message explains what is wrong
- **Relevant** – it indicates a real problem, not noise
- **Timely** – it fires neither too early nor too late

Bad alerts generate noise. Too much noise leads to alert fatigue, and alert fatigue leads to alerts being ignored — which is dangerous.

&nbsp;
### Common Examples of Alerts

Some common examples of alerts you might see in real environments:

- CPU usage above 90% for more than 5 minutes
- Memory usage constantly growing without being released
- Disk space below 10%
- An application returning too many 5xx errors
- A service or instance becoming unreachable
- High request latency

&nbsp;
### Alerts in the Prometheus Ecosystem
In the Prometheus ecosystem, alerts are defined using **PromQL expressions**, very similar to the queries we already used for dashboards.

Prometheus is responsible for:

- Evaluating alert rules
- Deciding when an alert should fire or resolve

However, Prometheus does **not** handle notifications by itself.

That’s where **Alertmanager** comes in.

We’ll understand what Alertmanager is, why it exists, and how it works together with Prometheus to deliver alerts to the right people, at the right time.

&nbsp;
### Configuring the First Alert in Prometheus

Now that we understand what an alert is and why it exists, it’s time to create our first alert using Prometheus.

In Prometheus, alerts are defined through **alerting rules**. These rules are written using PromQL expressions and are continuously evaluated by Prometheus.

When the condition defined in a rule is met for a certain period of time, the alert fires.

&nbsp;
### Alert Rules in Prometheus

Alert rules are usually defined in a separate YAML file and then loaded by Prometheus through the `rule_files` directive in the main configuration file (`prometheus.yml`).

A typical alert rule contains:

- A **name** for the alert
- A **PromQL expression** that defines the condition
- A **duration (`for`)**, which specifies how long the condition must be true before the alert fires
- **Labels**, used for grouping and routing alerts
- **Annotations**, used to add human-readable information to the alert

&nbsp;
### Creating the Alert Rules File

First, let’s create a directory to store our alert rules (if it doesn’t exist yet):

```bash
sudo mkdir -p /etc/prometheus/rules
```

Now let’s create our first alert rules file:

```bash
sudo vim /etc/prometheus/rules/alerts.yml
```

Inside the `alerts.yml` file, add the following content:

```yaml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 60
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: High CPU usage detected
          description: CPU usage is above 60% for more than 1 minutes on instance {{ $labels.instance }}
```

&nbsp;
### Understanding the Alert Rule

Let’s break down what this alert does:

- The expression calculates the **CPU usage percentage** using the `node_cpu_seconds_total` metric
- It triggers when CPU usage is **greater than 60%**
- The condition must be true for **1 minutes** before the alert fires
- A `severity` label is added to classify the alert
- Annotations provide a short summary and a detailed description

This helps anyone receiving the alert quickly understand what is happening and where.

&nbsp;
### Loading the Alert Rules in Prometheus

Now we need to tell Prometheus to load this alert rules file.

Edit the `prometheus.yml` file:

```bash
sudo vim /etc/prometheus/prometheus.yml
```

And make sure the `rule_files` section includes our new file:

```yaml
rule_files:
  - "rules/*.yml"
```

After adding the alert rules, restart Prometheus so it can load the new configuration:

```bash
sudo systemctl restart prometheus
```

To check if Prometheus loaded the alert correctly, access the Prometheus web UI:

```http://localhost:9090/alerts```

You should now see the `HighCPUUsage` alert listed there.  
If the condition is not met, it will appear as **inactive**.  
If CPU usage goes above the threshold for more than 2 minutes, it will become **firing**.

At this point, Prometheus is already capable of detecting problems and generating alerts.

In the next step, we’ll integrate Prometheus with **Alertmanager** to route, group, and send these alerts to real notification channels.

&nbsp;
### Alert in Firing State

After increasing the CPU usage on the node, the alert condition was met and the alert transitioned from **inactive** to **firing**.

This confirms that the alert rule is working as expected and that Prometheus is correctly evaluating the expression and triggering the alert when the defined threshold is exceeded.

Below is a screenshot showing the alert in the **firing** state in the Prometheus web interface:

![Prometheus Alert Firing](prometheus-alert-1.png)

At this stage, Prometheus is already capable of detecting issues and generating alerts based on metrics.

In the next steps, we will integrate **Alertmanager** to properly route, group, and deliver these alerts to notification channels such as email, Slack, or other integrations.

&nbsp;
### What Is Alertmanager?

**Alertmanager** is the component in the Prometheus ecosystem responsible for handling alerts after they are fired by Prometheus.

While Prometheus is responsible for **detecting issues** by evaluating alert rules, Alertmanager is responsible for **managing notifications**.

In simple terms:

- Prometheus detects and fires alerts  
- Alertmanager decides what to do with those alerts  

&nbsp;
### Why Alertmanager Exists

Prometheus can identify problems, but it does not know:

- Who should receive the alert  
- How the alert should be delivered  
- When alerts should be grouped or suppressed  

Alertmanager exists to solve these problems and make alerting usable in real-world environments.

&nbsp;
### Main Responsibilities of Alertmanager

Alertmanager provides several important features:

- **Alert grouping**  
  Groups related alerts into a single notification to reduce noise.

- **Deduplication**  
  Prevents the same alert from being sent multiple times.

- **Routing**  
  Routes alerts to different receivers based on labels such as severity, team, or environment.

- **Silencing**  
  Temporarily mutes alerts during maintenance windows.

- **Inhibition**  
  Suppresses less important alerts when a more critical alert is firing.

&nbsp;
### Prometheus vs Alertmanager

It is important to understand the separation of responsibilities:

| Component     | Responsibility |
|--------------|----------------|
| Prometheus   | Evaluate alert rules and change alert states |
| Alertmanager | Route, group, suppress, and send notifications |

Prometheus sends alerts to Alertmanager via HTTP, and from that point on, Alertmanager takes control of alert processing.

&nbsp;
### How Prometheus and Alertmanager Work Together

The alerting flow works as follows:

1. Prometheus evaluates alert rules  
2. An alert condition becomes true  
3. The alert changes to the **firing** state  
4. Prometheus sends the alert to Alertmanager  
5. Alertmanager processes the alert based on its configuration  
6. Notifications are delivered to the appropriate receivers  

This separation makes the alerting system flexible and scalable.

&nbsp;
### Alertmanager Configuration Overview

Alertmanager is configured using a YAML file, usually called `alertmanager.yml`.

This configuration defines:

- Global settings  
- Receivers (email, Slack, etc.)  
- Routing rules  
- Grouping behavior  
- Silences and inhibitions  

In the next steps, Alertmanager will be installed, configured, and integrated with Prometheus to send real notifications.

&nbsp;
### Why Alertmanager Is Essential in Production

Without Alertmanager, alerts quickly become noisy and difficult to manage.

Alertmanager ensures that:

- Alerts reach the right people  
- Notifications are sent at the right time  
- Alert noise is reduced  
- On-call engineers are not overwhelmed  

This makes Alertmanager a critical component in any production-grade monitoring stack.

&nbsp;
### Downloading Alertmanager

In this step, Alertmanager is installed manually using the official binary. This approach helps to better understand how Alertmanager works, where its files are located, and how it is managed as a system service.

```
https://github.com/prometheus/alertmanager/releases/
```

First, download the Alertmanager binary from the official Prometheus repository.

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.30.0/alertmanager-0.30.0.linux-amd64.tar.gz
```

Extract the archive:

```bash
tar -xvzf alertmanager-0.30.0.linux-amd64.tar.gz
cd alertmanager-0.30.0.linux-amd64
```

Move the Alertmanager binaries to `/usr/local/bin`:

```bash
sudo mv alertmanager /usr/local/bin/
sudo mv amtool /usr/local/bin/
```

Verify that the binaries are accessible:

```bash
alertmanager --version
amtool --version
```

&nbsp;
### Creating Alertmanager User and Directories

For security best practices, a dedicated user and directories are created for Alertmanager.

Create group and the user:

```bash
sudo groupadd -f alertmanager
sudo useradd -r -g alertmanager --no-create-home --shell /bin/false alertmanager
```

Create the required directories:

```bash
sudo mkdir -p /etc/alertmanager
sudo mkdir -p /etc/alertmanager/templates
sudo mkdir -p /var/lib/alertmanager
```

Set the correct ownership:

```bash
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown -R alertmanager:alertmanager /var/lib/alertmanager
```

&nbsp;
### Creating the Alertmanager Configuration File

Create the main configuration file `alertmanager.yml`:

```bash
sudo vim /etc/alertmanager/alertmanager.yml
```

Add a minimal configuration:

```yaml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

This configuration is sufficient to start Alertmanager without any notification integrations.

&nbsp;
### Creating the Systemd Service

Create the systemd service file `alertmanager.service`:

```bash
sudo vim /etc/systemd/system/alertmanager.service
```

Service configuration:

```yaml
[Unit]
Description=Alertmanager Service
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target
```

Reload systemd and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

&nbsp;
### Verifying Alertmanager Status

Check the service status:

```bash
sudo systemctl status alertmanager
```

If the service is running correctly, Alertmanager should be active.

Access the Alertmanager web interface:

```bash
http://localhost:9093
```

&nbsp;
### Sending Alerts to Slack

Slack is one of the most commonly used tools for alert notifications.  
Alertmanager has native support for Slack through **Incoming Webhooks**, which makes the integration very simple.

Let's configure it step by step.

&nbsp;
### Configuring Slack Incoming Webhooks

The first step is to create an **Incoming Webhook** in Slack.

1. Access your Slack workspace.
2. Go to **Admin → Apps & workflows**.
&nbsp;
![Slack App Configuration - Step 1](slack-config-app-01.png)

3. Search for **Incoming WebHooks**.
4. Click **Add to Slack**.
&nbsp;
![Slack App Configuration - Step 2](slack-config-app-02.png)

5. Choose the channel where the alerts will be sent (for example `#alerts`).
6. Slack will generate a **Webhook URL**.

This URL will look like this:

```bash
https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
```

**Important**:  
This URL is sensitive information. Never commit it to a public repository.  
In our configuration, we will reference it as `SLACK_API_URL`.

&nbsp;
### Configuring Alertmanager to Send Alerts to Slack

Now let's configure Alertmanager to use the Slack webhook.

Edit the Alertmanager configuration file:

```yaml
sudo vim /etc/alertmanager/alertmanager.yml
```

Below is an example of a complete configuration using Slack as the receiver:

```yaml
global:
  slack_api_url: 'SLACK_API_URL'
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'slack'
receivers:
- name: 'slack'
  slack_configs:
  - channel: '#alerts'
    send_resolved: true
    icon_url: https://eadn-wc05-13372774.nxedge.io/wp-content/uploads/2025/08/Pinguim-equipe.png
    title: >
      [{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ len .Alerts.Firing }}{{ end }}]
      {{ .CommonLabels.alertname }} on instance {{ .CommonLabels.instance }}
    text: >
      {{ range .Alerts }}
      *Alert:* {{ .Annotations.summary }}{{ if .Labels.severity }} - `{{ .Labels.severity }}`{{ end }}

      *Description:* {{ .Annotations.description }}

      *Details:*
      {{ range .Labels.SortedPairs }}

      • *{{ .Name }}:* `{{ .Value }}`
      {{ end }}
      {{ end }}
inhibit_rules:
- source_matchers:
  - severity="critical"
  target_matchers:
  - severity="warning"
  equal: ["alertname", "instance"]
```

&nbsp;
### Configuration Explanation

#### `global.slack_api_url`
Defines the Slack Incoming Webhook URL used by Alertmanager to send notifications to Slack.

&nbsp;
#### `route`
Controls how alerts are grouped and routed.

- `group_by`: Groups alerts by `alertname`
- `group_wait`: How long Alertmanager waits before sending the first alert notification
- `group_interval`: Minimum interval between alert group notifications
- `repeat_interval`: How often an alert is resent while it remains firing

&nbsp;
#### `receivers`
Defines where alerts are sent. In this case, alerts are sent to Slack.

&nbsp;
#### `send_resolved: true`
Sends a notification when the alert is resolved.

&nbsp;
#### `inhibit_rules`
Prevents lower severity alerts (for example `warning`) from being sent when a higher severity alert (`critical`) with the same `alertname` and `instance` is already firing.

&nbsp;
### Validating the Alertmanager Configuration

Before restarting Alertmanager, it is a good practice to validate the configuration file.

Use the following command:

```bash
amtool check-config /etc/alertmanager/alertmanager.yml
```

If everything is correct, you should see a success message.

&nbsp;
### Restarting Alertmanager

After validating the configuration, restart Alertmanager so it loads the new settings:

```bash
sudo systemctl restart alertmanager
```

&nbsp;
### Connecting Prometheus to Alertmanager

Now Prometheus needs to know where Alertmanager is running.

Edit the Prometheus configuration file:

```bash
sudo vim /etc/prometheus/prometheus.yml
```

Add the following section:

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
```

&nbsp;
After saving the file, restart Prometheus:

```bash
sudo systemctl restart prometheus
```

&nbsp;
### Alert Flow Overview

At this point, the alerting flow works as follows:

1. **Prometheus** evaluates alert rules.
2. When a rule is triggered, Prometheus sends the alert to **Alertmanager**.
3. **Alertmanager** processes, groups, and routes the alert.
4. The alert is delivered to **Slack**.

&nbsp;
### Alert Received in Slack

Once the alert enters the **firing** state, you should receive a message in the configured Slack channel, containing:

- Alert name
- Severity
- Description
- Labels and details
- Notification when the alert is resolved (if `send_resolved` is enabled)

&nbsp;
The image below shows two important alert states:

- **FIRING**: indicates that the alert condition is currently active.
- **RESOLVED**: indicates that the alert condition has been cleared and the system has returned to a normal state.

This behavior is possible because the `send_resolved: true` option is enabled in the Alertmanager configuration.

![Alertmanager Slack Notifications](slack-alert-messages.png)
