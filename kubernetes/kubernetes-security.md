# 🔥 Kubernetes Security Interview Questions

### 1. How does Kubernetes handle Linux kernel level security?
**Answer:**

*   **Namespaces (Isolation):**
    *   **PID Namespace:** Prevents a pod from seeing processes in other pods or the host.
    *   **Network Namespace:** Each pod gets its own IP address and routing table.
    *   **Mount Namespace:** Restricts the pod’s view of the filesystem.
*   **Control Groups (cgroups):**
    *   **Memory Limits:** The kernel will trigger an OOM (Out of Memory) kill if a container exceeds its limit.
    *   **CPU Shares:** Ensures fair distribution of CPU cycles.
    *   **Device Whitelisting:** Limits which physical devices (`/dev/`) a container can access.
*   **Seccomp (Secure Computing Mode):**
    *   Restricts the system calls (syscalls) a process can make to the Linux kernel.
    *   In Kubernetes, you apply a Seccomp profile (e.g., `RuntimeDefault`) via the `securityContext`.
*   **AppArmor and SELinux (MAC):**
    *   Mandatory Access Control systems that enforce system-wide security policies.
    *   **AppArmor:** Path-based (e.g., "deny writing to `/etc/`").
    *   **SELinux:** Label-based (e.g., "only processes with the `web_t` label can touch files with the `web_data_t` label").
*   **Linux Capabilities:**
    *   Breaks "root" powers into small pieces.
    *   Instead of giving a container full root access, you grant only the specific privilege needed (e.g., `CAP_NET_BIND_SERVICE`).

---

### 2. Give a YAML example of a "Hardened" Pod.
**Answer:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened-web-server
spec:
  securityContext:
    # 1. Apply a Seccomp profile at the Pod level
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: web-app
    image: nginx:alpine
    securityContext:
      # 2. Prevent the process from gaining more privileges
      allowPrivilegeEscalation: false
      
      # 3. Run as a non-root user
      runAsUser: 1000
      runAsNonRoot: true

      # 4. Drop all 'root' powers, add back only what's needed
      capabilities:
        drop:
          - ALL
        add:
          - NET_BIND_SERVICE # Allows binding to a port below 1024

      # 5. Make the root filesystem read-only
      readOnlyRootFilesystem: true
```
*   **seccompProfile:** Filters dangerous system calls.
*   **allowPrivilegeEscalation: false:** Prevents a process from gaining more privileges than its parent.
*   **runAsNonRoot: true:** Enforces that the container runs as a non-root user.
*   **capabilities: drop: - ALL:** Removes all default Linux capabilities.
*   **readOnlyRootFilesystem: true:** Prevents attackers from modifying the container's filesystem.

---

### 3. How do AppArmor and SELinux operate in Kubernetes?
**Answer:**
They are Mandatory Access Control (MAC) systems that enforce security policies at the kernel level.

| Feature | AppArmor | SELinux |
| :--- | :--- | :--- |
| **Logic** | **Path-based:** Rules are tied to file paths. | **Label-based:** Rules are tied to object labels. |
| **Default Distros** | Ubuntu, Debian, SUSE. | RHEL, CentOS, Fedora. |
| **K8s config** | Refers to a profile name on the host (`securityContext.appArmorProfile`). | Defines specific security context labels (`securityContext.seLinuxOptions`). |

---

### 4. How do you automate the loading of AppArmor profiles across a large cluster?
**Answer:**
Use a **DaemonSet**.

1.  **Store Profiles in a ConfigMap:** Place your AppArmor profile text into a ConfigMap.
2.  **Create a Loader DaemonSet:**
    *   This Pod runs on every node.
    *   It mounts the host's AppArmor directory (`/etc/apparmor.d`).
    *   It runs a script (e.g., using `apparmor_parser`) to load the profiles from the ConfigMap into the host's kernel.
    *   The DaemonSet container needs `privileged: true` to interact with the host kernel.

---

### 5. How would you enforce that no developer can deploy a pod unless it has these security settings enabled?
**Answer:**
Use **Pod Security Admission (PSA)**.

1.  **PSA Levels:** PSA provides three levels: `Privileged`, `Baseline`, and `Restricted`. The `Restricted` policy is the most secure and enforces many of the hardening practices.
2.  **Enforcement:** You apply the policy by labeling a namespace.
    ```bash
    kubectl label --overwrite ns production-apps \
      pod-security.kubernetes.io/enforce=restricted
    ```
3.  **Rejection:** Once labeled, the Kubernetes API will reject any Pod that doesn't meet the `Restricted` standard (e.g., requires `runAsNonRoot: true`, specific Seccomp profiles, etc.).

---

### 6. What is an admission controller in Kubernetes?
**Answer:**
An admission controller is a component that intercepts requests to the Kubernetes API server *after* authentication and authorization but *before* the object is persisted in etcd. It can validate or mutate requests to enforce policies.

**Request Flow:**
`kubectl request` → `Authentication` → `Authorization (RBAC)` → `Admission Controllers` → `etcd`

---

### 7. What are the two main types of admission controllers?
**Answer:**

1.  **Mutating Admission Controllers:**
    *   **Function:** Modify the request object.
    *   **Examples:** Injecting sidecars (like Istio), adding default labels, or setting default resource limits.
2.  **Validating Admission Controllers:**
    *   **Function:** Accept or reject the request.
    *   **Examples:** Enforcing security policies with OPA Gatekeeper or Kyverno, blocking privileged containers, or ensuring images come from a trusted registry.

**Order:** Mutating controllers run **first**, so validating controllers can check the final, modified object.

---

### 8. What are Validating Admission Policies?
**Answer:**
Validating Admission Policies are a **native** Kubernetes feature that allows you to write validation rules using **CEL (Common Expression Language)** directly in a Kubernetes object, without needing to run an external webhook server (like OPA Gatekeeper).

*   **How it works:** The API server evaluates the CEL expression against the incoming object. If the expression returns `false`, the request is rejected.
*   **Benefits:** No network calls, no extra pods to manage, simpler to use for common validation tasks.
*   **Note:** This is a newer feature and may not be enabled by default in all cloud providers (e.g., older EKS versions).

---

### Summary Tables

**Security Responsibility Chain:**

| Layer | Responsibility | Component |
| :--- | :--- | :--- |
| **Cluster** | Enforcement (The "Law") | Pod Security Admission (PSA) |
| **Node** | Distribution (The "Carrier") | DaemonSet (Profile Loader) |
| **Kernel** | Execution (The "Enforcer") | AppArmor / SELinux / Seccomp |

**PSA Enforcement Levels for AppArmor:**

| PSA Level | AppArmor Requirement | Resulting Kernel State |
| :--- | :--- | :--- |
| **Privileged** | No requirements. | Often Unconfined (Kernel protection is OFF). |
| **Baseline** | Disallows explicit `Unconfined` setting. | Defaults to `RuntimeDefault` (Kernel protection is ON). |
| **Restricted** | Requires `RuntimeDefault` or a specific `Localhost` profile. | Maximum Isolation (Strict Kernel filtering). |
