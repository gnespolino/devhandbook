## Java Beyond the Basics: Modern Practices

Java has evolved significantly. If you are still writing code like it's Java 8 (or worse, Java 5), you are missing out on features that make code safer, more concise, and easier to read.

### 1. Embrace Immutability with Records (Java 14+)

For years, we relied on Lombok's `@Data` or wrote boilerplate `getters`, `hashCode`, and `equals`. Java Records solve this natively. They are immutable data carriers.

**Old Way (Lombok or Verbose):**
```java
public class UserRequest {
    private final String username;
    private final String email;

    public UserRequest(String username, String email) {
        this.username = username;
        this.email = email;
    }
    // + Getters, equals, hashCode, toString...
}
```

**Modern Way:**
```java
public record UserRequest(String username, String email) {
    // Validation can go in the compact constructor
    public UserRequest {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
}
```
Records communicate intent: "This is just data."

### 2. Better Null Handling: `Optional` is not for Fields

`Optional<T>` is a great return type for methods that might not return a value. It forces the caller to handle the absence of data.

**Do:**
```java
public Optional<User> findByEmail(String email) { ... }
```

**Don't:**
Use it for class fields or method arguments. It adds overhead and serialization issues.
```java
// BAD
public void process(@NotNull Optional<User> user) { ... } 
```

### 3. Switch Expressions (Java 14+)

The old `switch` statement was error-prone (missing `break` statements). The new switch expression is exhaustive and returns a value.

```java
var type = switch (status) {
    case PENDING -> "Waiting";
    case APPROVED, VERIFIED -> "Done";
    case REJECTED -> "Failed";
    default -> throw new IllegalStateException("Unknown status: " + status);
};
```

### 4. Avoid "Exception Driven Logic"

Exceptions are expensive (stack trace generation) and disrupt control flow. Don't use them for expected business outcomes.

**Bad Practice:**
```java
try {
    userService.login(user);
} catch (UserNotFoundException e) {
    registerUser(user);
}
```

**Better Practice (Result Pattern):**
Return a wrapper object (like `Result<Success, Error>`) or use simple checks.
```java
if (userService.exists(user)) {
    userService.login(user);
} else {
    registerUser(user);
}
```

---

[Prev: 7 Tips from Linus Torvalds](./Linus_Torvalds_Tips.md) | [Back to Index](../../README.md) | [Next: The Truth About Microservices](./Microservices_Reality_Check.md)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FJava_Modern_Practices&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
