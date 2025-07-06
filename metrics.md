## Microservices Design Patterns: Observability Patterns

### 2\. Metrics 📊

  * **What it is:** **Metrics** are numerical measurements captured over time, representing various aspects of a **microservice's** behavior and performance. Unlike **logs**, which capture discrete events, **metrics** provide quantitative insights into the health, utilization, and performance of your system. They are typically collected at regular intervals and stored in a **time-series database**.

  * **How it works:**

    1.  **Metric Generation:** Each **microservice** instruments its code to emit numerical data points for key operational aspects (e.g., request count, error rate, latency, CPU utilization, memory usage). These data points are often tagged with **dimensions** (e.g., **service** name, endpoint, HTTP status code) for granular analysis.
    2.  **Metric Collection:** An agent or library within the **service** (or a sidecar **container**) collects these metrics and sends them to a centralized **metrics system**.
    3.  **Metric Aggregation & Storage:** The centralized **metrics system** aggregates these data points (e.g., sums, averages, percentiles) over time and stores them efficiently in a **time-series database**.
    4.  **Monitoring & Alerting:** Engineers use the **metrics system** to create **dashboards** for real-time visualization of system health and set up **alarms** that trigger notifications when metrics cross predefined thresholds, indicating potential issues.
    5.  **Performance Analysis:** Historical **metrics** data is used to analyze trends, identify performance bottlenecks, understand capacity requirements, and track the impact of deployments.

-----

### Metrics - Markdown Diagram:

```
+--------------------+        Emits Metrics with Dimensions        +-----------------------+
| [ Service A ]      | ------------------------------------------> | [ Metrics Collection ]|
| (Order Service)    | (e.g., RequestCount, ErrorRate, Latency)  | (e.g., CloudWatch Agent)|
+--------------------+                                             +-----------------------+
        |                                                                  |
        | Emits Metrics                                                    | Aggregates & Stores
        V                                                                  V
+--------------------+                                             +-----------------------+
| [ Service B ]      | ------------------------------------------> | [ Metrics Database ]  |
| (Payment Service)  |                                             | (e.g., CloudWatch/Prometheus)|
+--------------------+                                             +-----------------------+
                                                                             |
                                                                             | Queries for Dashboards & Alarms
                                                                             V
                                                                   +-----------------------+
                                                                   | [ Dashboards / Alarms ] |
                                                                   | (e.g., Grafana/CloudWatch)|
                                                                   +-----------------------+
```

**Explanation of the Markdown Diagram:**

  * **Service A, B:** Individual **microservices** instrumented to emit numerical data.
  * **Emits Metrics:** The flow of raw metric data points from each **service**.
  * **Metrics Collection:** An agent or component responsible for gathering these metrics.
  * **Metrics Database:** A specialized **time-series database** that stores the aggregated metric data.
  * **Dashboards / Alarms:** Tools used by engineers to visualize metrics and configure alerts.

-----

  * **Simple Example:**
    Consider an `API Gateway` fronting an `Order Service`.

    1.  The `Order Service` is instrumented to increment a counter for every incoming request and record the duration of each request.
    2.  It also increments an error counter whenever a request fails.
    3.  These metrics (`order_request_total`, `order_request_latency_seconds`, `order_error_total`) are continuously sent to a **metrics system**.
    4.  An engineer builds a **dashboard** showing the requests per second, average latency, and error rate for the `Order Service`.
    5.  An **alarm** is set up to trigger a notification if the `order_error_total` rate exceeds 1% for more than 5 minutes.
    6.  When a new deployment happens, the engineer can observe the **dashboard** to ensure the new version's latency remains stable and error rates don't spike, quickly identifying regressions.

  * **AWS Services for Metrics:**

      * **Amazon CloudWatch Metrics:** A core AWS service for collecting and tracking metrics, creating **dashboards**, and setting **alarms**. AWS services automatically publish many useful metrics to **CloudWatch**, and you can publish your custom application metrics.
      * **Amazon Managed Service for Prometheus (AMP):** A fully managed Prometheus-compatible **monitoring** service. Prometheus is a popular open-source system for collecting and analyzing **metrics**, especially in **containerized** environments.
      * **Amazon Managed Grafana:** A fully managed service for **Grafana**, an open-source analytics and visualization web application often used to create **dashboards** from **metrics** stored in **CloudWatch**, Prometheus, and other data sources.
      * **AWS Distro for OpenTelemetry (ADOT):** An AWS-supported distribution of the OpenTelemetry project, which provides SDKs and agents for collecting **metrics** (along with **traces** and **logs**) from your applications.
      * **Amazon OpenSearch Service:** While primarily for **logs**, it can also store and visualize certain types of **metrics**, especially aggregated ones.

