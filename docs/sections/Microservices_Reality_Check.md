## The Truth About Microservices

Microservices have become the default answer to "how should we build this?" regardless of the question. While powerful, they come with a massive hidden tax: **complexity**.

### 1. Distributed Systems are Hard

In a Monolith, function calls are fast, reliable, and in-memory.
In Microservices, function calls become network requests. Network requests are:
*   Slow (latency).
*   Unreliable (timeouts, dropped packets).
*   Insecure (need encryption).

**Fallacy:** "Microservices make things simpler."
**Reality:** They replace code complexity with infrastructure complexity.

### 2. The Modular Monolith (Modulith)

Before splitting your app into 10 services, consider a Modular Monolith.
*   **Single Deployable Unit:** Easy to test, easy to deploy.
*   **Strict Boundaries:** Use packages/modules (Java 9+ Modules, Gradle subprojects) to enforce separation.
*   **Separate Databases (Logical):** Different modules can own different tables (schemas) within the same DB instance.

If your modules are tightly coupled in a monolith, splitting them into microservices will only create a **Distributed Monolith**—the worst of both worlds.

### 3. When to use Microservices?

Use them only when you have a specific problem that a monolith cannot solve:
1.  **Independent Scaling:** The Video Processing module needs 100 CPUs, but the User Profile module needs only 1.
2.  **Organizational Scale:** You have 50+ developers. At that scale, stepping on each other's toes in a single repo becomes a bottleneck.
3.  **Polyglot Requirements:** One part needs to be in Python (Data Science), another in Java (Backend), another in Go (High concurrency).

### 4. Data Consistency is the Boss

In a monolith, you have ACID transactions. `COMMIT` and it's done.
In microservices, you have **Eventual Consistency**.
*   Service A charges the card.
*   Service B ships the order.
*   What if Service A succeeds but Service B fails? You need **Sagas** and **Compensating Transactions**.

Are you ready to write rollback logic for every business process? If not, stick to the Monolith.

---

[Prev: Java Modern Practices](./Java_Modern_Practices.md) | [Back to Index](../../README.md) | [Next: Testing Strategies](./Testing_Strategy.md)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FMicroservices_Reality_Check&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
