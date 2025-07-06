## Microservices Design Patterns: Deployment Patterns

### 1\. Blue/Green Deployment 🟦🟩

  * **What it is:** **Blue/Green Deployment** is a deployment strategy that minimizes downtime and reduces risk by running two identical production environments, "Blue" (the current live version) and "Green" (the new version). At any given time, only one environment is live, serving all production traffic. When a new version is ready, it's deployed to the inactive environment, tested thoroughly, and then traffic is switched over.

  * **How it works:**

    1.  **Provision Environments:** You set up two identical production environments (e.g., two sets of **EC2 instances**, **containers**, or **Lambda** versions). Let's call the currently active one "Blue" (running v1 of your **service**) and the inactive one "Green".
    2.  **Deploy New Version:** The new version (v2) of your **microservice** is deployed to the "Green" environment.
    3.  **Test Green:** Extensive testing is performed on the "Green" environment, often using synthetic traffic or internal users, to ensure the new version works correctly without impacting live users.
    4.  **Switch Traffic:** Once testing is complete and confidence is high, the **load balancer** or traffic router (e.g., **Route 53**) is updated to switch all incoming production traffic from the "Blue" environment to the "Green" environment. This switch is typically very fast.
    5.  **Monitor Green:** The "Green" environment is carefully **monitored** in production.
    6.  **Decommission/Retain Blue:** If the "Green" deployment is successful, the "Blue" environment can either be decommissioned, or kept as a rollback option for a period before being updated with the next new version. If issues arise, traffic can be quickly rolled back to "Blue."

-----

### Blue/Green Deployment - Markdown Diagram:

```
                           [ Users ]
                               |
                               | Traffic Router / Load Balancer
                               V
+-----------------------------------------------------------------+
|                                                                 |
|   ( Currently Live Traffic )   -------------------------------> | [ Blue Environment ]  (Old Version - v1)
|                                                                 |  (e.g., EC2, ECS, Lambda)
|   ( New Version Deploy )       +--------------------------+   |
|                                | [ Green Environment ]    |   |
|                                | (New Version - v2)       |   |
|                                | (e.g., EC2, ECS, Lambda) |   |
|                                +--------------------------+   |
|                                                                 |
+-----------------------------------------------------------------+

( AFTER SWITCHING TRAFFIC )

                           [ Users ]
                               |
                               | Traffic Router / Load Balancer
                               V
+-----------------------------------------------------------------+
|                                                                 |
|   ( New Live Traffic )   ------------------------------------> | [ Green Environment ] (New Version - v2)
|                                                                 |  (e.g., EC2, ECS, Lambda)
|   ( Blue Environment )                                          |
|   (Ready for Rollback or Decommission)                          |
+-----------------------------------------------------------------+
```

**Explanation of the Markdown Diagram:**

  * **Users:** Your end-users sending requests.
  * **Traffic Router / Load Balancer:** The mechanism that directs traffic to either the Blue or Green environment.
  * **Blue Environment:** The currently live production environment (running the old version).
  * **Green Environment:** The new production environment where the new version is deployed and tested.
  * The diagram shows the traffic initially directed to "Blue," then switching entirely to "Green" after successful deployment and testing.

