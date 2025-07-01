## Microservices Design Patterns: Decomposition Pattern

### 2. Strangler Pattern

* **What it is:** The Strangler Pattern (also known as "Strangler Fig Application" pattern) is a strategy used to incrementally migrate a **monolithic** application into a **microservices** architecture. It's particularly useful when you have a large, aging **monolith** that is too risky or costly to rewrite all at once.

* **How it works:**
    1.  **Identify a Feature/Functionality:** Choose a specific part of the **monolith** (a **business capability**) that you want to extract into a new **microservice**.
    2.  **Build the New Microservice:** Develop the new **microservice** with modern technologies, implementing the identified functionality from scratch.
    3.  **Redirect Traffic:** Implement a **facade** (like an **API Gateway** or a reverse proxy) in front of the **monolith**. This **facade** gradually redirects incoming requests for the newly extracted functionality to the new **microservice**, while other requests still go to the **monolith**.
    4.  **Gradual "Strangulation":** Over time, more and more functionalities are extracted from the **monolith** into new **microservices**, and the **facade** redirects more traffic. The **monolith** "shrinks" as its responsibilities are taken over by the new **services**, until eventually, it can be retired or "strangled."

* **Simple Example:**
    Imagine a large online retail **monolith** handling everything from product display to order processing.

    * **Step 1 (Identify):** You decide to extract the "Order Processing" functionality.
    * **Step 2 (Build):** You build a new, standalone "Order Processing" **microservice**.
    * **Step 3 (Redirect):** You put an **API Gateway** in front. When a customer clicks "Place Order," the **API Gateway** routes that request to the *new* "Order Processing" **microservice**. All other requests (like Browse products or managing accounts) still go to the *old* **monolith**.
    * **Step 4 (Strangulate):** Later, you might extract "User Account Management" into another new **microservice**, and the **API Gateway** starts redirecting those requests too. Eventually, the original **monolith** will have very few responsibilities left and can be decommissioned.

* **Benefit:** Allows for a safer, lower-risk, and incremental migration from a **monolith** to **microservices**, avoiding a large, risky "big bang" rewrite. It maintains continuous operation during the migration.

* **Recall:** **Monolith**, **Microservice**, **Business Capability**, **API Gateway**, **Service**.

---

### Key Terms (Reference)

* **API Gateway:** A single entry point for client requests that routes them to the appropriate **microservice** or **monolith**. Acts as a **facade**.
* **Business Capability:** A specific function a business performs.
* **CI/CD (Continuous Integration/Continuous Deployment):** Practices that automate code integration, testing, and deployment.
* **Domain-Driven Design (DDD):** A software development approach focusing on business domains.
* **Facade:** A software design pattern that provides a simplified interface to a larger body of code, or a system of parts. In the Strangler Pattern, it's the component that redirects traffic.
* **Microservice:** A small, independent **service** within an application.
* **Monolith:** A single, large, unified application.
* **Service:** A component of an application.