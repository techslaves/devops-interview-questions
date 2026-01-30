# Istio Interview Questions and Answers

This document contains common interview questions related to Istio Service Mesh and their answers.

### 1. What is Istio?
**Answer:**
Istio is a service mesh that provides traffic management, security, and observability for microservices without changing application code, using sidecar proxies.
*   **👉 Key value:** Controls service-to-service communication.

### 2. What problem does Istio solve?
**Answer:**
Istio solves:
*   Unreliable service communication.
*   Manual TLS & authentication between services.
*   Lack of visibility (metrics, traces).
*   Complex traffic control (canary, retries, circuit breaking).

### 3. What is a Service Mesh?
**Answer:**
A service mesh is an infrastructure layer that manages east-west traffic between services using proxies, handling retries, mTLS, telemetry, and routing.

### 4. What are the main components of Istio?
**Answer:**
*   **Envoy Proxy:** Data plane (sidecar).
*   **Istiod:** Control plane (config, certs, service discovery).
    *   *(Note: Older versions had Pilot, Citadel, Galley — now merged into Istiod).*

### 5. What is Envoy and why is it used?
**Answer:**
Envoy is a high-performance L7 proxy used as a sidecar to:
*   Intercept traffic.
*   Apply routing rules.
*   Enforce mTLS.
*   Collect metrics and traces.

### 6. What is Sidecar Injection?
**Answer:**
Sidecar injection adds an Envoy proxy container to each pod so Istio can control traffic.
**Types:**
*   **Automatic injection:** Via namespace label (`istio-injection=enabled`).
*   **Manual injection:** Using `istioctl kube-inject`.

### 7. What is Istio Control Plane?
**Answer:**
The control plane (**Istiod**) is responsible for:
*   Config distribution to Envoy.
*   Certificate management (mTLS).
*   Service discovery integration with Kubernetes.

### 8. How does traffic flow in Istio?
**Answer:**
1.  Request hits Envoy sidecar.
2.  Envoy applies routing/security rules.
3.  Traffic forwarded to destination Envoy.
4.  Destination Envoy passes traffic to app.
*   **👉 Key:** Application never talks directly to other services.

### 9. What is mTLS in Istio?
**Answer:**
**Mutual TLS** ensures:
*   Encryption in transit.
*   Service identity verification.
*   Zero trust networking.

**Istio automatically:**
*   Issues certificates.
*   Rotates certs.
*   Enforces policies.

### 10. Difference between PERMISSIVE and STRICT mTLS?
**Answer:**
*   **PERMISSIVE:** Allows both plaintext & mTLS (used during migration).
*   **STRICT:** Only mTLS traffic allowed (secure mode).

### 11. What is a VirtualService?
**Answer:**
`VirtualService` defines **how** traffic is routed to a service.
**Used for:**
*   Canary releases.
*   Header-based routing.
*   Traffic splitting.
*   *Example:* Send 90% traffic to v1, 10% to v2.

### 12. What is a DestinationRule?
**Answer:**
`DestinationRule` defines policies applied **after** routing.
**Includes:**
*   Load balancing.
*   Connection pooling.
*   Circuit breaking.
*   Subsets (v1, v2).

### 13. Difference between VirtualService and DestinationRule?
**Answer:**
*   **VirtualService:** Where traffic goes (Routing).
*   **DestinationRule:** How traffic behaves once it gets there (Policies).

### 14. What is Gateway in Istio?
**Answer:**
`Gateway` controls ingress or egress traffic into the mesh.
It defines:
*   Ports.
*   Protocols.
*   TLS settings.
*   *👉 Think of it as L7 load balancer configuration.*

### 15. Istio Ingress Gateway vs Kubernetes Ingress?
**Answer:**
*   **K8s Ingress:** Basic L7 routing.
*   **Istio Gateway:** Advanced routing, mTLS, retries, observability, and traffic splitting.

### 16. What is an Istio AuthorizationPolicy?
**Answer:**
Defines who can access what, based on:
*   Service identity.
*   Namespace.
*   HTTP methods.
*   Paths.
*   JWT claims.
*   *👉 Used for zero-trust access control.*

### 17. How does Istio handle authentication?
**Answer:**
*   **Peer authentication:** Service identity (mTLS).
*   **Request authentication:** End-user auth (JWT).

### 18. What observability features does Istio provide?
**Answer:**
*   **Metrics:** Prometheus.
*   **Tracing:** Jaeger.
*   **Visualization:** Kiali.
*   **Logging:** Envoy access logs.

### 19. What is Kiali?
**Answer:**
Kiali provides:
*   Service topology view.
*   Traffic flow visualization.
*   Health checks.
*   Config validation.

### 20. What is Circuit Breaking in Istio?
**Answer:**
Prevents cascading failures by:
*   Limiting connections.
*   Detecting unhealthy services.
*   Failing fast instead of retry storms.
*   *Configured using `DestinationRule`.*

### 21. How does Istio support Canary Deployments?
**Answer:**
Using `VirtualService` + `DestinationRule`:
*   Split traffic by percentage.
*   Gradually shift traffic.
*   Roll back instantly.

### 22. What happens if Envoy sidecar crashes?
**Answer:**
*   Pod restarts (if sidecar crashes).
*   Traffic may fail temporarily.
*   Application depends on Envoy for communication.
*   *👉 Sidecar is critical path.*

### 23. How does Istio affect performance?
**Answer:**
*   Adds latency (typically 1–5 ms).
*   Increases CPU/memory usage (for sidecars).
*   Trade-off for security & control.

### 24. Common Istio Production Issues?
**Answer:**
*   Misconfigured mTLS (enabling STRICT too early).
*   Incorrect `VirtualService` routing.
*   Envoy resource exhaustion.
*   Debugging complexity.

### 25. When should you NOT use Istio?
**Answer:**
*   Small/simple apps.
*   Low traffic systems.
*   Teams without strong Kubernetes maturity.

### 26. How do you troubleshoot Istio issues?
**Answer:**
*   `istioctl analyze`.
*   Envoy logs.
*   Kiali graph.
*   Check mTLS mode.
*   Validate `VirtualService`/`DestinationRule`.

### 27. Istio vs Linkerd?
**Answer:**
*   **Istio:** Feature-rich, complex, uses Envoy.
*   **Linkerd:** Simpler, lightweight, opinionated, uses Rust proxy.

### 28. How does Istio integrate with Kubernetes?
**Answer:**
*   Uses Kubernetes service discovery.
*   Injects sidecars via mutating webhook.
*   Applies CRDs for traffic policies.

### 29. What are Istio CRDs?
**Answer:**
Custom resources like:
*   `VirtualService`
*   `DestinationRule`
*   `Gateway`
*   `AuthorizationPolicy`
*   `PeerAuthentication`

### 30. Real-world use case of Istio?
**Answer:**
Secure service-to-service communication with mTLS, enforce zero-trust access, and safely roll out new versions using traffic splitting (Canary).
