## Microservices Design Patterns: Database Patterns

### 2. Database Per Service ✅

* **What it is:** This pattern is the **standard and recommended approach** for data management in a **microservices** architecture. It dictates that each **microservice** **owns its own database** (or its own dedicated schema within a larger **database**) and only that service can directly access it. Other **services** must communicate with it via its **API** to access or modify its data.

* **How it works:**
    * When you design your **microservices** around **business capabilities** (like "Order Management" or "User Accounts"), each resulting **service** is assigned its own dedicated data store.
    * This **database** is **private** to the service. No other service is allowed to directly query or manipulate its tables.
    * If another **service** needs data from a different service's **database**, it must request it through the owning service's exposed **API**.
    * This enables **polyglot persistence**, meaning different services can use different types of databases (e.g., a relational database for orders, a NoSQL document database for user profiles, a graph database for recommendations), choosing the best tool for the job.

* **Simple Example:**
    Consider our online bookstore with "Order Processing Service" and "User Account Service."

    * **With Database Per Service:**
        * The "Order Processing Service" has its own dedicated `orders_db` (e.g., PostgreSQL) containing order details.
        * The "User Account Service" has its own dedicated `users_db` (e.g., MongoDB) containing user profiles.
    * If the "Order Processing Service" needs a user's address for shipping, it *doesn't* directly query `users_db`. Instead, it calls an **API** endpoint (e.g., `GET /users/{userId}/address`) exposed by the "User Account Service," which then retrieves the address from `users_db`.

* **Advantages (Pros):**
    * **True Autonomy & Decoupling:** Each **service** is truly **independent** of others regarding data, promoting **independent deployment** and evolution. Schema changes within one service don't break others.
    * **Technology Flexibility (Polyglot Persistence):** Teams can pick the most suitable **database** technology for each **service's** specific needs, optimizing performance and development.
    * **Scalability:** Each **database** can be scaled independently, avoiding the **bottleneck** of a single shared **database**.
    * **Improved Resilience:** A **database** issue with one **service** won't necessarily bring down the **databases** of other **services**.

* **Disadvantages (Cons):**
    * **Distributed Transactions & Data Consistency:** This is the biggest challenge. Operations that conceptually span multiple **services** (and thus multiple **databases**) are hard. You can't use traditional **ACID transactions**. Patterns like **Sagas** and **Eventual Consistency** become necessary.
    * **Increased Operational Complexity:** Managing many different databases (potentially different types) requires more operational expertise, **monitoring**, and tooling.
    * **Data Duplication/Joins:** Sometimes data needs to be duplicated or synchronized across services, or complex client-side joins (or **Aggregator Pattern**) might be needed to combine data from various services.

* **Recall:** **Microservice**, **Database**, **API**, **Business Capability**, **Polyglot Persistence**, **Independent**, **Autonomous**, **Independent Deployment**, **Shared Database**, **Bottleneck**, **ACID Transactions**, **Sagas**, **Eventual Consistency**, **Monitoring**, **Aggregator Pattern**.

---

### Key Terms (Reference)

* **ACID Transactions:** Properties guaranteeing reliable **database** transactions (Atomicity, Consistency, Isolation, Durability). Not easily achieved across multiple **databases**.
* **Aggregator Pattern:** An **integration pattern** where a **service** retrieves data from multiple other **services** and combines it before returning to the client.
* **API (Application Programming Interface):** Rules for software communication.
* **Autonomous:** Able to operate independently.
* **Bottleneck:** A point in a system where flow is restricted.
* **Business Capability:** A specific function a business performs.
* **Database:** An organized collection of data.
* **Data Consistency:** Ensuring that data remains correct and coherent, especially across distributed systems.
* **Decentralization:** Spreading control or data across multiple entities.
* **Eventual Consistency:** A consistency model where data eventually becomes consistent across all systems, though there might be temporary inconsistencies.
* **Independent:** Not reliant on others for operation or deployment.
* **Independent Deployment:** The ability to deploy a **service** without deploying other **services**.
* **Microservice:** A small, independent **service** within an application.
* **Monitoring:** Tracking the health and performance of systems.
* **Polyglot Persistence:** Using different types of **databases** for different **services**.
* **Saga:** A **database pattern** for managing **distributed transactions** using a sequence of local transactions, with compensating actions for failures.
* **Service:** A component of an application.
* **Shared Database:** An **anti-pattern** where multiple **services** access the same central **database**.