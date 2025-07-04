## Microservices Design Patterns: Database Patterns

### 1. Shared Database 🛑

* **What it is:** The **Shared Database** pattern is when multiple **microservices** (or components of a **monolith**) directly access and share the **same central database schema**. Each service interacts with the same physical or logical **database**.

* **How it works (and why it's an anti-pattern for pure microservices):**
    * Historically, this is common in **monoliths** where all application components share a single **database**.
    * When migrating from a **monolith** to **microservices**, teams might initially split the application code but leave the **database** as a single, shared entity.
    * **The Problem:** While seemingly convenient, this approach tightly couples services together. Any change to the **database schema** (e.g., adding a column, changing a table name) by one service can inadvertently break other services relying on that same schema. This directly contradicts the **independent** and **autonomous** principles of **microservices**. It also prevents **polyglot persistence**.

* **Simple Example:**
    Imagine you've split your online bookstore **monolith** into two **microservices**: "User Account Service" and "Order Processing Service."

    * **With Shared Database:** Both the "User Account Service" and the "Order Processing Service" would access the *same* `users` table and `orders` table in a single, shared **database**.
    * If the "User Account Service" team decides to rename a column in the `users` table, the "Order Processing Service" will break unless its code is also updated to reflect that change, even though the order service only needs a user ID, not all user details. This creates a hidden dependency and hinders **independent deployment**.

* **Advantages (Pros - mostly for initial migration or small systems):**
    * **Simpler Initial Migration:** Can be a transitional step when breaking down a **monolith**, as it avoids complex data migration upfront.
    * **Simplified Transactions:** Easier to implement **ACID transactions** spanning multiple "logical" services if they're still sharing the same underlying **database**.

* **Disadvantages (Cons - why it's an Anti-Pattern for Microservices):**
    * **Tight Coupling:** Services are tightly coupled via the shared **database schema**. Schema changes require coordination across multiple teams and **services**, hindering **independent deployment**.
    * **Reduced Autonomy:** Services are not truly **autonomous** because they depend on a shared data store.
    * **No Polyglot Persistence:** You're stuck with one **database** technology, preventing different **services** from using the best **database** for their specific needs (e.g., a graph **database** for recommendations vs. a relational **database** for orders).
    * **Scalability Bottleneck:** The single **database** can become a **bottleneck** for the entire system as services scale.
    * **Maintenance Overhead:** Refactoring the shared schema becomes increasingly difficult as more services use it.

* **Recall:** **Microservice**, **Monolith**, **Database**, **Schema**, **Independent**, **Autonomous**, **Polyglot Persistence**, **Independent Deployment**, **Anti-pattern**, **Bottleneck**.

---

### Key Terms (Reference)

* **ACID Transactions:** A set of properties (Atomicity, Consistency, Isolation, Durability) that guarantee **database** transactions are processed reliably. Easier to achieve with a single **database**.
* **Anti-pattern:** A common solution to a problem that is usually ineffective or counterproductive.
* **Autonomous:** Able to operate independently.
* **Bottleneck:** A point in a system where the flow of data or execution is significantly restricted, slowing down the entire system.
* **Database:** An organized collection of data.
* **Database Schema:** The formal description of the structure of a **database**, including tables, columns, data types, and relationships.
* **Independent:** Not reliant on others for operation or deployment.
* **Independent Deployment:** The ability to deploy a **service** without deploying other **services**.
* **Microservice:** A small, independent **service** within an application.
* **Monolith:** A single, large, unified application.
* **Polyglot Persistence:** Using different types of **databases** for different **services**.
* **Service:** A component of an application.