## Microservices Design Principles

1.  **Independent & Autonomous:**
    * **Principle:** Each microservice should be able to operate, develop, and deploy independently without requiring changes or redeployment of other **services**.
    * **Example:** Your "User Account" service can be updated and deployed at any time, without needing to touch the "Product Catalog" or "Order Processing" services. This aligns with the concept of **CI/CD**.
    * **Recall:** **Service**, **CI/CD**.

2.  **Domain-Driven (Business Capability Focused):**
    * **Principle:** Services should be designed around specific **business capabilities** or bounded contexts within your domain, rather than technical layers (like UI, business logic, data).
    * **Example:** Instead of a single "data access layer" service, you'd have an "Order Management Service" that encapsulates all logic and data related to orders, and a separate "User Profile Service" for user data.
    * **Recall:** **Business Capability**.

3.  **High Cohesion:**
    * **Principle:** All the code and data within a single service should be strongly related to its specific **business capability**. A service should do one thing and do it well.
    * **Example:** The "Payment Processing" service should only handle payments. It shouldn't suddenly take on user authentication logic. If you change something about payments, you only change this one service.
    * **Recall:** **Service**, **Business Capability**.

4.  **Database Decentralization (Database per Service):**
    * **Principle:** Each microservice should own its data persistence and usually has its own dedicated **database**. This minimizes coupling between services and allows for **polyglot persistence**.
    * **Example:** The "Product Catalog" service manages its product data in its own database, while the "User Accounts" service manages user data in a separate database. They don't share one large central database.
    * **Recall:** **Database**, **Polyglot Persistence**.

5.  **Resilient:**
    * **Principle:** Microservices should be designed to handle failures gracefully. If one service fails, it shouldn't cause a cascade of failures throughout the entire system.
    * **Example:** If the "Recommendation" service goes down, the "Product Catalog" service should still be able to display products, perhaps just without recommendations. This uses patterns like **Circuit Breakers** and **Timeouts**.
    * **Recall:** **Service**, **Circuit Breaker**, **Timeout**.

6.  **Discoverable:**
    * **Principle:** Services need a way to find and communicate with each other dynamically, especially in an environment where **services** can scale up/down or be deployed to different network locations. This is done via **Service Discovery**.
    * **Example:** When the "Order Management" service needs to talk to the "Inventory" service, it asks a "Service Registry" (part of **Service Discovery**) for the current location of the "Inventory" service.
    * **Recall:** **Service**, **Service Discovery**.

7.  **Single Source of Truth:**
    * **Principle:** While data is **decentralized** across multiple **databases**, for any given piece of business information, there should be *one authoritative service* that owns and is the **single source of truth** for that data. Other services might *cache* or *replicate* this data, but they don't own it.
    * **Example:** The "User Account" service is the **single source of truth** for a user's email address. The "Order Management" service might know the user's email for a specific order, but it gets that from the "User Account" service and doesn't own it.
    * **Recall:** **Database**, **Service**, **Decentralization**.

---

### Key Terms (Reference)

* **API (Application Programming Interface):** A set of rules for how software components communicate.
* **Business Capability:** A specific function a business performs (e.g., User Accounts).
* **CI/CD (Continuous Integration/Continuous Deployment):** Practices that automate code integration, testing, and deployment.
* **Circuit Breaker:** A **resilience pattern** that prevents a failing **service** from causing cascade failures by stopping calls to it after a certain number of failures.
* **Database:** Where data is stored. In microservices, often one per **service**.
* **Decentralization:** Spreading control or data across multiple entities rather than centralizing it.
* **Monolith:** A single, large, unified application.
* **Polyglot Persistence:** Using different types of **databases** for different **services**.
* **Resilience:** The ability of a system to recover gracefully from failures.
* **Service:** A small, independent, and autonomous part of a microservices application.
* **Service Discovery:** The process by which **services** locate each other in a distributed system. Includes a **Service Registry**.
* **Service Registry:** A **database** of available **services** and their network locations, used by **Service Discovery**.
* **Single Source of Truth:** The principle that for any specific data element, there is one primary, authoritative location or **service** that owns that data.
* **Timeout:** A **resilience pattern** that sets a maximum time an operation will wait for a response before failing.