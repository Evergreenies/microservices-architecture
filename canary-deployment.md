## Microservices Design Patterns: Deployment Patterns

### 2\. Canary Deployment 🐤

  * **What it is:** **Canary Deployment** is a deployment strategy that releases a new version of a **microservice** to a small, controlled subset of users or servers first. This "canary" group is carefully **monitored** for performance, errors, and user feedback. If the new version performs well and no issues are detected, it's gradually rolled out to the rest of the user base. If problems arise, the impact is limited to the small canary group, and the change can be quickly **rolled back**. It's named after the "canary in the coal mine" practice.

  * **How it works:**

    1.  **Deploy Canary:** A new version (v2) of the **microservice** is deployed to a small number of servers or instances, forming the "canary group." The vast majority of traffic still goes to the old version (v1).
    2.  **Route Small Percentage of Traffic:** A very small percentage of live production traffic (e.g., 1-5%) is routed to the canary group.
    3.  **Monitor Intensively:** The canary group's performance, error rates, latency, resource utilization, and business metrics are **monitored** intensely. User feedback (if applicable) is also gathered.
    4.  **Evaluate & Decide:**
          * **If successful:** If the canary performs well, more traffic is gradually shifted to the new version in increments (e.g., 25%, 50%, 100%), with continued **monitoring** at each stage.
          * **If issues:** If issues are detected, traffic is immediately diverted back to the old version (v1), limiting the blast radius of the problem.
    5.  **Full Rollout/Rollback:** Once the new version handles 100% of the traffic successfully, the old version can be decommissioned. If issues prevent a full rollout, the canary is rolled back.

-----

### Canary Deployment - Markdown Diagram:

```
                          [ Users ]
                             |
                             | Traffic Router / Load Balancer
                             V
+-----------------------------------------------------------------+
|                                                                 |
|   ( Major Traffic )    ---------------------------------------> | [ Old Version ] (v1)
|                                                                 |
|   ( Small % Traffic )  ----------------+                       |
|                                        V                       |
|                               +-------------------+            |
|                               | [ Canary Group ]  |            |
|                               | (New Version - v2)|            |
|                               +-------------------+            |
|                                        ^                        |
|                                        |                        |
|                                 (Intensive Monitoring)          |
+-----------------------------------------------------------------+

( IF SUCCESSFUL, GRADUALLY SHIFT MORE TRAFFIC )

( IF FAILED, SHIFT ALL TRAFFIC BACK TO OLD VERSION )
```

**Explanation of the Markdown Diagram:**

  * **Users:** Your end-users sending requests.
  * **Traffic Router / Load Balancer:** The mechanism that intelligently splits traffic between the old version and the canary group.
  * **Old Version (v1):** The current, stable production version handling most of the traffic.
  * **Canary Group (v2):** A small set of instances running the new version, receiving a controlled amount of traffic.
  * **Intensive Monitoring:** Crucial for detecting issues quickly in the canary group.

-----

  * **Simple Example:**
    You have a `Recommendation Service` that suggests products to users (v1). You've developed v2 with an improved recommendation algorithm.

    1.  You deploy `Recommendation Service v2` to 5% of your **Lambda** functions or **ECS** tasks.
    2.  You configure your **API Gateway** or **Application Load Balancer** to send 5% of all `Recommendation Service` requests to these v2 instances, with the remaining 95% still going to v1.
    3.  You **monitor** key metrics:
          * Error rates from v2 instances.
          * Latency of v2 responses.
          * Conversion rates or user engagement metrics for the canary group.
    4.  If after 30 minutes, all metrics for v2 are stable or improved, you gradually increase traffic to 25%, then 50%, then 100%.
    5.  If, at any point, v2 shows a spike in errors or a drop in engagement, you immediately shift 100% of traffic back to v1.

  * **AWS Services for Canary Deployment:**

      * **Amazon Route 53:** Can use weighted routing policies to direct a percentage of traffic to a different set of resources.
      * **Elastic Load Balancing (ELB) - Application Load Balancer (ALB):** Can use target group weighting to direct a percentage of requests to a new target group containing the canary instances.
      * **AWS CodeDeploy:** Specifically supports automated canary deployments for **EC2**/On-premises, **Lambda**, and **ECS** by gradually shifting traffic and rolling back based on **CloudWatch** alarms.
      * **Amazon ECS / Amazon EKS:** **Container** orchestration platforms allow deploying new versions as a separate task set/deployment and gradually shifting traffic to it.
      * **AWS Lambda:** Supports versioning and aliases, allowing you to easily shift a percentage of traffic to a new **Lambda** function version.
      * **Amazon CloudWatch / AWS X-Ray / Amazon Managed Service for Prometheus / Amazon Grafana:** Essential for comprehensive **monitoring** of the canary group's health and performance.

-----

### Advantages (Pros):

  * **Reduced Risk:** Limits the impact of potential issues to a small subset of users, preventing a wide-scale outage.
  * **Real-World Testing:** Allows testing the new version with live production traffic and real user behavior before a full rollout.
  * **Faster Rollback:** If problems occur, reverting to the old version is quick, often just by shifting traffic back.
  * **Gradual Rollout:** Provides time to **monitor** and respond to issues, improving confidence in the new release.
  * **A/B Testing Capable:** Can be extended to perform **A/B testing** on features for specific user segments.

-----

### Disadvantages (Cons):

  * **Increased Complexity:** More complex to set up, manage, and **monitor** than simple in-place deployments. Requires robust **monitoring** and automated rollback mechanisms.
  * **Longer Deployment Cycle:** The phased rollout can take longer than a direct full deployment.
  * **Partial User Experience:** A small segment of users might experience issues or features that the general user base doesn't, which can be confusing for support or for user feedback.
  * **Database Migrations:** Still requires careful handling of **database** schema changes, ensuring forward and backward compatibility as both old and new versions might be accessing the same **database** concurrently.

-----

### Key Terms (Reference)

  * **A/B Testing:** A method of comparing two versions of a single variable to determine which performs better.
  * **Amazon CloudWatch:** An AWS **monitoring** and observability service.
  * **Amazon ECS (Elastic Container Service):** A highly scalable, high-performance **container** orchestration service.
  * **Amazon EKS (Elastic Kubernetes Service):** A managed Kubernetes service.
  * **Amazon Route 53:** A scalable cloud DNS web service.
  * **AWS CodeDeploy:** An AWS service that automates software deployments.
  * **AWS Lambda:** A serverless compute service.
  * **AWS X-Ray:** An AWS service that helps developers analyze and debug distributed applications.
  * **Canary Deployment:** A deployment strategy that gradually releases a new version to a small subset of users/servers.
  * **Container:** A lightweight, portable package for software.
  * **Database:** An organized collection of data.
  * **Deployment Pattern:** A common strategy for how applications or **services** are packaged and run.
  * **Elastic Load Balancing (ELB):** Distributes incoming application traffic across multiple targets.
  * **Load Balancer:** Distributes incoming network traffic across multiple servers or instances.
  * **Microservice:** A small, independent **service** within an application.
  * **Monitoring:** Tracking the health and performance of systems.
  * **Prometheus:** An open-source **monitoring** system with a time-series **database**.
  * **Rollback:** Reverting to a previous, stable version of a system or **service**.
  * **Service:** A component of an application.