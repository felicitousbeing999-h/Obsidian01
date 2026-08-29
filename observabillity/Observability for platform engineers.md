
![Pasted image 20260810021200](../Pasted%20image%2020260810021200.png)

## Observability or monitoring?

Observability is fundamentally **a property of a system**, allowing you to deduce its internal state by querying it from the outside. It signifies a conceptual evolution designed to overcome the limitations of traditional monitoring, Application Performance Management (APM), and log analytics, especially crucial for cloud-native applications.

Conversely, **monitoring** is **the** **practice of collecting and processing telemetry** - such as metrics, logs, and traces - with the aim of achieving observability. While often confused due to marketing, monitoring typically indicates _what_ is happening, whereas observability delves into _why_ it is happening.

If monitoring tells you _something is wrong_, **observability tells you _why_**.
## The different types of telemetry

Telemetry, often referred to as **signals**, represents the data collected and processed to achieve observability. Traditionally, telemetry was conceptualized around the "three pillars": logs, metrics, and traces. However, modern observability now treats these, and other types, as signals that must seamlessly work together, rather than standalone silos.

**Types of telemetry (signals)**:

- **Logs**: Descriptions of events at a specific point in time, providing **detail and error context**. They can be structured and linked to a trace.
- **Metrics**: Quantitative measurements collected as **time series** (e.g., counters, gauges, histograms), which are effective for **showing trends, anomalies, and alerting**.
- **Traces**: Represent a single user transaction's journey through a distributed system, composed of multiple **spans** (individual units of work). They show how requests flow across services, highlighting latencies and dependencies.
- **Profiles**: Reveal **CPU or memory consumption** at runtime to pinpoint bottlenecks.
- **Real User Monitoring (RUM)**: Collects information on how end-users interact with applications, providing full visibility into feature behaviour in the wild.

Important to consider: **Telemetry without context is just data.**


## Why should platform engineers care about observability?

Platform engineers must care about observability because it is fundamental to their success in managing today's increasingly complex, cloud-native systems. Their role involves **taking care of the increasing levels of complexity underpinning applications**, which feature many moving parts and failure modes.

Observability empowers platform engineers to:

- **Deliver a good developer experience** through their platform.
- **Detect and troubleshoot issues fast**, ensuring that fixes are truly effective.
- **Provide confidence to deploy faster and resolve incidents quicker**.
- **Help developers deliver a good user experience** to their end-users.
## Observability and Platform-as-a-Product

Platform engineers treat their platform as a product, with developers as their customers. In this "platform as a product" mindset, **observability is a core feature** rather than a mere checklist.

By viewing observability as a core product feature of the platform, they can abstract complexity, enforce standards, and provide paved paths for developers, ultimately reducing toil and scaling their impact. Key elements include:

- **Auto-instrumentation**: Often via the OpenTelemetry Operator, it provides telemetry without requiring code changes, reducing developer toil.
- **Enforced Semantic Conventions**: Ensures consistent, queryable, and portable telemetry metadata across teams.
- **Default dashboards and alert rules**: Offers ready-to-use analysis tools, sometimes managed as code (e.g., Perses).
- **Correlation**: Logs, metrics, and traces are pre-wired and correlated, simplifying troubleshooting.

Ultimately, this reduces friction for developers, allowing them to focus on business logic while ensuring reliable and consistent insights.

![Pasted image 20260811042319](../Pasted%20image%2020260811042319.png)

- **Prometheus** for storing **metrics**.
- **OpenSearch** for storing **logs**.
- **Jaeger** for storing **traces**.
- **Perses** for **dashboards**.
- **OpenTelemetry** acts as the **"glue"**, collecting, processing, and exporting telemetry to these different backends, enabling correlation.
OTel is _not_ a proprietary all-in-one observability tool or a query language; instead, it acts as **"the glue"** that standardizes how telemetry (logs, metrics, traces, profiles, and real user monitoring) is collected, structured, and transmitted to various backends. Its core components include **APIs, SDKs**, and the **OpenTelemetry Collector**, a versatile tool for processing, filtering, and routing telemetry.






# # PromQL: the platform engineer’s language

PromQL is often called the **"lingua franca" of the cloud-native observability worl**d
- It is expressive, composable, and battle-tested, with years of production use. Its widespread familiarity means most engineers (and all LLMs) already know how to write it, simplifying onboarding and analysis.
-  Particularly used for metrics-based monitoring and detecting infrastructure anomalies like high CPU usage



## OSS vs. Open Core: choose the right model

When selecting observability tools, platform engineers must understand the distinction between truly Open Source Software (OSS) and Open Core models.

- **Open Core tools** often appear open but reserve many key features behind a commercial paywall. This can lead to vendor lock-in for critical functionalities.
- **Truly open tools**, such as Prometheus and Perses, are typically governed by organisations like the CNCF, with their entire feature set being community-driven, portable, and extensible.

It is crucial to evaluate governance, community activity, and licensing terms, not just the project's superficial "open-source" label. Choosing the right model directly impacts your ability to maintain portability, extendibility, and adapt your observability stack long-term without commercial constraints.










![Pasted image 20260811043135](../Pasted%20image%2020260811043135.png)



## Logs in OpenTelemetry

OpenTelemetry (OTel) considers **logs** a core telemetry signal, representing an **event description at a specific point in time**
 A crucial feature is their ability to link to a span in a trace by including its trace ID and span ID. This enables automatic cross-signal correlation


## Metrics in OpenTelemetry

OpenTelemetry (OTel) treats **metrics** as a core telemetry signal, collecting time series data.

OTel metrics are structured, correlated, and flexible, similar to Prometheus. They are meant to collect time series:

- Counter: accumulates over time, e.g., number of requests
- UpDownCounter: increases or decreases, e.g., concurrent sessions
- Gauge: snapshot of a value, read at the time of export
- Histogram: aggregates value distributions, e.g., request latency
- Asynchronous instruments: for when you don’t control the increment, e.g., reading memory usage directly

![Pasted image 20260811043342](../Pasted%20image%2020260811043342.png)

## Resources

In OpenTelemetry, **resources are metadata** that describe the **origin of telemetry**, such as a process, container, or Kubernetes pod. They are defined by **attributes** like `service.name`, `cloud.region`, or `k8s.pod.name`.

These resources apply to all telemetry signals (logs, metrics, traces). OpenTelemetry uses semantic conventions to standardise these attributes, ensuring consistency across systems. This enables meaningful grouping, filtering, and joining of telemetry