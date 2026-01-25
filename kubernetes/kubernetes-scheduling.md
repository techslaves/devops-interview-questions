# Kubernetes Scheduling Interview Questions

This document contains common interview questions related to Kubernetes Scheduling and their answers.

### 1. How does Kubernetes decide which node a Pod runs on?
**Answer:**
The `kube-scheduler` is responsible for assigning a node to a Pod. It follows a 3-step process:
1.  **Filtering:** Removes nodes that don’t meet the Pod's requirements (CPU/Memory resources, Taints, nodeSelector, Affinity).
2.  **Scoring:** Ranks the remaining eligible nodes based on priorities (e.g., least requested resources, image locality, affinity preferences).
3.  **Binding:** Assigns the Pod to the node with the highest score.

### 2. What happens if no node is available for a Pod?
**Answer:**
*   The Pod remains in the **Pending** state.
*   The scheduler continuously retries to schedule it.
*   **Common Reasons:**
    *   Insufficient resources (CPU/Memory) on nodes.
    *   Taints on nodes without matching tolerations.
    *   Strict affinity rules that cannot be met.
*   **Debugging:** Check events using `kubectl describe pod <pod-name>`.
    *   *Example Event:* `0/5 nodes are available: 3 Insufficient cpu, 2 node(s) had taint {node-role.kubernetes.io/master: }.`

### 3. What is the difference between `nodeSelector` and `nodeAffinity`?
**Answer:**

| Feature | nodeSelector | nodeAffinity |
| :--- | :--- | :--- |
| **Flexibility** | Very basic (key-value pair). | Advanced (set-based logic). |
| **Operators** | Exact match only. | `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`. |
| **Soft rules** | ❌ No (Hard constraint only). | ✅ Yes (`preferredDuringScheduling`). |
| **Recommended** | ❌ Legacy (simple use cases). | ✅ Yes (Standard). |

### 4. What are taints and tolerations?
**Answer:**
*   **Taints:** Applied to **Nodes** to repel Pods.
*   **Tolerations:** Applied to **Pods** to allow them to schedule on tainted nodes.

**Example:**
```shell
kubectl taint nodes node1 dedicated=infra:NoSchedule
```
Only Pods with a matching toleration in their spec can be scheduled on `node1`.

### 5. Explain the difference between `NoSchedule`, `PreferNoSchedule`, and `NoExecute`.
**Answer:**
These are **Taint Effects**:
*   **NoSchedule:** New Pods will **not** be scheduled on the node unless they tolerate the taint. Existing Pods are unaffected.
*   **PreferNoSchedule:** The scheduler will **try** to avoid placing the Pod on the node, but will do so if no other nodes are available (Soft constraint).
*   **NoExecute:** New Pods won't schedule, and **existing Pods** on the node without the toleration will be **evicted** immediately.

### 6. What is Pod Affinity and Anti-Affinity?
**Answer:**
*   **Pod Affinity:** Rules to place Pods **close** to each other (e.g., on the same node or availability zone).
    *   *Use Case:* Low latency communication between a web app and its cache.
*   **Pod Anti-Affinity:** Rules to spread Pods **apart** (e.g., different nodes or zones).
    *   *Use Case:* High Availability (HA). Ensuring replicas of the same application don't run on the same node (to avoid single point of failure).

### 7. What is `topologyKey` in affinity rules?
**Answer:**
It defines the **domain** or scope for the scheduling decision (i.e., "co-located" relative to what?).
*   `kubernetes.io/hostname`: Pods must be on the **same node**.
*   `topology.kubernetes.io/zone`: Pods must be in the **same Availability Zone**.

### 8. Difference between `requiredDuringScheduling` and `preferredDuringScheduling`?
**Answer:**
*   **requiredDuringSchedulingIgnoredDuringExecution:** **Hard constraint**. The Pod **must** meet the rule. If no node matches, the Pod stays **Pending**.
*   **preferredDuringSchedulingIgnoredDuringExecution:** **Soft constraint**. The scheduler will **try** to find a matching node, but if not, it will schedule the Pod anywhere.

### 9. How do resource requests vs limits affect scheduling?
**Answer:**
*   **Requests:** Used by the **Scheduler**. A node must have enough free capacity to satisfy the Pod's request to be selected.
*   **Limits:** Enforced by the **Container Runtime (kubelet)**. They are ignored during scheduling. If a Pod exceeds its limit, it may be throttled (CPU) or OOMKilled (Memory).

### 10. Explain Pod Priority and Preemption.
**Answer:**
*   **Pod Priority:** Indicates the importance of a Pod relative to others (defined via `PriorityClass`).
*   **Preemption:** If a high-priority Pod cannot be scheduled due to lack of resources, the scheduler attempts to **evict (preempt)** lower-priority Pods to free up space.
*   **Use Case:** Ensuring critical system components or monitoring agents always run, even if the cluster is full.

### 11. What are scheduling profiles and plugins?
**Answer:**
The Kubernetes Scheduler is highly customizable via the **Scheduling Framework**.
*   It runs a series of **Plugins** at different extension points (phases):
    1.  **QueueSort:** Sorts pods in the scheduling queue.
    2.  **Filter:** Filters out nodes (like predicates).
    3.  **Score:** Ranks nodes (like priorities).
    4.  **Bind:** Binds Pod to Node.
