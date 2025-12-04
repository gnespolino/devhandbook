## Testing Strategies that Don't Suck

Testing is not about checking off a box for "Code Coverage". It's about **confidence**. Confidence to refactor, confidence to ship, and confidence that you haven't broken existing features.

### 1. The Testing Pyramid (Real World Edition)

*   **Unit Tests (70%):** Fast, isolated, cheap. Test logic, not frameworks.
*   **Integration Tests (20%):** Test the wiring. Does my Service talk to the Database correctly? Does the Controller return the right JSON?
*   **E2E (End-to-End) Tests (10%):** Slow, brittle. Test critical user journeys (e.g., "Login -> Buy Item").

### 2. Mockito: A Double-Edged Sword

Mocks are useful to isolate the Unit under test, but **over-mocking** leads to fragile tests.

**Bad Test (Testing the Implementation):**
```java
// If you change the internal implementation to use a different method, this test fails!
verify(repository, times(1)).findById(any());
```

**Good Test (Testing the Behavior):**
Focus on the input and output.
```java
User result = service.getUser(1);
assertEquals("John", result.getName());
```

### 3. Testcontainers > In-Memory DBs

Don't use H2 for testing if you use PostgreSQL in production. They behave differently (syntax, locking, constraints).
Use **Testcontainers**. It spins up a real Docker container with your actual database version for Integration Tests.

```java
@Testcontainers
class UserRepositoryTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");
    
    // Run tests against real Postgres
}
```

### 4. TDD (Test Driven Development)

TDD is not about writing tests. It's about **designing APIs**.
1.  **Red:** Write a test that fails. (Forces you to think: "How do I want to call this method?")
2.  **Green:** Write just enough code to pass. (Prevents over-engineering.)
3.  **Refactor:** Clean it up. (Safe because you have the test.)

If you write tests *after* the code, you often write tests that fit the implementation, rather than tests that verify the requirements.

---

[Prev: The Truth About Microservices](./Microservices_Reality_Check.md) | [Back to Index](../../README.md) | [Next: Linux for Developers](./Linux_For_Developers.md)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FTesting_Strategy&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