-----

### Advantages (Pros):

  * **Quantifiable Insights:** Provides numerical data to objectively assess system performance, health, and capacity.
  * **Real-time Monitoring & Alerting:** Enables proactive detection of issues and automated alerts when thresholds are breached.
  * **Trend Analysis:** Useful for understanding historical performance, identifying long-term trends, and planning capacity.
  * **Dashboards & Visualization:** Easy to create visual representations of system behavior, facilitating quick operational overview.
  * **Efficient Storage:** **Time-series databases** are highly optimized for storing and querying numerical data over time, making them cost-effective for long retention periods compared to raw **logs**.

-----

### Disadvantages (Cons):

  * **Limited Detail:** **Metrics** are aggregates; they tell you *what* is happening (e.g., error rate is high) but often not *why* (e.g., which specific request failed). For root cause analysis, you often need to complement them with **logs** and **traces**.
  * **Pre-defined Nature:** You need to decide which metrics to collect in advance. If you don't instrument for a specific metric, you can't get that data later.
  * **Cardinality Issues:** High-**cardinality** **dimensions** (e.g., unique user IDs, full URL paths) can lead to an explosion in the number of unique metric time series, significantly increasing storage costs and query times.
  * **Instrumentation Overhead:** Adding **metrics** instrumentation to code requires developer effort and can introduce minor performance overhead if not done carefully.

-----

### Key Terms (Reference)

  * **Alarm:** An automatic notification triggered when a **metric** crosses a predefined threshold.
  * **API Gateway:** A single entry point for clients to access **microservices**.
  * **Amazon CloudWatch Metrics:** An AWS service for collecting and tracking metrics.
  * **Amazon Managed Grafana:** A managed service for **Grafana** **dashboards**.
  * **Amazon Managed Service for Prometheus (AMP):** A managed Prometheus-compatible **monitoring** service.
  * **Amazon OpenSearch Service:** A managed service for log analytics and search.
  * **AWS Distro for OpenTelemetry (ADOT):** An AWS-supported distribution for collecting **observability** data.
  * **AWS Lambda:** A serverless compute service.
  * **Canary Deployment:** A deployment strategy for gradual rollouts, heavily reliant on **monitoring** metrics.
  * **Cardinality:** In **metrics**, refers to the number of unique values for a given **dimension**. High **cardinality** can be problematic.
  * **Container:** A lightweight, portable package for software.
  * **Dashboard:** A visual display of key **metrics** and data, typically in real-time.
  * **Database:** An organized collection of data.
  * **Dimension:** A key-value pair that categorizes a **metric**, allowing for more granular analysis.
  * **Error Rate:** The percentage of requests that result in an error.
  * **Instrumentation:** Adding code to an application to collect **observability** data (e.g., **metrics**, **logs**, **traces**).
  * **Latency:** The time delay between a request and a response.
  * **Logs:** Timestamped records of events and state changes.
  * **Microservice:** A small, independent **service** within an application.
  * **Monitoring:** Tracking the health and performance of systems, often using **metrics**.
  * **Observability:** The ability to understand the internal state of a system by examining its external outputs (**logs**, **metrics**, **traces**).
  * **Service:** A component of an application.
  * **Time-series Database:** A **database** optimized for storing and querying data points with associated timestamps.
  * **Trace:** A record of the path a request takes through multiple **services** in a distributed system.