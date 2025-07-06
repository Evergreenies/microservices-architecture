## Distributed Transaction Patterns

### 6\. Two-Phase Commit (2PC) 🤝🚫

  * **What it is:** **Two-Phase Commit (2PC)** is a classic **distributed transaction** protocol that ensures all **participants** in a transaction either **commit** (complete) their changes or **rollback** (undo) them. It aims for **strong consistency** and **atomicity** across multiple **databases** or resource managers. Unlike **Saga**, which relies on **eventual consistency** and **compensating transactions**, 2PC tries to provide global **ACID** properties.

  * **How it works:** 2PC involves a central **coordinator** and multiple **participants** (resource managers like databases or **services** managing their own data). It proceeds in two distinct phases:

    **Phase 1: Prepare Phase (Vote Request)**

      * The **coordinator** sends a "prepare" request to all **participants**.
      * Each **participant** performs the transaction's work locally, but doesn't commit it. It logs the transaction's state (e.g., "ready to commit") and locks the resources involved.
      * Each **participant** then votes:
          * **"Yes" (VOTE\_COMMIT):** If it successfully prepared and is ready to commit.
          * **"No" (VOTE\_ABORT):** If it encountered any issue and cannot commit.

    **Phase 2: Commit/Rollback Phase (Decision)**

      * The **coordinator** collects all votes:
          * If *all* **participants** voted "Yes," the **coordinator** sends a "commit" command to all **participants**.
          * If *any* **participant** voted "No" (or if a timeout occurs), the **coordinator** sends a "rollback" command to all **participants**.
      * Each **participant** then either commits its local transaction or rolls it back, releasing any held locks.
      * Finally, each **participant** sends an acknowledgment back to the **coordinator**.

-----

### Two-Phase Commit (2PC) - Markdown Diagram:

```
                            +--------------------------+
                            |     [ Coordinator ]      |
                            +--------------------------+
                                         |
               Phase 1: Prepare (Vote Request)         |
                                         V
         +-------------------------------------------------------------+
         |                                                             |
         V                                                             V
+----------------------+                                    +----------------------+
| [ Participant A ]    |                                    | [ Participant B ]    |
| - Do work, lock data | <------------------------------+   | - Do work, lock data |
| - Log "prepared"     |                                |   | - Log "prepared"     |
+----------------------+                                |   +----------------------+
         |                                              |            |
         | Vote (Yes/No)                                |            | Vote (Yes/No)
         V                                              |            V
                            +--------------------------+
                            |     [ Coordinator ]      |  <-- Collects votes
                            +--------------------------+
                                         |
               Phase 2: Commit / Rollback (Decision)   |
                                         V
         +-------------------------------------------------------------+
         |                                                             |
         V                                                             V
+----------------------+                                    +----------------------+
| [ Participant A ]    |                                    | [ Participant B ]    |
| - Commit/Rollback    | <------------------------------+   | - Commit/Rollback    |
| - Release locks      |                                |   | - Release locks      |
+----------------------+                                |   +----------------------+
         |                                              |            |
         | Acknowledgment (Completion)                  |            | Acknowledgment (Completion)
         V                                              V            V
                                   [ Transaction Done ]
```

**Explanation of the Markdown Diagram:**

  * **Coordinator:** The central authority that orchestrates the **distributed transaction**.
  * **Participants:** The individual resource managers (e.g., **databases** or **services**) that perform **local transactions** as part of the larger **distributed transaction**.
  * **Phase 1 (Prepare):** The **coordinator** instructs **participants** to get ready and vote. **Participants** lock resources and signal readiness.
  * **Phase 2 (Commit/Rollback):** Based on all votes, the **coordinator** decides if the transaction should be fully **committed** or entirely **rolled back**. **Participants** then act accordingly and release locks, finally acknowledging completion.

