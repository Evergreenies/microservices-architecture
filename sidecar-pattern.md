## Microservices Design Patterns: Decomposition Pattern

### 3. Sidecar / Sidekick Pattern

* **What it is:** The Sidecar pattern involves deploying a companion **service** (the "sidecar") alongside a primary application **service**. The sidecar augments or enhances the primary application's functionality without being part of its core codebase. It's like a sidecar attached to a motorcycle – it rides along with the main vehicle but serves a separate purpose.

* **How it works:**
    1.  **Co-located Deployment:** The sidecar **service** is deployed in the same host or **container** as the primary application **service**. They share the same lifecycle (start, stop, scale together).
    2.  **Dedicated Functionality:** The sidecar handles cross-cutting concerns or auxiliary tasks that are common across many **services**, but aren't core to any single service's **business capability**. This keeps the primary **service** focused and **highly cohesive**.
    3.  **Communication:** The primary **service** communicates with its sidecar, often through a lightweight inter-process communication (IPC) mechanism like `localhost` network calls.

* **Simple Example:**
    Imagine your "Order Processing" **microservice** needs to send all its operational **logs** to a centralized **logging** system, and collect **metrics** about its performance.

    * **Without Sidecar:** The "Order Processing" service itself would need code to connect to the logging system, format logs, handle retries, and send metrics. This adds complexity to its core business logic.
    * **With Sidecar:** You deploy a "Logging & Metrics Sidecar" **service** alongside your "Order Processing" **microservice**. The "Order Processing" service simply writes its logs to a local file or standard output. The Sidecar then picks up these logs, adds relevant metadata, formats them, and reliably sends them to the centralized **logging** system. It also collects metrics from the primary service and forwards them to a **monitoring** system.

    Now, the "Order Processing" **microservice** remains focused on its **business capability**, and the logging/metrics concerns are handled by the dedicated sidecar.

* **Benefit:**
    * **Decoupling:** Separates cross-cutting concerns from the main application logic, promoting **high cohesion** for the primary **service**.
    * **Reusability:** The sidecar can be reused with many different primary **services** without modifying their code.
    * **Technology Agnostic:** The sidecar can be developed in a different language or technology stack than the primary application.

* **Recall:** **Service**, **Container**, **Business Capability**, **High Cohesion**, **Logs**, **Metrics**, **Monitoring**.

---

### Key Terms (Reference)

* **API (Application Programming Interface):** Rules for software communication.
* **Business Capability:** A specific function a business performs.
* **Container:** A lightweight, portable package for software (e.g., **Docker**).
* **Cross-Cutting Concerns:** Aspects of a system that affect many or all **services** (e.g., **logging**, security, **monitoring**).
* **Docker:** A popular tool for **containerization**.
* **High Cohesion:** The principle that all elements within a **service** are strongly related to its specific **business capability**.
* **Logs:** Records of events and activities within **services**, useful for debugging.
* **Metrics:** Quantifiable measures used to track the performance and health of **services**.
* **Microservice:** A small, independent **service** within an application.
* **Monitoring:** Tracking the health and performance of systems using **metrics** and **logs**.
* **Service:** A component of an application.