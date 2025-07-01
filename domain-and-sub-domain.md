## Microservices Design Patterns: Decomposition Pattern

### 1. Domain and Sub-Domain Pattern

* **What it is:** This pattern is a strategy for breaking down a large, complex application (a **monolith**) into smaller, manageable **microservices**. It's based on the idea of **Domain-Driven Design (DDD)**, where you identify the core business areas and their specific responsibilities.

* **How it works:**
    1.  **Identify the Core Domain:** This is the central, most critical part of your business (e.g., in e-commerce, it might be "Order Fulfillment").
    2.  **Identify Subdomains:** These are supporting parts of your business. Subdomains can be:
        * **Core Subdomains:** Directly contribute to your competitive advantage (e.g., "Pricing Engine" for dynamic pricing).
        * **Supporting Subdomains:** Necessary but not unique to your business; they support core activities (e.g., "User Authentication").
        * **Generic Subdomains:** Common functionality that doesn't provide a competitive advantage and can often be bought off-the-shelf (e.g., "Email Sending Service").
    3.  **Define Bounded Contexts:** For each subdomain, define its "bounded context." This is a clear boundary within which a specific domain model (terms, rules, and data) is consistent and applies.
    4.  **Map Services to Bounded Contexts:** Each **bounded context** then becomes a strong candidate for its own **microservice**. This ensures **high cohesion** within the service and helps define clear **APIs** between services.

* **Simple Example:**
    Imagine an online bookstore (**Monolith**).

    * **Core Domain/Subdomain:** "Book Catalog Management" (what books are available, details, pricing). This could be one **microservice**.
    * **Supporting Subdomain:** "User Account Management" (user profiles, addresses, payment methods). This could be another **microservice**.
    * **Supporting Subdomain:** "Order Processing" (handling customer orders, shipping details). This becomes a third **microservice**.
    * **Generic Subdomain:** "Notification Service" (sending order confirmation emails). This might be a separate, reusable **microservice**.

    By doing this, each **microservice** owns its specific domain logic and data, promoting **independent** development and deployment.

* **Benefit:** Leads to **services** that are truly **autonomous**, **highly cohesive**, and well-aligned with business needs, making the system easier to understand, manage, and scale.

* **Recall:** **Monolith**, **Microservice**, **API**, **High Cohesion**, **Independent**, **Service**.

---

### Key Terms (Reference)

* **API (Application Programming Interface):** Rules for software communication.
* **Autonomous:** Able to operate independently.
* **Bounded Context:** A logical boundary within **Domain-Driven Design** where a specific domain model is consistent. Often maps to a **microservice**.
* **Business Capability:** A specific function a business performs.
* **Domain-Driven Design (DDD):** A software development approach that focuses on modeling software to match a domain of business knowledge.
* **High Cohesion:** The principle that all elements within a **service** are strongly related to its specific **business capability**.
* **Independent:** Not reliant on others for operation or deployment.
* **Microservice:** A small, independent **service** within an application.
* **Monolith:** A single, large, unified application.
* **Service:** A component of an application.