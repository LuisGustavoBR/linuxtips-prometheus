## Module 4 - Grafana Integration and Dashboards

### What are we going to see today?

One of the most amazing things about Prometheus is the huge number of possible integrations. Today is the day we will see how to integrate Prometheus with Grafana and Alertmanager.

Of course, we will come back to these tools many times throughout the training, but today the goal is to understand how they work and how we can integrate them with Prometheus.

&nbsp;
### Grafana

Grafana is an absolutely amazing tool. It’s almost impossible to find someone who doesn’t know it — and even harder to find someone who doesn’t like it.

But what exactly is Grafana?

Let me explain!

Grafana is an open-source project maintained by Grafana Labs, a Swedish company that today offers several products such as Grafana Cloud, Loki, Tempo, and more.

Grafana is a powerful web application that allows us to visualize data in real time through beautiful and highly customizable dashboards, using data from multiple sources — Prometheus being one of the most popular ones.

Basically, Grafana allows you to create very sophisticated queries to retrieve data from TSDBs (Time Series Databases), such as Prometheus, and then turn that data into awesome dashboards and alerts.

Grafana is a tool I personally use a lot and highly recommend. Even if you don’t use Prometheus, Grafana can consume data from many different sources such as MySQL, PostgreSQL, MongoDB, and others.

As mentioned before, Grafana does not rely only on Prometheus as a data source. It supports several different data sources, also known as `datasources`, such as:

