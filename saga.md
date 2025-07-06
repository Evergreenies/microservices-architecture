## Microservices Design Patterns: Database Patterns

### 4\. SAGA 🔄

  * **What it is:** The **Saga** pattern is a way to manage **distributed transactions** and maintain **data consistency** across multiple **microservices**, each with its own **database**. It's crucial because traditional **ACID transactions** cannot span multiple independent databases. A **Saga** is a sequence of **local transactions**, where each transaction updates its own database and publishes an **event** that triggers the next local transaction in the sequence. If any step fails, the **Saga** executes **compensating transactions** to undo the changes made by previous successful steps, ensuring atomicity across the distributed system.

  * **Why it's needed:** With the **Database Per Service** pattern, a single logical business operation (like placing an order) might need to update data in several different **services** (e.g., deducting inventory, processing payment, creating an order). You can't use a single global **ACID transaction** across these separate databases. **Saga** provides a mechanism for **eventual consistency** and atomicity in a distributed context.

  * **How it works (Two main approaches):**

    **A. Choreography Saga (Decentralized):**

      * Each **service** involved performs its **local transaction** and then publishes an **event**.
      * Other **services** subscribe to and react to these **events** to perform their own **local transactions**.
      * If a **service** fails its **local transaction**, it publishes a "rollback" or "failure" **event**, triggering **compensating transactions** in previous services that listen for such failure events.

    **Choreography Saga - Markdown Diagram:**

    ```
    +------------------------+      OrderCreated Event      +--------------------------+
    | [ Order Service ]      | ---------------------------> | [ Inventory Service ]    |
    | - Create Order (Tx)    |                              | - Deduct Stock (Tx)      |
    +------------------------+                              +--------------------------+
             |                                                       |
             | StockDeducted Event / StockDeductionFailed Event    |
             V                                                       V
    +------------------------+      PaymentProcessed Event    +--------------------------+
    | [ Payment Service ]    | <--------------------------- | [ Order Service ]        |
    | - Process Payment (Tx) |                              | - Update Order Status    |
    +------------------------+                              |   (Success/Failure)      |
             |                                                +--------------------------+
             | PaymentFailed Event                            |
             V                                                V
    ( If any Tx fails, other services perform Compensating Transactions )
    ```

      * **Flow Description:**
        1.  `Order Service` performs its **local transaction** (creates order) and publishes `OrderCreated Event`.
        2.  `Inventory Service` listens, performs its **local transaction** (deducts stock), and publishes `StockDeducted Event` (or `StockDeductionFailed Event` on failure).
        3.  `Payment Service` listens to `StockDeducted Event`, performs its **local transaction** (processes payment), and publishes `PaymentProcessed Event` (or `PaymentFailed Event` on failure).
        4.  `Order Service` (and potentially others) listens for success/failure events. If a failure event (e.g., `PaymentFailed Event`) is received, it triggers **compensating transactions** (e.g., `Order Service` cancels order, `Inventory Service` might replenish stock if it had deducted it).

    **B. Orchestration Saga (Centralized):**

      * An **Orchestrator service** (or a dedicated Saga orchestrator) manages and coordinates all the **local transactions** in the **Saga**.
      * The **Orchestrator** sends commands to participant **services** telling them what to do.
      * Participant **services** perform their **local transaction** and respond back to the **Orchestrator** with success or failure.
      * The **Orchestrator** then decides the next step, including executing **compensating transactions** if a failure occurs.

    **Orchestration Saga - Markdown Diagram:**

    ```
                                     +--------------------------+
                                     | [ Orchestrator Service ] |
                                     +--------------------------+
                                              |
                                              | 1. Send Command: Create Order
                                              V
    +------------------------+      (Response: OrderCreated)
    | [ Order Service ]      | <----------------------------------+
    | - Create Order (Tx)    | ---------------------------------->| 2. Send Command: Deduct Stock
    +------------------------+      (Response: StockDeducted)   |
                                              |                   |
                                              V                   |
    +------------------------+ <----------------------------------+
    | [ Inventory Service ]  | ---------------------------------->| 3. Send Command: Process Payment
    | - Deduct Stock (Tx)    |      (Response: PaymentProcessed)  |
    +------------------------+                                    |
                                              |                   |
                                              V                   |
    +------------------------+ <----------------------------------+
    | [ Payment Service ]    | ---------------------------------->| (Saga Completed or Rollback Initiated)
    | - Process Payment (Tx) |                                    |
    +------------------------+                                    |
                                                                  | If any step fails:
                                                                  | - Orchestrator sends Compensating Commands
                                                                  +---------------------------------------------> (to previous services)
    ```

      * **Flow Description:**
        1.  The `Orchestrator Service` initiates the Saga by sending a command (e.g., "Create Order") to the `Order Service`.
        2.  The `Order Service` performs its **local transaction** and responds to the `Orchestrator`.
        3.  Based on the `Order Service`'s response, the `Orchestrator` sends the next command (e.g., "Deduct Stock") to the `Inventory Service`.
        4.  This continues until all steps are complete. If any service responds with a failure, the `Orchestrator` sends **compensating commands** to previously completed **services** to roll back their **local transactions**.

  * **Simple Example (Choreography Saga): Online Order Placement**

    1.  `[ Order Service ]` receives an order. It creates the order (**local transaction** on **orders\_db**) and publishes `OrderCreatedEvent` to an **Amazon SNS** topic.
    2.  `[ Inventory Service ]` subscribes to `OrderCreatedEvent` via an **Amazon SQS** queue. It tries to deduct stock (**local transaction** on **inventory\_db**).
          * If successful, it publishes `StockDeductedEvent`.
          * If stock is insufficient, it publishes `StockDeductionFailedEvent`.
    3.  `[ Payment Service ]` subscribes to `StockDeductedEvent`. It processes payment (**local transaction** on **payment\_db**).
          * If successful, it publishes `PaymentProcessedEvent`.
          * If payment fails, it publishes `PaymentFailedEvent`.
    4.  `[ Order Service ]` subscribes to `StockDeductionFailedEvent` or `PaymentFailedEvent`. If either occurs, it performs a **compensating transaction** (e.g., marks order as `cancelled` in **orders\_db** and publishes `OrderCancelledEvent`).
    5.  `[ Inventory Service ]` and `[ Payment Service ]` might subscribe to `OrderCancelledEvent` to perform their own **compensating transactions** (e.g., refund payment, replenish stock).

  * **AWS Services for Sagas:**

      * **Choreography:**
          * **Amazon SNS** (Simple Notification Service) for publishing **events**.
          * **Amazon SQS** (Simple Queue Service) for **services** to consume **events** reliably.
          * **AWS Lambda** as the compute for individual **microservices**.
          * **Amazon DynamoDB** or **Amazon RDS** for **database per service**.
      * **Orchestration:**
          * **AWS Step Functions** acts as the **Orchestrator**, defining the workflow, managing state, and invoking **Lambda** functions (your **services**) for each step. It handles retries and rollback logic.
          * **AWS Lambda** for individual **service** operations.
          * **Amazon DynamoDB** or **Amazon RDS** for **database per service**.

  * **Advantages (Pros):**

      * **Data Consistency in Distributed Systems:** Solves the challenge of achieving atomicity across multiple **databases** without central coordination.
      * **High Decoupling:** **Services** remain **independent** as they only rely on **events** or commands from the orchestrator, not direct calls to each other's data stores.
      * **Resilience:** If one **service** fails, the **Saga** can be designed to roll back or retry, maintaining system integrity.

  * **Disadvantages (Cons):**

      * **Increased Complexity:** **Sagas** are significantly more complex to design, implement, debug, and **monitor** than traditional **ACID transactions**. You need to design **compensating transactions** carefully.
      * **Eventual Consistency:** The system is **eventually consistent**, meaning at any given moment, data might not be fully synchronized across all **services** until the **Saga** completes.
      * **Debugging Challenges:** Tracing a **Saga's** execution across multiple **services** and **events** can be difficult.

