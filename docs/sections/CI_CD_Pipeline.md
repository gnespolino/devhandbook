## CI/CD: The Safety Net

CI/CD (Continuous Integration / Continuous Deployment) is not just about automating deployments. It's about **reducing fear**.

### 1. Continuous Integration (CI)

The goal: **Main branch is always green.**

Every time you push code:
1.  **Compile:** Does it build?
2.  **Test:** Do unit tests pass?
3.  **Lint:** Is the code style correct? (Checkstyle, Spotless).
4.  **Security Scan:** Are there vulnerable dependencies? (OWASP Dependency Check).

If any of these fail, the code is rejected. This prevents "it works on my machine" syndrome.

### 2. Continuous Deployment (CD)

The goal: **Click a button (or do nothing) to go live.**

Manual deployments are slow and error-prone (copy-pasting JARs, editing config files by hand).
A CD pipeline:
1.  Builds the Docker Image.
2.  Pushes it to a Registry.
3.  Updates the Kubernetes cluster / Server.

### 3. Pipeline as Code

Define your pipeline in Git (e.g., `.github/workflows/main.yml` or `Jenkinsfile`).
Do not configure build steps in the UI of the CI tool.
If the pipeline configuration is code, it can be versioned, reviewed, and rolled back.

### 4. Shift Left

Test early, fail early.
Don't wait for QA to find bugs. Run tests in the PR. Run security scans in the PR.
The earlier you find a bug, the cheaper it is to fix.

---

[Prev: Linux for Developers](./Linux_For_Developers.md) | [Back to Index](../../README.md) | [Next: Event-Driven Architecture](./Event_Driven_Architecture.md)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FCI_CD_Pipeline&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
