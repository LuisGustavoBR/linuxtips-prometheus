## Introduction

In this module, we explore what Prometheus is and the problems it was designed to solve.  
We examine the different types of monitoring and how they compare, setting the foundation for understanding modern observability practices.  

You will learn about the history of Prometheus, including its origins at SoundCloud and the motivations behind its architecture and design. We then dive into Prometheus’ core components, how it fits into the observability ecosystem, and how it integrates with other systems.  

Finally, we install Prometheus, explore its default configuration, access the web interface, and run our first PromQL query — marking the beginning of our hands-on monitoring journey.

## Why Do We Need Tools Like Prometheus?

Before diving into what monitoring actually is, it’s important to understand the broader context. This helps clarify why Prometheus exists and why monitoring systems are critical for modern environments.

Every company has an infrastructure where one or more applications are running and consuming compute resources — whether on-premises, in a datacenter, or more commonly today, in a cloud provider.

Consider a simple example: an online store (an e-commerce platform). Think about how many components must be monitored to ensure everything is working as expected.

For instance, this store likely uses a web server such as Nginx. But Nginx itself must run somewhere — a VM, a container, a Pod, or a cloud instance. If we choose to run Nginx inside a VM, the list of things we need to monitor grows quickly:  
- the VM itself  
- the hypervisor managing the VM  
- the physical host running the hypervisor  
- the Linux OS inside the VM  
- all network components handling customer traffic  

And this is still a very simple scenario. Real environments involve high availability, resilience, compliance requirements, and strict operational constraints.

Even just the host running the VM includes enough metrics to keep you busy for days. There is no shortage of components that require monitoring in a real-world environment.

Monitoring, however, is not only important when something breaks. It helps track patterns, understand how systems behave, and observe ongoing activities. For example, while writing these notes, I am tracking the flight of family members returning to Brazil — all thanks to some monitoring system providing real-time information.

For technology professionals, understanding how to implement and maintain a reliable monitoring solution is essential. Monitoring provides the data needed to ensure reliability, efficiency, and visibility — all of which directly impact business value.

A good monitoring system lets us determine whether a service is fully operational or degraded and potentially delivering a poor experience to users.

### What Does It Mean to Monitor?

Monitoring, in essence, means knowing the current status of the component you care about.  
For example, when dealing with a physical Linux machine, you may want to monitor the hardware to ensure that CPU overheating does not affect system availability. Similarly, you need to monitor the operating system to avoid issues such as a full disk or an overloaded network interface impacting performance.

And that’s without even touching the guest OS inside the VM or the application itself — such as Nginx.

Monitoring means knowing what is happening in your infrastructure, applications, and services **in real time**, and ensuring that the appropriate teams are notified when a failure or anomaly occurs. Alerts should contain enough information to support efficient troubleshooting and recovery.

#### Monitoring vs Observability

Now that you have a sense of what monitoring means, we can go a bit deeper and introduce the broader concept of **observability** — which enhances our ability to understand and analyze system behavior.

Using the same example of Nginx running on a Linux VM:  
Previously we talked about **events** related to hardware failures and **metrics** such as network bandwidth usage. But imagine going further and being able to **measure the latency of each request** processed by your web server.

Imagine also capturing and storing **logs** for every request that returns an error or contains performance-related information.

Or even better, imagine tracing the entire **lifecycle of a user request** — from the moment it enters the first component of your infrastructure to the final millisecond of its execution — while also collecting internal **metrics** that describe how each service behaved along the way.

And finally, imagine consolidating all this data and correlating it visually through comprehensive **dashboards**, filled with graphs and tables that help you understand system behavior during failures or performance degradation.

If you can imagine all that, then you’ve pictured observability at its best.

Observability is commonly described as consisting of three pillars:

- Logs  
- Metrics  
- Traces  

I personally add a fourth pillar: **Events**.  
Being able to correlate events with logs, metrics, and traces provides invaluable insights during incidents or unexpected behavior. So let’s update the list:

- Logs  
- Metrics  
- Traces  
- Events  

And to be completely honest, I would still add **Dashboards** as a fundamental element — because visualizing these pillars together is essential for fully understanding how systems behave, especially during failures.

If your environment provides visibility across these four pillars, congratulations — you have a solid observability foundation!  

In the examples we discussed earlier, you can see that all four pillars are represented. Go back and check — I bet you noticed it too. 😉

At this point, you might be asking yourself:

**“So… what does Prometheus have to do with all of this?”**

That’s exactly what we’ll cover next. 😄
