## Module 2: PromQL and Data Model

Welcome to **Module 2** of your training!  
Yes — I consider this a full training experience, not just a guide to getting the best out of the amazing Prometheus ecosystem.

Today, we are going to learn how to write our **first PromQL queries**, and to do that, we must understand the **Prometheus data model** in detail.  
We will explore what a metric truly is in Prometheus, how it is structured, and how it behaves. You will also learn how to create your **own custom metric** and build your first real queries.

We will build our **first exporter** using Python and Docker, and we’ll dive into the different **metric types** Prometheus supports — how they work, when to use each of them, and why they matter.

Finally, we will explore our first **PromQL functions**, giving you even more power to write effective and meaningful queries as your training continues.

## Prometheus Data Model

The Prometheus data model is very straightforward, so let’s start by taking a real metric and querying its current value. This will help you clearly understand how the data model works in practice.

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
