# Deployment Strategies Interview Questions

This document contains common interview questions related to Deployment Strategies (Rolling, Blue-Green, Canary, etc.) and their answers.

### 1. What is a Rolling Deployment?
**Answer:**
A rolling deployment gradually replaces old pods with new ones, ensuring zero or minimal downtime.

**Kubernetes Implementation:**
*   Default behavior of `Deployment`.
*   Controlled using:
    ```yaml
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1        # How many extra pods can be created
        maxUnavailable: 0  # How many pods can be unavailable during update
    ```

**When to use:**
*   Stateless applications.
*   Backward-compatible changes.

**Limitation:**
*   No easy rollback to exact previous version state (traffic is mixed during rollout).

### 2. What is a Recreate Deployment?
**Answer:**
All old pods are terminated **before** new pods start, causing downtime.

**Kubernetes Implementation:**
```yaml
strategy:
  type: Recreate
```

**Use case:**
*   Applications that cannot run multiple versions simultaneously (e.g., legacy apps).
*   DB schema breaking changes where old code breaks with new schema.

### 3. What is Blue-Green Deployment?
**Answer:**
Two identical environments (Blue = live, Green = new). Traffic is switched instantly from Blue to Green after validation.

**Kubernetes Implementation Approach:**
1.  Deploy two versions: `app-blue` (v1) and `app-green` (v2).
2.  Single Service points to only one version via selector.
3.  Switch traffic by updating Service selector:
    ```yaml
    selector:
      version: green
    ```

**Rollback:**
*   Instant → switch selector back to `blue`.

**Tools:**
*   Native Kubernetes Service.
*   Argo Rollouts.
*   Helm.

**Pros/Cons:**
*   ✅ Zero downtime, Instant rollback.
*   ❌ Double infrastructure cost.

### 4. What is Canary Deployment?
**Answer:**
Gradually expose a new version to a small subset of users before full rollout.

**Kubernetes Implementation (Basic):**
*   Two deployments: Stable (90% replicas) and Canary (10% replicas).
*   Same Service selects both.
    ```yaml
    replicas:
      stable: 9
      canary: 1
    ```

**Advanced Canary (Production-grade):**
*   Using **Argo Rollouts / Istio / NGINX Ingress**.
*   Traffic split by percentage (not just replica count).
*   Metrics-based promotion and automated rollback.
    ```yaml
    strategy:
      canary:
        steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - setWeight: 50
    ```

**Use case:**
*   High-risk changes.
*   User-facing services where you want to limit blast radius.

### 5. Difference Between Canary and Blue-Green?
**Answer:**

| Feature | Canary | Blue-Green |
| :--- | :--- | :--- |
| **Traffic** | Gradual shift (10% → 50% → 100%). | Instant switch (0% → 100%). |
| **Risk** | Low (only affects small subset). | Medium (affects everyone at once). |
| **Cost** | Low (only need small extra capacity). | High (requires double capacity). |
| **Rollback** | Gradual/Instant. | Immediate. |

### 6. How do you implement Canary using Ingress?
**Answer:**
Ingress controllers like NGINX support traffic splitting using annotations.

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
```

**Benefit:**
*   No service-level changes required.
*   Traffic controlled at the ingress layer.

### 7. What is Shadow (Dark) Deployment?
**Answer:**
The new version receives real production traffic, but responses are **not** returned to users. It's used to test how the new version handles load without impacting users.

**Kubernetes Implementation:**
*   Duplicate traffic using **Service Mesh (Istio mirroring)** or **API Gateway (Envoy)**.
    ```yaml
    mirror:
      host: app-shadow
    ```

**Use case:**
*   Performance testing.
*   Observability validation.

### 8. What is A/B Deployment?
**Answer:**
Different versions serve traffic based on specific user attributes (headers, cookies, geo-location), rather than just random percentage.

**Kubernetes Implementation:**
*   Requires **Istio** or **NGINX Ingress**.
*   Route by headers:
    ```yaml
    match:
      - headers:
          user-type:
            exact: beta
    ```

**Use case:**
*   Feature experimentation.
*   UX testing for specific user groups.

### 9. How do readiness and liveness probes affect deployments?
**Answer:**
*   **Readiness probe:** Controls traffic routing during rollout. If a new pod isn't ready, K8s won't send traffic to it and won't kill the old pod (pausing the rollout).
*   **Liveness probe:** Restarts unhealthy pods.

**Key Interview Point:**
Without readiness probes, Kubernetes may route traffic to unhealthy pods during rollout, causing downtime even in a Rolling Update.

### 10. How do you monitor and decide promotion/rollback?
**Answer:**
Use metrics to validate the health of the new version:
*   **Metrics:** Error rate (HTTP 5xx), Latency, CPU/memory usage, Business KPIs.
*   **Tools:** Prometheus + Grafana, Argo Rollouts AnalysisTemplates, Datadog / New Relic.

### 11. Why not always use Rolling Deployment?
**Answer:**
Rolling deployments cannot safely handle:
*   **Breaking changes:** If v2 API is incompatible with v1, clients might fail when hitting v2 pods while v1 is still running.
*   **Stateful services:** Database schema changes might not support mixed versions.
*   **API incompatibility:** Hence Canary or Blue-Green is preferred for safer transitions.

### 12. Which strategy do you recommend in production?
**Answer:**
**Canary with automated analysis** (using tools like Argo Rollouts and Prometheus).
*   **Reason:** It balances risk, cost, and safety. It limits the blast radius of bugs while not requiring double the infrastructure cost of Blue-Green.
