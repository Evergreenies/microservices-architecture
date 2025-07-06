## Microservices Design Patterns: Communication Patterns

### 1\. Synchronous Communication 📞

  * **What it is:** **Synchronous communication** is a **request-response** model where the client (the calling **service** or application) sends a request to another **service** and then **waits** for an immediate response. The client's execution is **blocked** until it receives a reply or a timeout occurs. It's like a phone call where both parties need to be present and respond in real-time.

  * **How it works:**

    1.  The client initiates a request to a target **service**.
    2.  The client's process pauses and waits.
    3.  The target **service** processes the request.
    4.  The target **service** sends a response back to the client.
    5.  The client receives the response and resumes its processing.

-----

### Synchronous Communication - Markdown Diagram:

```
+--------------------+            HTTP/gRPC/REST Call            +--------------------+
| [ Client Service ] | -----------------------------------------> | [ Target Service ] |
| (Waits for response)|                                           | (Processes request)|
+--------------------+ <----------------------------------------- +--------------------+
      (Response)
```

**Explanation of the Markdown Diagram:**

  * **Client Service:** Initiates the communication and blocks its own execution.
  * **Target Service:** Receives and processes the request, then sends back a response.
  * The arrow from Client to Target represents the request, and the arrow back represents the response. The `(Waits for response)` on the client side highlights the blocking nature.

-----

  * **Simple Example:**
    Imagine a `Web UI` calling an `Order Service` to fetch order details.

    1.  The `Web UI` sends an **HTTP GET** request to `/orders/{orderId}` on the `Order Service`.
    2.  The `Web UI`'s thread waits.
    3.  The `Order Service` receives the request, queries its **database**, and prepares the order details.
    4.  The `Order Service` sends back an **HTTP 200 OK** response with the order data.
    5.  The `Web UI` receives the data, unblocks, and displays the order details to the user.

  * **AWS Services for Synchronous Communication:**

      * **Amazon API Gateway:** Serves as the front door for **APIs**, routing incoming **HTTP** requests to backend **services**.
      * **AWS Lambda:** Often invoked synchronously via **API Gateway** or direct function calls.
      * **Amazon EC2** / **Amazon ECS** / **Amazon EKS:** Host **microservices** that expose **RESTful APIs** (or gRPC) via **HTTP** endpoints, typically fronted by **Elastic Load Balancers** (ELB).
      * **Elastic Load Balancing (ELB):** Distributes incoming application traffic across multiple targets, like EC2 instances or containers, handling **HTTP** requests.

-----

### Advantages (Pros):

  * **Simplicity:** Conceptually straightforward to implement and understand, as it mimics traditional function calls.
  * **Immediate Feedback:** The client gets an immediate response, indicating success or failure, which is useful for user-facing operations requiring real-time validation.
  * **Easier Error Handling:** Errors can often be handled directly within the request-response cycle.

-----

### Disadvantages (Cons):

  * **Tight Coupling:** The client and **service** are tightly coupled in time; both must be available for the communication to succeed. If the target **service** is down or slow, the client is affected.
  * **Reduced Resilience:** A failure in a downstream **service** can cascade back to the upstream client, leading to degraded performance or timeouts.
  * **Limited Scalability:** If a **service** becomes a **bottleneck**, it can slow down or block many clients. Clients might need to hold resources while waiting.
  * **Lack of Asynchronous Operations:** Not suitable for long-running processes, as the client would have to wait indefinitely.

-----

### Key Terms (Reference)

  * **API (Application Programming Interface):** Rules for software communication.
  * **Amazon API Gateway:** An AWS service for creating, publishing, maintaining, monitoring, and securing **RESTful APIs**.
  * **Amazon EC2 (Elastic Compute Cloud):** A web service that provides resizable compute capacity in the cloud.
  * **Amazon ECS (Elastic Container Service):** A highly scalable, high-performance **container** orchestration service.
  * **Amazon EKS (Elastic Kubernetes Service):** A managed Kubernetes service.
  * **AWS Lambda:** A serverless compute service that runs code in response to events.
  * **Blocked:** A state where a process or thread pauses its execution until an external event (like a response) occurs.
  * **Bottleneck:** A point in a system where the flow is restricted.
  * **Client:** The component or **service** that initiates a request.
  * **Container:** A lightweight, portable package for software.
  * **Database:** An organized collection of data.
  * **Elastic Load Balancing (ELB):** Distributes incoming application traffic across multiple targets.
  * **HTTP (Hypertext Transfer Protocol):** The foundation of data communication for the World Wide Web.
  * **Microservice:** A small, independent **service** within an application.
  * **Request-Response:** A communication pattern where a client sends a request and expects a response.
  * **Resilience:** Ability of a system to recover from failures.
  * **RESTful APIs (Representational State Transfer APIs):** A set of architectural constraints for designing web **services**, typically over **HTTP**.
  * **Service:** A component of an application.
  * **Synchronous:** Occurring at the same time; in communication, implying waiting for a response.
  * **Timeout:** A predefined period after which an operation is automatically terminated if it hasn't completed.