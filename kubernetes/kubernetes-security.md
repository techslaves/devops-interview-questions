# Kubernetes Security Interview Questions and Answers

This document contains common interview questions related to Kubernetes Security and their answers.

## 1. How does Kubernetes handle Linux kernel level security?
**Answer:**

**1. Namespaces (Isolation)**

Namespaces are the fundamental building block of containers. They provide the illusion that a process is running in its own dedicated environment. The kernel ensures that a process in one namespace cannot see or affect processes in another.

PID Namespace: Prevents a pod from seeing processes in other pods or the host.

Network Namespace: Each pod gets its own IP address and routing table.

Mount Namespace: Restricts the pod’s view of the filesystem.

**2. Control Groups (cgroups)**

While Namespaces provide isolation, cgroups provide resource management. They prevent a single container from consuming all system resources (a "noisy neighbor" or DoS attack).

Memory Limits: The kernel will trigger an OOM (Out of Memory) kill if a container exceeds its limit.

CPU Shares: Ensures fair distribution of CPU cycles.

Device Whitelisting: Limits which physical devices (/dev/) a container can access.

**3. Seccomp (Secure Computing Mode)**

Seccomp restricts the system calls (syscalls) a process can make to the Linux kernel. Even if an attacker gains shell access to your container, Seccomp can prevent them from executing dangerous commands like mount or ptrace.

Implementation: In Kubernetes, you apply a Seccomp profile (usually a JSON file) via the securityContext.

The "RuntimeDefault" Profile: Kubernetes can automatically apply a default profile that blocks roughly 50 dangerous syscalls out of the 300+ available.

**4. AppArmor and SELinux (MAC)**

These are Mandatory Access Control (MAC) systems. Unlike standard Linux permissions (where the "owner" decides access), MAC allows the administrator to define system-wide security policies.

AppArmor: Uses program paths to restrict capabilities (e.g., "this application cannot write to /etc/"). It is common on Ubuntu/Debian.

SELinux: Uses labels (contexts) assigned to every file, process, and port. It is the default on RHEL/Fedora/OpenShift.

Kubernetes usage: You can label your Pods so the kernel prevents them from accessing sensitive host files, even if the pod is running as "root."

**5. Linux Capabilities**

Traditionally, Linux had a binary choice: you were either a normal user or the all-powerful root. Capabilities break "root" powers into small pieces.

Instead of giving a container full root access, Kubernetes allows you to grant only the specific privilege needed:

CAP_NET_BIND_SERVICE: Allows a process to bind to ports below 1024.

CAP_CHOWN: Allows changing file ownership.

Best Practice: Use allowPrivilegeEscalation: false in your Pod Security Standards to prevent processes from gaining more permissions than they started with.
