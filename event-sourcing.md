## Microservices Design Patterns: Database Patterns

### 5\. Event Sourcing 📖

  * **What it is:** **Event Sourcing** is a design pattern where all changes to an application's state are stored as a sequence of immutable, time-ordered **events**. Instead of just saving the *current state* (like in a traditional **database**), you save *every action* that led to that state. The **event store** becomes the primary **single source of truth** for the application's data.

  * **How it works:**

    1.  **Commands Generate Events:** When a **command** (e.g., `UpdateOrderStatus`, `AddProductToCart`) is executed, it doesn't directly update the **database** state. Instead, it generates one or more **events** that describe the change (e.g., `OrderStatusUpdatedEvent`, `ProductAddedToCartEvent`).
    2.  **Events are Appended to an Event Store:** These **events** are saved (appended) to an **event store** in an immutable, sequential log. The **event store** is the definitive record of all changes.
    3.  **State is Derived from Events:** The current state of an entity (e.g., an `Order` object) is reconstructed by replaying all the **events** related to that entity from the **event store** in chronological order.
    4.  **Events Published for Consumption:** The **event store** typically publishes these **events** to a stream or **message broker**, allowing other **services** or **read models** (as in **CQRS**) to subscribe and react, updating their own internal states or projections.

-----

### Event Sourcing - Markdown Diagram:

```
                  Client Application
                          |
                          V
                 [ Command Service ]
                 (e.g., OrderService)
                          |
                          |  1. Executes Command
                          |  2. Generates Event(s)
                          V
           +-----------------------------+
           | [ Event Store ]             |  <--- Primary Source of Truth
           |   (Immutable Event Log)     |
           +-----------------------------+
                          |
                          |  3. Publishes Events to Stream
                          V
        +-----------------------------------+
        | [ Event Stream / Message Broker ] |
        +-----------------------------------+
             |            |            |
             V            V            V
    [ Projection Service ]  [ Read Model DB ]   [ Other Services / Sagas ]
    (Replays Events to build) (Denormalized Data for Queries) (React to events)
    (e.g., User Profile)
          Current State / Materialized View
```
**Explanation of the Markdown Diagram:**
* **Client Application:** Initiates actions (commands).
* **Command Service:** Receives commands, applies business logic, and generates corresponding **events**.
* **Event Store:** The core component that durably stores all **events** in an append-only log. It is the definitive record.
* **Event Stream / Message Broker:** The mechanism through which **events** from the **Event Store** are published, allowing other components to subscribe.
* **Projection Service / Read Model DB / Other Services / Sagas:** These components consume the **events** from the stream.
    * **Projection Services** and **Read Model DBs** (often part of **CQRS**) rebuild and store optimized views of the data for querying.
    * **Other Services** can react to **events** to perform their own actions (as in **Choreography Sagas**).

---

* **Simple Example: User Account State**
    * **Traditional:** A `users` table has columns like `balance`. When balance changes, you update the `balance` column. You only know the *current* balance.
    * **With Event Sourcing:**
        * **Events:** `AccountCreatedEvent`, `MoneyDepositedEvent(amount)`, `MoneyWithdrawnEvent(amount)`, `AccountBlockedEvent`.
        * **Event Store:** Stores a sequence like:
            1.  `AccountCreatedEvent` (for User A)
            2.  `MoneyDepositedEvent(100)` (for User A)
            3.  `MoneyWithdrawnEvent(30)` (for User A)
        * To get User A's current balance, you replay these **events**: Start at 0, add 100, subtract 30 = 70.
        * The `AccountBlockedEvent` could trigger notifications, update a "Read Model" for blocked users, etc.

* **Relation to CQRS and Sagas:**
    * **Complements CQRS:** **Event Sourcing** provides a natural way to build the "Write Model" for **CQRS**. The **event store** serves as the write-optimized **database**, and the **events** stream directly feeds the "Read Models."
    * **Supports Sagas:** **Events** from the **event store** can directly trigger steps in a **Choreography Saga**, or be consumed by an **Orchestrator** in an **Orchestration Saga**.

* **AWS Services for Event Sourcing:**
    * **Event Store:** **Amazon DynamoDB** (for high-volume, low-latency append-only logs), **Amazon S3** (for archiving or as a cold **event store**).
    * **Event Stream/Message Broker:** **Amazon Kinesis** (Data Streams or Firehose for real-time event streaming), **Amazon SNS** / **Amazon SQS** (for simpler **event** distribution).
    * **Projection/Read Model Storage:** **Amazon DynamoDB**, **Amazon RDS**, **Amazon OpenSearch Service**, **Amazon ElastiCache** (for materialized views).
    * **Compute:** **AWS Lambda** (for processing **commands**, generating **events**, and building **projections**).

---

### Advantages (Pros):

* **Complete Audit Trail:** You have a full, immutable history of every change, which is excellent for auditing, debugging, and compliance.
* **Replayability:** The ability to "replay" **events** allows you to reconstruct past states, debug issues by replaying scenarios, and even build new **read models** or analytics projections from historical data.
* **Decoupling:** **Services** don't query each other's current state directly; they react to **events**, promoting loose coupling.
* **Time-Travel Debugging:** Easier to understand *how* the system reached a certain state.
* **Concurrency:** Handling concurrent updates can be simpler as you're appending **events**, not directly modifying shared state.

---

### Disadvantages (Cons):

* **Increased Complexity:** This is a highly complex pattern. Requires new ways of thinking about data, queries, and **data consistency**.
* **Querying State:** Directly querying the current state is not straightforward; you either have to replay events or build and query **read models**, which adds overhead.
* **Event Schema Evolution:** Changing the schema of past **events** (e.g., adding a new field to an `OrderCreatedEvent`) can be challenging, requiring careful versioning or migration strategies.
* **Data Volume:** The **event store** can grow very large, very quickly.
* **Learning Curve:** Significant learning curve for development teams.

---

### Key Terms (Reference)

* **ACID Transactions:** Properties ensuring reliable **database** transactions.
* **Command:** An operation that changes the state of the system (a "write" operation).
* **CQRS (Command Query Responsibility Segregation):** Pattern separating read and write operations.
* **Data Consistency:** Ensuring that data remains correct and coherent.
* **Database:** An organized collection of data.
* **Denormalized View:** A **database** schema optimized for reading, often by duplicating or combining data.
* **Distributed Transaction:** A transaction involving multiple independent data stores or **services**.
* **Event:** A record of something significant that happened in the system (e.g., `OrderCreated`, `ProductStockUpdated`). Immutable facts.
* **Event Bus:** A channel for **services** to publish and subscribe to **events** (also called **Message Broker**).
* **Event Sourcing:** A pattern where application state changes are stored as an immutable sequence of **events**.
* **Event Store:** The primary persistent storage for the sequence of **events** in an **Event Sourcing** system.
* **Immutable:** Unchangeable; once recorded, an **event** cannot be altered.
* **Local Transaction:** A transaction that operates within a single **service's** database.
* **Message Broker:** An intermediary software that enables different applications to communicate by exchanging messages (**Events**).
* **Microservice:** A small, independent **service** within an application.
* **Projection:** A materialized view or a derived state created by replaying **events** from an **event store**, often used for querying.
* **Read Model:** A data model or **database** optimized for query operations, often built from **events** in **CQRS** and **Event Sourcing**.
* **Saga:** A pattern for managing **distributed transactions** using **local transactions** and **compensating transactions**.
* **Service:** A component of an application.
* **Single Source of Truth:** The principle that for any specific data element, there is one primary, authoritative location or **service** that owns that data. In **Event Sourcing**, this is the **Event Store**.