-----

  * **Simple Example:**
    You have an `Authentication Service` running in production (v1). You've developed v2 with new security features.

    1.  Your current live environment is "Blue" running `Authentication Service v1`.
    2.  You provision an identical "Green" environment.
    3.  You deploy `Authentication Service v2` to the "Green" environment.
    4.  You run automated and manual tests against the "Green" environment to ensure v2 works perfectly.
    5.  Once confident, you update your **Elastic Load Balancer** (ELB) to point all incoming requests for the `Authentication Service` from the "Blue" targets to the "Green" targets.
    6.  Users immediately start using `Authentication Service v2`. If any critical issues are found, you quickly revert the ELB's target group back to "Blue" (v1), ensuring minimal impact.

  * **AWS Services for Blue/Green Deployment:**

      * **Amazon Route 53:** Can manage DNS traffic routing, allowing you to update DNS records to switch traffic between environments (e.g., weighted routing, simple routing).
      * **Elastic Load Balancing (ELB):** Essential for directing traffic to the active environment. You can switch target groups or listeners to move traffic between Blue and Green application instances (**EC2**, **ECS**, **EKS**).
      * **AWS CodeDeploy:** Specifically designed for Blue/Green deployments for **EC2**/On-premises and **Lambda**. It automates the traffic shifting and rollback capabilities.
      * **Amazon ECS / Amazon EKS:** For **containerized microservices**, you can manage Blue/Green deployments by provisioning new task sets/deployments for the "Green" version and then switching the **load balancer** to point to the new tasks.
      * **AWS Lambda:** Supports versioning and aliases, allowing you to create a new **Lambda** version (Green) and then shift traffic to it using aliases, with easy rollback.
      * **AWS CloudFormation / AWS CDK:** For provisioning and managing identical Blue and Green infrastructure stacks as code.

-----

### Advantages (Pros):

  * **Zero Downtime Deployment:** Users experience no downtime during the deployment switch, as the new version is fully running and tested before going live.
  * **Fast Rollback:** If issues are found in the new "Green" version, traffic can be instantly switched back to the stable "Blue" environment, significantly reducing outage time.
  * **Reduced Risk:** New versions are thoroughly tested in a production-like environment before going live.
  * **Testing in Production:** Allows for final verification in a production-scale environment before exposure to live users.

-----

### Disadvantages (Cons):

  * **Increased Infrastructure Cost:** Requires maintaining two full production environments simultaneously, which doubles infrastructure costs during the deployment process.
  * **Database Migrations:** Handling **database** schema changes in Blue/Green can be complex. You need to ensure the **database** schema is compatible with both the old ("Blue") and new ("Green") versions of the application, especially if a rollback is needed. This often involves forward and backward compatible changes.
  * **Complex State Management:** If **microservices** maintain session state, ensuring a seamless transition without losing user sessions can be tricky.
  * **Deployment Complexity:** Setting up and managing the two environments and the traffic switching mechanism adds operational complexity.

-----

### Key Terms (Reference)

  * **API (Application-Programming Interface):** Rules for software communication.
  * **Amazon EC2 (Elastic Compute Cloud):** A web service that provides resizable compute capacity in the cloud (a **virtual machine**).
  * **Amazon ECS (Elastic Container Service):** A highly scalable, high-performance **container** orchestration service.
  * **Amazon EKS (Elastic Kubernetes Service):** A managed Kubernetes service.
  * **Amazon Route 53:** A scalable cloud DNS web service.
  * **AWS CodeDeploy:** An AWS service that automates software deployments.
  * **AWS CloudFormation / AWS CDK:** Infrastructure as Code services for provisioning AWS resources.
  * **AWS Lambda:** A serverless compute service.
  * **Blue/Green Deployment:** A deployment strategy using two identical environments for seamless transitions.
  * **Container:** A lightweight, portable package for software.
  * **Database:** An organized collection of data.
  * **Deployment Pattern:** A common strategy for how applications or **services** are packaged and run.
  * **Downtime:** A period when a system or service is unavailable.
  * **Elastic Load Balancing (ELB):** Distributes incoming application traffic across multiple targets.
  * **Load Balancer:** Distributes incoming network traffic across multiple servers or instances.
  * **Microservice:** A small, independent **service** within an application.
  * **Monitoring:** Tracking the health and performance of systems.
  * **Resilience:** Ability of a system to recover from failures.
  * **Rollback:** Reverting to a previous, stable version of a system or **service**.
  * **Service:** A component of an application.
  * **Virtual Machine (VM):** A software computer that, like a physical computer, runs an operating system and applications.