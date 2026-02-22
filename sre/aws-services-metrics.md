# AWS Services Metrics Interview Questions

This document contains common interview questions related to monitoring and observability metrics for various AWS services.

### What metrics do you track for EKS and why?
**Answer:**
I monitor EKS across 4 dimensions:

**A. Control Plane (Availability Risk)**
*   **Source:** Amazon CloudWatch.
*   **Metrics:**
    *   API server latency.
    *   API server 4xx/5xx errors.
    *   API throttling.
    *   Cluster failed requests.
*   **Why?** If the control plane degrades, scaling, scheduling, and rollouts fail.

**B. Node Layer (Capacity & HA)**
*   **Metrics:**
    *   Node Ready state.
    *   CPU/Memory utilization.
    *   Disk pressure.
    *   Network packets dropped.
    *   EC2 status checks.
*   **Risk Detected:** AZ imbalance, Spot interruptions, Resource exhaustion.

**C. Pod Layer (Workload Health)**
*   **Metrics:**
    *   Pod restarts.
    *   OOMKilled count.
    *   CrashLoopBackOff.
    *   CPU throttling.
    *   Pod readiness failures.
    *   Desired vs available replicas.

**D. Service Layer (User Experience)**
*   **Golden Signals:** Latency (P95/P99), Error rate, Throughput, Saturation.
*   **Architect Insight:** EKS availability is meaningless if service latency is high.

### What metrics do you track for MSK?
**Answer:**

**Broker Health**
*   Active controller count.
*   Under-replicated partitions (Risk of data loss).
*   Offline partitions.
*   Leader imbalance.
*   ISR (In-Sync Replicas) shrink/expand rate.

**Performance**
*   Bytes in/out per broker.
*   Request latency.
*   **Consumer lag** (Critical for real-time systems).
*   Produce request failures.
*   Network throughput.

**Capacity**
*   Disk usage %.
*   CPU utilization.
*   JVM heap usage.
*   GC (Garbage Collection) time.
*   **Note:** Kafka failures often happen due to Disk full, High GC pauses, or Network saturation.

### What metrics do you track for EC2?
**Answer:**

**Infrastructure Health**
*   Status check failed (instance/system).
*   CPU utilization.
*   CPU credits (for T-series instances).
*   Network in/out.
*   Disk read/write ops.
*   EBS burst balance.

**Architect Thinking:**
*   **CPU low but latency high?** → Possibly an I/O bottleneck.

### What do you monitor for CloudFront?
**Answer:**

**Availability & Performance**
*   4xx error rate.
*   5xx error rate.
*   Origin latency.
*   **Cache hit ratio** (Low ratio = higher origin cost + latency).
*   Total requests.
*   Bytes downloaded.

### What are the critical metrics for API Gateway?
**Answer:**

**Critical Metrics**
*   Latency (P95/P99).
*   4xx errors (Client issue).
*   5xx errors (Backend issue).
*   **Integration latency** (High = downstream service slow).
*   Throttled requests.
*   Request count.

### What metrics matter for Pods?
**Answer:**
*   Container CPU usage.
*   **CPU throttling** (Often indicates wrong limits configuration).
*   Memory usage.
*   **OOMKilled** count.
*   Restart count.
*   Readiness probe failures.
*   HPA scaling events.

### What metrics do you track for RDS?
**Answer:**

**Availability**
*   DB connections.
*   **Replica lag**.
*   Failover events.
*   Deadlocks.

**Performance**
*   CPU utilization.
*   Freeable memory.
*   **Disk queue depth**.
*   Read/write latency.
*   IOPS.
*   Buffer cache hit ratio.

**Architect Thinking:**
*   **High CPU + low connections** → Expensive queries.
*   **High connections + low CPU** → Connection storm.

### What metrics matter for Redshift?
**Answer:**

**Performance**
*   Query duration.
*   **WLM (Workload Management) queue length** (If high → need workload separation).
*   Concurrency scaling usage.
*   Disk usage.
*   CPU utilization.
*   Commit queue length.

### How do you design centralized observability for all AWS services?
**Answer:**
I design observability in layers:

**1. Metrics Aggregation**
*   **Sources:** CloudWatch native metrics, Prometheus for Kubernetes, Custom metrics via OpenTelemetry.
*   **Tools:** Prometheus, Grafana.

**2. Logs**
*   Structured JSON logs.
*   Centralized log store (e.g., OpenSearch, CloudWatch Logs).
*   Log correlation using request IDs.

**3. Traces**
*   **Tools:** OpenTelemetry, AWS X-Ray.
*   **Flow:** Trace across CloudFront → API Gateway → EKS → RDS → MSK.

**4. Alerting Strategy**
*   **SLO-based alerts** (Service Level Objectives).
*   Multi-layer alerts (Infrastructure + Application).
*   Avoid noisy threshold alerts.
*   Implement auto-remediation where possible.
