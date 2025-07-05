## Microservices Design Patterns: Database Patterns

### 3\. CQRS (Command Query Responsibility Segregation) 🔄

  * **What it is:** **CQRS** is a design pattern that **separates** the operations that **change data (Commands)** from the operations that **read data (Queries)**. Instead of using a single data model or **database** for both, **CQRS** allows you to use different models, and even different **databases**, optimized specifically for reads or writes.

  * **How it works:**

    1.  **Commands (Write Side):** These are operations that *change the state* of the system (e.g., `CreateOrder`, `UpdateProductStock`). **Commands** are typically task-based, encapsulate business logic, and aim for strong **data consistency**. They're processed by a "Command Model" (or "Write Model") which updates the **database** optimized for writing.
    2.  **Queries (Read Side):** These are operations that *retrieve data* from the system without changing its state (e.g., `GetOrderDetails`, `ListAllProducts`). **Queries** are processed by a "Query Model" (or "Read Model") which accesses a **database** optimized for reading. This read database is often a **denormalized view** of the data, designed for fast queries.
    3.  **Synchronization:** After a **Command** updates the Write Model's **database**, it often publishes **events** (using an **Event Bus** or **Message Broker**). The Read Model then subscribes to these **events** and updates its own **database** asynchronously. This means the Read Model is typically **eventually consistent** with the Write Model.

  * **Simple Example:**
    Consider an e-commerce platform.

      * **Without CQRS:** A single "Product Service" and its **database** might handle both updating product inventory (writes) and displaying product details for customers (reads). Updating inventory might involve complex validation and locking, slowing down product display for millions of customers.
      * **With CQRS:**
          * **Write Side:** An "Inventory Management Service" handles `UpdateProductStock` **commands**. It uses a highly normalized **database** (e.g., **Amazon RDS** PostgreSQL) to ensure strict **ACID transactions** and accurate stock levels. After updating stock, it publishes a `ProductStockUpdated` **event** to an **Amazon SNS** topic.
          * **Read Side:** A "Product Catalog Service" subscribes to `ProductStockUpdated` **events** via an **Amazon SQS** queue. It maintains a separate, highly **denormalized database** (e.g., **Amazon DynamoDB** or **Amazon OpenSearch Service** for search) specifically for displaying product information quickly to customers. This read model might include cached images, aggregated ratings, and all details needed for display in a single document, optimized for fast reads.

    Now, updating stock doesn't impact customer Browse performance, and the read side can scale independently to handle millions of queries.

-----

### CQRS - Markdown Diagram:

```
                  Client Application
                   /           \
                  /             \
                 V               V
     [ Write/Command Service ]    [ Read/Query Service ]
     (e.g., OrderService)         (e.g., ProductCatalogService)
             |                          ^
             |                          | Reads Data
             V                          |
    [ Write Model / DB ]        [ Read Model / DB ]
    (e.g., Amazon RDS)          (e.g., Amazon DynamoDB, Amazon OpenSearch)
             |                          ^
             | Publishes Event          | Updates from Event
             V                          |
    [ Event Bus / Message Broker ]
    (e.g., Amazon SNS / Amazon SQS)
```

**Explanation of the Markdown Diagram:**

  * **Client Application:** This is where user requests originate.
  * **Write/Command Service:** Handles all operations that modify data (e.g., creating an order, updating stock). It interacts with the **Write Model / DB**.
  * **Read/Query Service:** Handles all operations that retrieve data (e.g., displaying product details, listing orders). It interacts with the **Read Model / DB**.
  * **Write Model / DB:** The **database** specifically optimized for write operations (often normalized, ensuring strong **data consistency** with **ACID transactions**).
  * **Read Model / DB:** A separate **database** (often a **denormalized view**) specifically optimized for read operations, potentially with different technologies for performance.
  * **Event Bus / Message Broker:** This acts as the synchronization mechanism. When the Write Service updates its **database**, it publishes an **event** to the **Event Bus**. The Read Service subscribes to these **events** and updates its Read Model asynchronously, leading to **eventual consistency**.

-----

### AWS Services for CQRS:

  * **Write Side Databases:** **Amazon RDS** (for relational data like PostgreSQL, MySQL), **Amazon DynamoDB** (for high-performance NoSQL writes).
  * **Read Side Databases:** **Amazon DynamoDB** (for fast reads, often with **denormalized views**), **Amazon ElastiCache** (for caching), **Amazon OpenSearch Service** (for search queries), **Amazon Redshift** (for analytical queries).
  * **Eventing/Synchronization:** **Amazon SNS** (for publishing **events**), **Amazon SQS** (for reliable message queuing for **event** consumption), **AWS Lambda** (to process **events** and update read models).
  * **Compute:** **AWS Lambda**, **Amazon ECS** / **Amazon EKS** (for running **microservices**).

-----

### Advantages (Pros):

  * **Independent Scaling:** Read and write workloads often differ. **CQRS** allows you to scale the **read model** (**database**) independently from the **write model** (**database**), optimizing resource usage.
  * **Optimized Models:** You can create data models and choose **database** technologies (using **polyglot persistence**) that are perfectly optimized for either reads or writes, leading to better performance.
  * **Flexibility:** Each side can evolve independently. The **read model** can be easily changed to support new UI requirements or reports without affecting the complex write logic.
  * **Separation of Concerns:** Clear separation makes the code easier to understand, maintain, and secure.

-----

### Disadvantages (Cons):

  * **Increased Complexity:** This is the primary drawback. You're maintaining two data models and often two **databases**, plus the mechanism for synchronizing them (e.g., **Event Bus**), which adds significant design, development, and operational overhead.
  * **Eventual Consistency:** Since the **read model** is asynchronously updated, it might not always reflect the absolute latest state of the **write model**. This is acceptable for many scenarios (like seeing slightly stale product stock for a moment) but problematic for others (like banking transactions where immediate consistency is paramount).
  * **Operational Overhead:** More **databases** and synchronization mechanisms mean more infrastructure to deploy, **monitor**, and manage.

-----

### Key Terms (Reference)

  * **ACID Transactions:** Properties (Atomicity, Consistency, Isolation, Durability) ensuring reliable **database** transactions. Difficult to achieve globally in **microservices**.
  * **Command:** An operation that changes the state of the system (a "write" operation).
  * **Data Consistency:** Ensuring that data remains correct and coherent.
  * **Database:** An organized collection of data.
  * **Database Per Service:** A design pattern where each **microservice** owns its own dedicated **database**.
  * **Denormalized View:** A **database** schema optimized for reading, often by duplicating or combining data to avoid complex joins.
  * **Event:** A record of something significant that happened in the system (e.g., `OrderCreated`, `ProductStockUpdated`).
  * **Event Bus:** A channel for **services** to publish and subscribe to **events** (also called **Message Broker**).
  * **Eventual Consistency:** A consistency model where data eventually becomes consistent across all systems, though there might be temporary inconsistencies.
  * **Message Broker:** An intermediary software that enables different applications to communicate by exchanging messages (e.g., **Event Bus**).
  * **Microservice:** A small, independent **service** within an application.
  * **Monitor:** Tracking the health and performance of systems.
  * **Polyglot Persistence:** Using different types of **databases** for different **services**.
  * **Query:** An operation that retrieves data from the system without changing its state (a "read" operation).
  * **Service:** A component of an application.