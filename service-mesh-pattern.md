## Microservices Design Patterns: Decomposition Pattern

### 4. Service Mesh Pattern 🕸️

* **What it is:** A **Service Mesh** is a dedicated infrastructure layer that handles **service-to-service communication** within a **microservices** architecture. It doesn't run in your application code but sits alongside your **services**, typically implemented using **sidecar** proxies. Its primary goal is to offload **cross-cutting concerns** related to networking, security, and **observability** from individual **services**.

* **How it works:**
    1.  **Data Plane (Proxies/Sidecars):** Each **microservice** instance has its own lightweight proxy (a **sidecar**) deployed alongside it in the same **container** or host. All incoming and outgoing network traffic for that **service** goes *through* its **sidecar**. These proxies manage traffic, implement **resilience patterns** (like **Circuit Breakers** and **Timeouts**), and collect **metrics** and **logs**.
    2.  **Control Plane:** A central control plane manages and configures all the **sidecar** proxies across the entire **service mesh**. It sets policies for traffic routing, security (**mTLS**), and **observability**.
    3.  **Application Code Simplification:** Because the **service mesh** handles so many networking and **cross-cutting concerns**, the application code in your **microservices** remains simpler and focused purely on its **business capability**.

* **Simple Example:**
    Imagine you have "User Account," "Product Catalog," and "Order Processing" **microservices**.

    * **Without Service Mesh:** Each **service** would need code to handle things like retrying failed **API** calls, encrypting communication, collecting detailed **metrics**, or routing traffic based on specific rules (e.g., send 5% of "Order Processing" traffic to a new version).
    * **With Service Mesh:** You deploy a **service mesh** (e.g., Istio, Linkerd). Each **service** automatically gets a **sidecar** proxy. Now, when "Order Processing" calls "Product Catalog," the call actually goes through the "Order Processing" **sidecar**, then through the "Product Catalog" **sidecar**. The **service mesh** handles:
        * **Automatic Retries:** If "Product Catalog" is temporarily busy, the **sidecar** can retry the call automatically.
        * **Load Balancing:** Evenly distributes requests to multiple instances of "Product Catalog."
        * **Security:** Automatically encrypts all communication between **services** (**mTLS**).
        * **Traffic Routing:** Allows you to direct a small percentage of users to a new version of "Product Catalog" for **Canary deployments**.
        * **Observability:** Collects **logs** and **metrics** for all inter-service communication, enabling **distributed tracing**.

* **Benefit:** Provides robust **observability**, improved **resilience**, and enhanced security for inter-service communication *without* embedding that logic into your application code. This reduces developer burden and allows for centralized control over the network.

* **Recall:** **Sidecar**, **Microservice**, **Service**, **Cross-Cutting Concerns**, **Resilience**, **Observability**, **Metrics**, **Logs**, **API**, **Container**, **Circuit Breaker**, **Timeout**, **Business Capability**, **Canary Deployment**.

---

### Key Terms (Reference)

* **API (Application Programming Interface):** Rules for software communication.
* **Business Capability:** A specific function a business performs.
* **Canary Deployment:** A deployment strategy where a new version of a **service** is rolled out to a small subset of users before a full rollout, often managed by a **service mesh**.
* **Circuit Breaker:** A **resilience pattern** preventing cascade failures.
* **Container:** A lightweight, portable package for software (e.g., **Docker**).
* **Cross-Cutting Concerns:** Aspects affecting many **services** (e.g., **logging**, security, **monitoring**).
* **Distributed Tracing:** Tracking requests as they flow through multiple **microservices** to understand latency and identify bottlenecks, often facilitated by a **service mesh**.
* **Logs:** Records of events and activities within **services**.
* **mTLS (Mutual Transport Layer Security):** A security protocol that provides two-way authentication between clients and servers, often handled by a **service mesh**.
* **Metrics:** Quantifiable measures tracking performance and health.
* **Microservice:** A small, independent **service** within an application.
* **Observability:** The ability to understand the internal state of a system from its external outputs (**logs**, **metrics**, traces).
* **Resilience:** Ability of a system to recover from failures.
* **Service:** A component of an application.
* **Service Mesh:** A dedicated infrastructure layer for managing **service-to-service communication**, typically using **sidecars**.
* **Sidecar:** A companion **service** deployed alongside a primary **service** to handle auxiliary tasks.
* **Timeout:** A **resilience pattern** setting a max wait time for an operation.