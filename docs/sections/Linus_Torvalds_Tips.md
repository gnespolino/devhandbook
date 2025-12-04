## 7 Tips from Linus Torvalds to Avoid Bad Code

Linus Torvalds, the creator of Linux and Git, is legendary for his pragmatic, no-nonsense approach to software engineering. His philosophy emphasizes clarity, simplicity, and maintainability above all else. Here are 7 pieces of advice attributed to him that can help you avoid writing "garbage" code, expanded with context for modern development.

### 1. Keep it simple or don't do it

Complexity is the silent killer of software projects. The more complex your code, the harder it is to test, maintain, and extend. 
*   **Apply KISS (Keep It Simple, Stupid):** Always choose the simplest solution that solves the problem.
*   **Avoid Over-engineering:** Don't build generic frameworks for problems you *might* have in the future. Solve the problem you have today.
*   If a solution feels "clever," it's often a warning sign. Code should be boringly obvious.

### 2. Delete useless code without fear

Hoarding code "just in case" creates technical debt and confusion.
*   **Trust Version Control:** You use Git for a reason. If you delete something and realize you need it six months later, it’s safely stored in your history.
*   **Remove Dead Code:** Commented-out blocks, unused functions, and unreachable logic add cognitive load to anyone reading the file. Keep your codebase lean.

### 3. If you need comments, rewrite it

Comments often apologize for bad code. If you find yourself writing a paragraph to explain what a function does, the code itself is likely too opaque.
*   **Self-Documenting Code:** Use descriptive variable and function names. `calculateTotal(cart)` is better than `func1(x) // calculates total`.
*   **Refactor for Clarity:** Extract complex logic into well-named helper functions.
*   **The "Why" vs. The "What":** Comments should explain *why* a decision was made (business logic, specific constraints), never *what* the code is doing (syntax).

### 4. Don't mix refactors with fixes

Mixing structural changes (refactoring) with functional changes (bug fixes or features) in a single commit or pull request is a recipe for disaster.
*   **Atomic Commits:** A commit should do one thing. If you find a variable to rename while fixing a bug, do it in a separate commit.
*   **Reviewability:** It’s impossible for a reviewer to verify a bug fix if it’s buried in 500 lines of reformatting.
*   **Safety:** If the refactor breaks something, you can revert it without losing the bug fix, and vice versa.

### 5. If you can't explain it quickly, it's wrong

If you struggle to explain your design or logic to a colleague in simple terms, you probably don't understand it well enough, or you've made it too complicated.
*   **The Duck Test:** Try explaining it to a rubber duck (or a coworker). If you get stuck or sound confused, your code is likely the problem.
*   **Clear Mental Model:** Good code reflects a clear mental model of the problem domain.

### 6. Make it work first, optimize later

Premature optimization is a common trap. Developers often waste time optimizing code that isn't a bottleneck, making it unreadable in the process.
*   **Correctness First:** A fast program that gives the wrong answer is useless. Focus on writing clean, correct code first.
*   **Measure, Don't Guess:** Only optimize when you have profiling data proving a specific part of the code is slow.
*   **Readability > Micro-performance:** In most business applications, readable code is far more valuable than saving a few CPU cycles.

### 7. Small commits or you are hiding something

Large, monolithic commits are a black box. They obscure the history and intent of the changes.
*   **Granularity:** Small commits allow you to tell a story. "Add user model," "Add validation," "Update tests" is a clear narrative.
*   **Bisecting:** If a bug is introduced, finding the culprit in a 10-line commit is trivial. Finding it in a 1,000-line commit is a nightmare.
*   **Transparency:** Large dumps of code look suspicious and are exhausting to review. Show your work.

---

[Prev: Clean Code: Design Principles (SOLID)](./CC_DP_SOLID.md) | [Back to Index](../../README.md) | [Next: Coming soon](https://github.com/gnespolino)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FLinus_Torvalds_Tips&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
Happy coding! 🎉
##### An html version of this file is available [HERE](https://gnespolino.github.io/devhandbook/index.html)
