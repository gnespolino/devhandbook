## Linux for Developers: Power at Your Fingertips

As backend developers, our code usually runs on Linux servers. Developing on Windows/Mac without understanding Linux is like a race car driver who doesn't know how to change a tire.

### 1. The Shell is Your IDE

Learn to love the terminal. It’s faster and more powerful than any GUI for certain tasks.

*   **Grep:** Find text in files instantly. `grep -r "NullPointerException" /var/log`
*   **Pipe (|):** Chain commands together.
    *   `cat access.log | grep "404" | wc -l` -> Count how many 404 errors occurred.
*   **Top/Htop:** See what's eating your CPU.

### 2. File Permissions (chmod/chown)

Understand `755`, `644`, and what `rwx` means.
*   `chmod +x script.sh`: Make it executable.
*   `chown user:group file`: Change ownership.
Security breaches often happen because of bad permissions.

### 3. SSH Tunneling

Need to connect to a database inside a private VPC? Don't open a port to the world. Use an SSH Tunnel.
```bash
ssh -L 5432:localhost:5432 user@bastion-server
```
Now you can connect your local DBeaver to `localhost:5432` and it securely routes to the remote DB.

### 4. Everything is a File

In Linux, sockets are files, devices are files, processes are files (in `/proc`).
This philosophy allows standard tools to work on everything. You can `cat` a hardware device stream or `vim` a process configuration.

---

[Prev: Testing Strategies](./Testing_Strategy.md) | [Back to Index](../../README.md) | [Next: CI/CD Pipelines](./CI_CD_Pipeline.md)

![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgnespolino.github.io%2Fdevhandbook%2Fdocs%2Fsections%2FLinux_For_Developers&label=VISITORS&countColor=%23263759)

---
## License
This repository is open-source under the [MIT License](/LICENSE.md).
