# Kubernetes Interview Questions

This document contains common interview questions related to Kubernetes and their answers.

### 1. What’s the difference between taints/tolerations and node affinity? When would you use each?
**Answer:**
*   **Taints/Tolerations:** Used to **repel** pods from nodes.
    *   *Use Case:* Dedicated nodes (e.g., GPU nodes) where you only want specific workloads to run.
*   **Node Affinity:** Used to **attract** pods to nodes.
    *   *Use Case:* Ensuring a pod runs on a node with specific hardware (e.g., SSD) or in a specific zone.

**Key Difference:** Taints ensure pods *don't* go where they shouldn't. Affinity ensures pods *do* go where they should.

### 2. What are Kubernetes probes?
**Answer:**
Probes are health checks used by Kubernetes to decide when to restart a container, when to send traffic, or when a container has started successfully.

**Types:**
*   **Liveness Probe:** Is the container alive? (If no, restart it).
*   **Readiness Probe:** Can it receive traffic? (If no, remove from Service endpoints).
*   **Startup Probe:** Has the app finished starting? (If no, wait; blocks other probes).

### 3. Difference between Liveness and Readiness?
**Answer:**

| Probe | Purpose | Failure Impact |
| :--- | :--- | :--- |
| **Liveness** | Detect stuck apps (deadlocks). | Pod is **restarted**. |
| **Readiness** | Control traffic flow. | Pod is **removed from Service endpoints** (no traffic). |
| **Startup** | Handle slow-starting apps. | Disables liveness checks until success. |

**⚠️ Interview Trap:** Readiness failure does **NOT** restart the pod.

### 4. When should you use a Startup Probe?
**Answer:**
Use it for **slow-starting applications** (e.g., legacy Java apps, large data loaders).
*   **Why:** It prevents Kubernetes from killing the container via the Liveness probe before it has had a chance to fully initialize.

### 5. What happens if a liveness probe keeps failing?
**Answer:**
1.  The container is **restarted** by the kubelet.
2.  The restart count increases.
3.  If it continues, the Pod enters a **CrashLoopBackOff** state with increasing backoff delays.

**Advanced Tip:** Badly configured liveness probes are a common cause of self-inflicted outages.

### 6. Can probes cause cascading failures?
**Answer:**
**Yes.**
*   **How:** If a liveness probe checks an external dependency (like a Database) and the DB goes down, *all* pods might fail the probe simultaneously.
*   **Result:** Kubernetes restarts all pods at once, causing a "thundering herd" when they come back up, potentially crashing the DB again.
*   **Best Practice:** **Never** check databases or downstream services in liveness probes. Only check local app health.

### 7. What is a CRD (Custom Resource Definition)?
**Answer:**
CRDs extend the Kubernetes API by allowing you to define custom resource types that look and act like native Kubernetes objects.

**Example:**
```yaml
kind: Backup
apiVersion: storage.example.com/v1
```
Now Kubernetes understands `kubectl get backups`.

### 8. CRD vs ConfigMap?
**Answer:**

| Feature | CRD | ConfigMap |
| :--- | :--- | :--- |
| **Structure** | Structured schema (OpenAPI). | Key-value pairs. |
| **Versioning** | Supports API versioning. | No versioning. |
| **Behavior** | Works with Controllers/Operators. | Passive data storage. |

**Key Insight:** CRDs model **behavior** and custom logic; ConfigMaps store **configuration** data.

### 9. What is OpenAPI schema in CRD?
**Answer:**
It defines the structure and validation rules for the Custom Resource.
*   **Includes:** Field types (string, int), required fields, defaults, and validation patterns.
*   **Importance:**
    *   Prevents bad YAML from being applied.
    *   Enables `kubectl` validation.
    *   Required for Server-Side Apply.

### 10. What are CRD versions (v1alpha1, v1beta1, v1)?
**Answer:**
They represent the **stability** of the API, not the Kubernetes version.
*   **v1alpha1:** Experimental, breaking changes allowed, may be dropped.
*   **v1beta1:** Mostly stable, well-tested, semantics preserved.
*   **v1:** Production ready, long-term compatibility guarantees.

### 11. What is a Kubernetes Operator?
**Answer:**
An Operator is a custom controller that uses CRDs to encode human operational knowledge into software.
**Formula:** `Operator = CRD + Controller + Reconciliation Logic`.

### 12. What problem do Operators solve?
**Answer:**
They automate complex, stateful application lifecycle management that standard Kubernetes primitives cannot handle.
*   **Examples:** Database backups, failover, schema upgrades, restoring from snapshots.
*   **Popular Operators:** Prometheus Operator, Postgres Operator, ECK (Elastic Cloud on K8s).

### 13. Explain the reconciliation loop in an Operator.
**Answer:**
1.  **Watch:** The Operator watches for changes in the CRD (Desired State).
2.  **Compare:** It checks the actual state of the system.
3.  **Reconcile:** It takes actions (create pods, update config) to make the Actual State match the Desired State.
4.  **Loop:** This runs continuously.

**Key Concept:** Operators are **event-driven** but **state-reconciled**.

### 14. Operator vs Helm?
**Answer:**

| Feature | Helm | Operator |
| :--- | :--- | :--- |
| **Role** | Package Manager / Template Engine. | Control Loop / Automation. |
| **Scope** | Day 1 (Installation). | Day 2 (Lifecycle Management). |
| **Logic** | No runtime logic. | Full runtime logic (backup, heal, scale). |

**Golden Line:** Helm installs; Operators operate.

### 15. Can Operators replace SREs?
**Answer:**
**No.** Operators automate *known* workflows and standard procedures. SREs are needed to handle *unknown* failures, architecture design, and complex debugging.

### 16. Can CRDs be namespaced or cluster-scoped?
**Answer:**
**Yes.** This is defined in the CRD definition under `spec.scope`.
*   `Namespaced`: Resources exist within a namespace (default).
*   `Cluster`: Resources exist globally (like Nodes or StorageClasses).

### 17. What happens if an Operator crashes?
**Answer:**
1.  Kubernetes restarts the Operator pod (it's just a pod).
2.  The state of the managed application is **preserved** (stored in etcd via CRs).
3.  Once restarted, the reconciliation loop resumes and checks the state again.
*   **Why safe?** Operators are designed to be **idempotent**.

### 18. How do you secure CRDs and Operators?
**Answer:**
*   **RBAC:** Limit what the Operator can do (e.g., only access specific namespaces).
*   **Admission Webhooks:** Validate CRD data before it's persisted.
*   **Namespace Isolation:** Run operators in dedicated namespaces.
*   **Security Context:** Run operator pods as non-root.

### 19. Real-world failure scenario: Liveness Probe causing outages.
**Answer:**
**Scenario:** A Liveness probe was configured to check database connectivity.
1.  **Event:** The Database had a minor hiccup (slow response).
2.  **Impact:** The Liveness probe failed for *all* application pods.
3.  **Result:** Kubernetes restarted all pods simultaneously.
4.  **Cascading Failure:** The restarting pods overwhelmed the DB with new connections ("Thundering Herd"), causing the DB to crash again.
**Fix:** Move dependency checks (like DB connection) to the **Readiness Probe**, or use a dedicated health endpoint that only checks the app's internal state.