*   You can define multiple **Profiles** to enable/disable specific plugins for different workloads.

### 12. How does Kubernetes ensure even Pod distribution?
**Answer:**
*   **Pod Topology Spread Constraints (`topologySpreadConstraints`):** Allows you to control how Pods are spread across your cluster among failure-domains such as regions, zones, nodes, and other user-defined topology domains.
*   **Pod Anti-Affinity:** Can be used to force spreading.
*   **Default Scheduler Scoring:** The `SelectorSpreadPriority` function tries to minimize the number of pods belonging to the same service on the same node.

### 13. How does scheduling work in EKS / cloud environments?
**Answer:**
The scheduler is cloud-agnostic, but it respects cloud-specific constraints via labels and admission controllers:
*   **Node Labels:** Nodes are labeled with Region and Zone (`topology.kubernetes.io/zone`). Affinity rules use these.
*   **Volume Topology:** If a Pod requests an EBS volume that exists in `us-east-1a`, the scheduler filters out nodes in other zones because EBS volumes are zonal.
*   **Cluster Autoscaler:** If the scheduler marks a Pod as `Pending` (unschedulable), the Cluster Autoscaler detects this and spins up a new EC2 node.

### 14. What are `topologySpreadConstraints` and why are they better than anti-affinity?
**Answer:**
They control the even distribution of Pods across zones, nodes, or other topology domains without the hard failures of anti-affinity.

**Benefits:**
*   **Flexibility:** Allows for "skew" (e.g., max difference of 1 pod between zones) rather than strict "one per zone".
*   **Autoscaling:** Works better with Cluster Autoscaler as it doesn't over-constrain the scheduler.
*   **Production Ready:** Preferred for High Availability (HA) workloads to ensure balanced spreading.

### 15. Scenario: You have a set of nodes equipped with expensive GPUs. How do you ensure that only machine learning workloads run on these nodes, and that general web-app pods don't "accidentally" get scheduled there?
**Answer:**
You apply a **Taint** to the GPU nodes and a **Toleration** to the ML pods.

**Step 1 (Taint):** `kubectl taint nodes gpu-node-1 hardware=gpu:NoSchedule`. This "repels" any pod that doesn't explicitly say it can handle this taint.
```shell
kubectl taint nodes gpu-node-1 hardware=gpu:NoSchedule
```

**Step 2 (Toleration):** In the ML Pod spec, add a toleration for the key `hardware=gpu`.

**Follow-up:** Does a toleration guarantee the pod will land on the GPU node?
**No.** A toleration only *allows* it. To *force* it there, you must also use **Node Affinity**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ml-training-pod
spec:
  containers:
  - name: cuda-container
    image: nvidia/cuda:11.0-base
  # This allows the pod to be scheduled on the tainted node
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  # Optional: Use nodeAffinity to FORCE it to land there
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values:
            - gpu
```

### 16. Scenario: You are deploying 3 replicas of a critical microservice. How do you ensure that Kubernetes doesn't put all 3 replicas on the same worker node (which would create a single point of failure)?
**Answer:**
Use **Pod Anti-Affinity** with a `topologyKey`.

**The Strategy:** Set a `requiredDuringSchedulingIgnoredDuringExecution` rule. Use the label of the pod itself (e.g., `app: my-service`) as the selector and set `topologyKey: "kubernetes.io/hostname"`.

**Result:** The scheduler will look at the hostname of nodes; if a node already has a pod with `app: my-service`, it will reject the second replica for that node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-server
            # This checks the 'hostname' label on the node
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx:latest
```

### 17. Scenario: You have a Web-App and a Redis Cache. For performance reasons, you want the Web-App pod to always run on the same node as the Redis pod to reduce network latency. How do you achieve this?
**Answer:**
Use **Pod Affinity**.

**The Strategy:** In the Web-App deployment, define a `podAffinity` rule that searches for pods with the label `app: redis` using `topologyKey: "kubernetes.io/hostname"`.

**Result:** The scheduler will prioritize (or require) placing the Web-App on whatever node is currently hosting the Redis pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - redis-cache
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: web-app
    image: my-web-app:v1
```

### 18. Scenario: Your cluster is at 95% capacity. A critical "Payment Processing" pod needs to be deployed immediately. How do you ensure it gets scheduled even if the cluster is full?
**Answer:**
Implement **Pod Priority and Preemption**.

**The Mechanism:**
1.  Create a `PriorityClass` object with a high integer value (e.g., 1,000,000).
2.  Assign `priorityClassName: high-priority` to the Payment pod.
3.  **The Result:** The scheduler will identify lower-priority pods (like "Image Resizers" or "Dev-test" pods), evict them (preemption), and give their resources to the Payment pod.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-payments
value: 1000000
globalDefault: false
description: "Used for payment processing pods that must run."
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-processor
spec:
  containers:
  - name: processor
    image: payment-gateway:v2
  priorityClassName: critical-payments
```