-----

  * **Simple Example: Bank Transfer**
    Imagine transferring money from Account X (managed by `Bank A Service` with `DB A`) to Account Y (managed by `Bank B Service` with `DB B`).

    1.  A `Transaction Coordinator` **service** (acting as the **coordinator**) initiates the 2PC.
    2.  **Phase 1 (Prepare):**
          * The `Coordinator` asks `Bank A Service` to debit X. `Bank A Service` deducts money from X, marks the change as "prepared," and locks X. It votes "Yes."
          * The `Coordinator` asks `Bank B Service` to credit Y. `Bank B Service` adds money to Y, marks the change as "prepared," and locks Y. It votes "Yes."
    3.  **Phase 2 (Commit):**
          * Since both voted "Yes," the `Coordinator` tells both `Bank A Service` and `Bank B Service` to "commit."
          * Both **services** finalize their changes, making them permanent, and release locks.
          * If `Bank A Service` had a temporary issue and voted "No," the `Coordinator` would tell both to "rollback," and no money would be transferred.

  * **Why it's generally *not* used in Microservices:**
    While 2PC provides strong guarantees, it's rarely used in pure **microservices** architectures due to its significant drawbacks:

      * **Blocking Nature:** **Participants** hold locks on resources during both phases, which can lead to high latency and reduce concurrency.
      * **Single Point of Failure:** The **coordinator** is a single point of failure. If it crashes during Phase 2, **participants** might be left in an "**in-doubt state**," holding locks indefinitely, requiring manual intervention.
      * **Performance Bottleneck:** The centralized nature and synchronous communication make it a **bottleneck** for scalable, highly available systems.
      * **Lack of Autonomy:** It tightly couples **services** (or their **databases**) together, violating the **independent** and **autonomous** principles of **microservices**.

  * **AWS Services:**

      * AWS services typically avoid native 2PC for **distributed transactions** due to its limitations.
      * Most AWS **databases** like **Amazon RDS** or **Amazon DynamoDB** focus on providing **ACID transactions** *within* a single **database** instance or table.
      * For **distributed transactions** across different **services** with **Database Per Service**, AWS encourages patterns like **Saga** (often implemented with **AWS Step Functions** and messaging services like **Amazon SNS** / **Amazon SQS**) or embracing **eventual consistency**.

-----

### Advantages (Pros):

  * **Strong Consistency (ACID):** Guarantees **atomicity** across multiple resource managers, which is vital for certain critical operations (e.g., financial transactions where **data consistency** is paramount).
  * **Simpler Development (Conceptual):** From a developer's perspective, it's conceptually simpler than **Sagas** as it hides the distributed complexity behind a single "transaction."

-----

### Disadvantages (Cons):

  * **Blocking Operations:** **Participants** hold locks on resources for the entire duration of the transaction, severely impacting concurrency and system throughput, especially in high-volume scenarios.
  * **Single Point of Failure:** If the **coordinator** fails, **participants** can be left in an "**in-doubt state**," holding locks indefinitely, leading to system outages.
  * **Performance Bottleneck:** The synchronous, two-phase handshake adds significant latency and overhead, making it unsuitable for highly scalable systems.
  * **Reduced Autonomy:** Tightly couples **services** together, going against the core **microservices** principle of **independence** and **autonomy**.
  * **Lack of Resilience:** Susceptible to network partitions and crashes.

-----

### Key Terms (Reference)

  * **ACID Transactions:** Properties (Atomicity, Consistency, Isolation, Durability) ensuring reliable **database** transactions.
  * **Atomicity:** A property of transactions ensuring that all operations within a transaction are completed successfully, or none are.
  * **Autonomous:** Able to operate independently.
  * **Bottleneck:** A point in a system where the flow is restricted.
  * **Commit:** To make the changes of a transaction permanent in a **database**.
  * **Compensating Transaction:** An action that undoes the effects of a previous **local transaction** in a **Saga**.
  * **Coordinator:** The central component in 2PC that manages the **distributed transaction**.
  * **Data Consistency:** Ensuring that data remains correct and coherent.
  * **Database:** An organized collection of data.
  * **Database Per Service:** A design pattern where each **microservice** owns its own dedicated **database**.
  * **Distributed Transaction:** A transaction that involves multiple independent data stores or **services**.
  * **Eventual Consistency:** A consistency model where data eventually becomes consistent, though there might be temporary inconsistencies.
  * **In-doubt State:** A state where a **participant** in 2PC has prepared a transaction but hasn't received a commit or rollback decision from the **coordinator**, holding locks indefinitely.
  * **Independent:** Not reliant on others for operation or deployment.
  * **Local Transaction:** A transaction that operates within a single **service's** database.
  * **Microservice:** A small, independent **service** within an application.
  * **Participant:** A resource manager (e.g., **database** or **service**) involved in a **distributed transaction** managed by 2PC.
  * **Rollback:** To undo the changes made by a transaction, restoring the **database** to its state before the transaction began.
  * **Saga:** A pattern for managing **distributed transactions** as a sequence of **local transactions**, with **compensating transactions** to handle failures.
  * **Service:** A component of an application.