-----

### Key Terms (Reference)

  * **ACID Transactions:** Properties (Atomicity, Consistency, Isolation, Durability) ensuring reliable **database** transactions. Difficult to achieve globally in **microservices**.
  * **Atomic (Atomicity):** A property of transactions ensuring that all operations within a transaction are completed successfully, or none are.
  * **Compensating Transaction:** An action that undoes the effects of a previous **local transaction** in a **Saga**, typically in case of a failure later in the Saga.
  * **Data Consistency:** Ensuring that data remains correct and coherent across distributed systems.
  * **Database:** An organized collection of data.
  * **Database Per Service:** A design pattern where each **microservice** owns its own dedicated **database**.
  * **Distributed Transaction:** A transaction that involves multiple independent data stores or **services**.
  * **Event:** A record of something significant that happened in the system (e.g., `OrderCreated`, `ProductStockUpdated`).
  * **Eventual Consistency:** A consistency model where data eventually becomes consistent across all systems, though there might be temporary inconsistencies.
  * **Local Transaction:** A transaction that operates within a single **service's** database.
  * **Message Broker:** An intermediary software that enables different applications to communicate by exchanging messages (**Events**).
  * **Microservice:** A small, independent **service** within an application.
  * **Monitor:** Tracking the health and performance of systems.
  * **Orchestrator:** A central component or **service** that coordinates the execution of a multi-step workflow or **Saga**.
  * **Saga:** A design pattern for managing **distributed transactions** as a sequence of **local transactions**, with **compensating transactions** to handle failures.
  * **Service:** A component of an application.