- [Prometheus](https://grafana.com/grafana/plugins/grafana-prometheus-datasource)
- [InfluxDB](https://grafana.com/grafana/plugins/grafana-influxdb-datasource)
- [MySQL](https://grafana.com/grafana/plugins/grafana-mysql-datasource)
- [Postgres](https://grafana.com/grafana/plugins/grafana-postgres-datasource)
- [Elasticsearch](https://grafana.com/grafana/plugins/grafana-elasticsearch-datasource)
- [Google Cloud Monitoring](https://grafana.com/grafana/plugins/grafana-google-cloud-monitoring-datasource)
- [Azure Monitor](https://grafana.com/grafana/plugins/grafana-azure-monitor-datasource)
- [AWS CloudWatch](https://grafana.com/grafana/plugins/grafana-aws-cloudwatch-datasource)
- [OpenTSDB](https://grafana.com/grafana/plugins/grafana-opentsdb-datasource)

These data sources are called `datasources`, and Grafana is able to connect to them, retrieve the required data, and use it to build dashboards and visualizations.

The best part is that Grafana treats `datasources` as plugins. This means you can even create your own datasource if Grafana does not support a specific data source you need — or if you want to integrate Grafana with a system you built yourself. Pretty awesome, right?

&nbsp;
#### Installing Grafana

Let’s start our journey by installing Grafana. In this first example, we will install Grafana as a service on Linux, following the same approach we used to install Prometheus.

Later on, we will also see Prometheus and Grafana running as containers in our beloved Kubernetes — don’t worry!

So, let’s get started.

The first thing we need to do is check Grafana’s official documentation. There you will find installation guides for different operating systems, such as:

- [Linux installation](https://grafana.com/docs/grafana/latest/installation/debian/)
- [Windows installation](https://grafana.com/docs/grafana/latest/installation/windows/)
- [macOS installation](https://grafana.com/docs/grafana/latest/installation/mac/)

In this example, we will install Grafana on Linux, following the Debian/Ubuntu documentation.

Fortunately for us, Grafana provides an official repository that makes installation on Linux distributions like Ubuntu very easy.

First, let’s make sure we have some required packages installed:

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
```

These packages are:
- `apt-transport-https` – allows apt to use HTTPS repositories
- `software-properties-common` – allows apt to manage additional repositories
- `wget` – a tool used to download files from the internet

Before adding the Grafana repository, we need to add its GPG key:

```bash
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

Now we can adds the Grafana repository to our system:

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Next, update the package cache and install Grafana:

```bash
sudo apt-get update
sudo apt-get install grafana-enterprise
```

If everything went well, Grafana is now installed. Let’s start the Grafana service and enable it to start automatically on boot:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

To check if Grafana is running, execute:

```bash
sudo systemctl status grafana-server
```

If the output looks similar to this, Grafana is up and running:

```bash
grafana-server.service - Grafana instance
     Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-12-17 01:39:31 UTC; 1min 31s ago
       Docs: http://docs.grafana.org
   Main PID: 40039 (grafana)
      Tasks: 21 (limit: 9433)
     Memory: 154.9M (peak: 155.7M)
        CPU: 15.373s
     CGroup: /system.slice/grafana-server.service
```

&nbsp;
Since Grafana is a web application, we can access it using a browser. Open the following URL:

```bash
http://localhost:3000
```

![Grafana - Login](grafana-login-page.png)

Great! We are now looking at the Grafana login screen. This means we need a user to access the application.

By default, Grafana creates a user with:
- Username: `admin`
- Password: `admin`

Let’s log in using these credentials.

On your first login, Grafana will ask you to change the default password. Feel free to choose any password you want.

![Grafana - Home](grafana-home-page.png)

Now we are successfully logged in and seeing Grafana’s home screen. For now, there’s not much to see — but we’ll fix that very soon.

&nbsp;
#### Adding Prometheus as a Data Source

We will explore Grafana in much more detail during Module 4, but just so you don’t get *too* curious, let’s already add Prometheus as a data source in Grafana so we can start creating our dashboards.

The first thing we need to do is access the Data Source configuration page. To do that, click on the left-side menu on `Connections` and then on `Data Sources`:

Now click on the `Add data source` button and select Prometheus:

![Grafana - Add Data Source](grafana-data-source-prometheus-1.png)

&nbsp;

Now let’s fill in the Prometheus configuration.

The first field is the Data Source name — let’s use `Prometheus`.

Next, fill in the Prometheus URL. In our case, it is `http://localhost:9090`.

For now, we won’t configure any extra settings such as authentication, alerting, or advanced options.

Once everything is set, click on the `Save & Test` button to save the configuration and test the connection to Prometheus:

![Grafana - Add Data Source](grafana-data-source-prometheus-2.png)

&nbsp;

![Grafana - Add Data Source](grafana-data-source-prometheus-3.png)

&nbsp;

Done! Grafana is now successfully configured to connect to Prometheus.  
Next, let’s create our very first dashboard.

&nbsp;
#### Creating Our First Dashboard

Alright! The big moment has arrived: let’s create our very first Grafana dashboard.  
It will be a very simple one for now — after all, it’s our first dashboard.

The first step is to click on the left-side menu on `Dashboard`, and then click on the `New Dashboard` button:

![Grafana - New Dashboard](grafana-dashboards-1.png)

&nbsp;

Now choose the data source. Click on `Prometheus`:

![Grafana - New Dashboard](grafana-dashboards-2.png)

&nbsp;
To make things easier, let’s split this screen into three areas:

1. The first area is the panel configuration section, where we can configure the panel title, chart type, and other visual settings.

2. The second area is the `Data Source` configuration section, where we choose which data source will feed the panel.

3. The third area is the query configuration section, where we define which metric we want to visualize.

![Grafana - New Dashboard](grafana-dashboards-3.png)

&nbsp;
#### Grafana Dashboards – CPU & Memory

I created two dashboards in **Grafana** using metrics collected by **Prometheus + node_exporter**.

The goal is to provide clear and practical monitoring of **CPU** and **Memory** usage on Linux servers.

&nbsp;
#### CPU Dashboard

The CPU dashboard shows:
- CPU usage in **percentage (%)**
- Per-core visualization
- Metrics based on `node_cpu_seconds_total`
- Usage calculation excluding idle time

This dashboard is useful for:
- Load analysis
- Peak detection
- Real-time troubleshooting

```promql
100 - (rate(node_cpu_seconds_total{mode="idle"}[30s]) * 100)
```

![CPU Dashboard](grafana-dashboards-4.png)

&nbsp;
#### Memory Dashboard

The memory dashboard shows:
- **Total Memory (GB)**
- **Used Memory (GB)**
- Calculations based on:
  - `MemTotal`
  - `MemAvailable`
- Representation aligned with real Linux memory behavior (cache and buffers are considered)

This dashboard helps to:
- Monitor actual RAM usage
- Avoid misleading “memory full” scenarios
- Support alerting and capacity planning

```promql
node_memory_MemTotal_bytes / 1024 / 1024 / 1024
```

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / 1024 / 1024 / 1024
```

![Memory Dashboard](grafana-dashboards-5.png)

Done! Our first dashboard is ready.  
It’s still quite simple, but it already gives you a good taste of how powerful Grafana is, right?

We’ll come back to Grafana later today to explore it a bit more, but now let’s shift our focus to Alertmanager for a moment.
