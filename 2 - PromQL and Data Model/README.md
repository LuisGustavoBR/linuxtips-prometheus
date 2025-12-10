## Module 2: PromQL and Data Model

Welcome to **Module 2** of your training!  
Yes — I consider this a full training experience, not just a guide to getting the best out of the amazing Prometheus ecosystem.

Today, we are going to learn how to write our **first PromQL queries**, and to do that, we must understand the **Prometheus data model** in detail.  
We will explore what a metric truly is in Prometheus, how it is structured, and how it behaves. You will also learn how to create your **own custom metric** and build your first real queries.

We will build our **first exporter** using Python and Docker, and we’ll dive into the different **metric types** Prometheus supports — how they work, when to use each of them, and why they matter.

Finally, we will explore our first **PromQL functions**, giving you even more power to write effective and meaningful queries as your training continues.

&nbsp;
## Prometheus Data Model

The Prometheus data model is very straightforward, so let’s start by taking a real metric and querying its current value. This will help you clearly understand how the data model works in practice.

&nbsp;
### Querying a Metric

Let’s run a query to check the current value of the `up` metric for the server where Prometheus is running.

Simply enter the metric name:

```
up
```

Remember: we run the query through the Prometheus web UI on port **9090**, which is the default Prometheus port.

Open your browser and visit:

```
http://localhost:9090/
```

![Query retrieving the current value of the up metric](up-query-result.png)

Query result:

```
up{app="prometheus", instance="localhost:9090", job="prometheus"}
```

This single line follows a very specific Prometheus pattern. Once you understand it, everything becomes much easier.

&nbsp;
### Querying via Terminal

If you'd like to get the same result from the terminal instead of the browser, run:

```
curl -GET http://localhost:9090/api/v1/query --data-urlencode "query=up"
```

Just to clarify what the `curl` command above does, let’s break it down step by step:

- `curl` is a tool that allows you to make HTTP requests. In this case, we are asking it to perform a GET request to `http://localhost:9090/api/v1/query` and send a query to Prometheus.

- The request is sent to the Prometheus API endpoint `http://localhost:9090/api/v1/query`, which is the default query URL used by Prometheus.

- The `curl` command includes the query we want to execute — in this example, the `up` metric — by passing the parameter `query=up`.

- We use the `--data-urlencode` flag, which allows `curl` to send data using URL encoding. This works similarly to the `--data` flag, but ensures that the data is properly encoded before being sent.

&nbsp;

The output looks like this:

```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
            "__name__": "up",
            "instance": "localhost:9090",
            "job": "prometheus"
            },
        "value": [
            1661595487.119,
            "1"
            ]
        }
        ]
    }
}
```

Now let’s understand what is actually happening.

&nbsp;
### What did Prometheus return?

Let’s break down the result into parts so everything becomes clearer.

Prometheus always returns data following a consistent structure, and once you understand this pattern, reading any metric becomes much easier.

The result is divided into **two main components**:

---

### **1. The *identifier***  
This includes the **metric name** you queried and all associated **labels**.  
Labels provide context and allow you to filter metrics more precisely. For example, you can query the same metric with different label values, such as `instance="localhost:9090"`, `instance="webserver-01"`, or `instance="webserver-02"`.

---

### **2. The *value***  
This is the current value of the metric — the latest data point collected by Prometheus.

---

To make this easier to visualize, imagine the metric divided into these sections:  
- the metric name  
- its labels  
- the actual metric value  

This structure is the foundation of the Prometheus data model.

![Data Model Prometheus](data-model-prometheus-1.jpg)

If you zoom in on the diagram, you’ll notice that the result is subdivided into even more pieces. Besides the metric name, we also have the labels with their respective values, and finally the current value of the metric.

Take a look at this other diagram:

![Data Model Prometheus](data-model-prometheus-2.jpg)

What we can understand by looking at this query result is that our server is running — meaning Prometheus is working correctly. Notice that the result only shows the value **`1`**; it doesn’t tell us how long it has been running. It only reflects the *current* state. A value of **`1`** means the server is up, and a value of **`0`** would mean the server is down.

If we want to check the value of the `up` metric for the Prometheus server over the *last hour*, we need to express that in our query, which becomes:

```promql
up{instance="localhost:9090", job="prometheus"}[1h]
```

In this case, we are specifying that we want the values of the `up` metric from the `localhost:9090` server, under the `prometheus` job, for the **last hour** — we’re being very specific. :)

To request metrics from the last hour, we use the `[1h]` parameter at the end of the query.  
You can use:

- `h` for hours  
- `d` for days  
- `w` for weeks  
- `m` for months  
- `y` for years

![Data Model Prometheus](query-prometheus-1.jpg)

The result was huge, right?  
And you're wondering why?

Remember that we configured Prometheus to perform a *scrape* every **15 seconds**.  
So, when you request the last **1 hour** of data, Prometheus returns **many points**, one for each scrape.

Here’s just the first result so you can understand it better:

```
up{instance="localhost:9090", job="prometheus"} 1 @1661595094.114
```

Notice that now we have `@1661595094.114`, which is the **timestamp** of the scrape — the exact moment when that metric value was collected.

Now that we understand the data model returned by Prometheus, we can move forward and start exploring and simplifying queries using the powerful Prometheus query language: **PromQL**!

