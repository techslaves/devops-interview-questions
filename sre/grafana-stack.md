# Grafana Stack Interview Questions and Answers

This document contains common interview questions related to the Grafana Stack (Prometheus, Grafana, Alertmanager) and Kubernetes monitoring.

## 1️⃣ Prometheus

### 1. What is Prometheus?
**Answer:**
Prometheus is an open-source monitoring system that scrapes metrics over HTTP and stores them as time-series data.

### 2. Push vs Pull model in Prometheus?
**Answer:**
Prometheus primarily uses a **pull model**, scraping `/metrics` endpoints at configured intervals.

### 3. What is a metric in Prometheus?
**Answer:**
A metric is a time-series identified by a name and a set of key-value labels.

### 4. Types of metrics in Prometheus?
**Answer:**
*   **Counter:** Only increases (e.g., `requests_total`).
*   **Gauge:** Can go up/down (e.g., `memory_usage`).
*   **Histogram:** Buckets + count + sum (e.g., latency).
*   **Summary:** Quantiles calculated client-side.

**Interview tip:** Use Histogram for SLOs and latency distributions.

### 5. What is PromQL?
**Answer:**
PromQL is the Prometheus Query Language used to query and aggregate time-series data.

### 6. Example – CPU usage per pod?
**Answer:**
```promql
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
```

### 7. Difference between `rate()` and `irate()`?
**Answer:**
*   **rate():** Averaged over a time window (smoother).
*   **irate():** Instant spike detection (more volatile).

### 8. Prometheus architecture?
**Answer:**
Targets expose metrics → Prometheus scrapes → Stores in TSDB → Queried by Grafana → Alerts via Alertmanager.

### 9. What is Alertmanager?
**Answer:**
Handles alerts sent by Prometheus: grouping, deduplication, silencing, and routing to receivers like Slack, Email, or PagerDuty.

### 10. How does Prometheus scale?
**Answer:**
Prometheus scales vertically. For large setups, use **federation** or remote storage solutions like **Thanos** or **Cortex**.

### 11. What is federation?
**Answer:**
One Prometheus server scrapes metrics from another Prometheus server.

---

## 2️⃣ Grafana

### 12. What is Grafana?
**Answer:**
Grafana is a visualization and dashboarding tool for metrics, logs, and traces.

### 13. Is Grafana a monitoring tool?
**Answer:**
**No**, Grafana visualizes data; it does not collect metrics itself.

### 14. What are Grafana dashboards?
**Answer:**
Dashboards are collections of panels visualizing metrics using queries.

### 15. What is a panel?
**Answer:**
A panel is a single visualization component like a graph, gauge, or table.

### 16. What data sources does Grafana support?
**Answer:**
Prometheus, Loki, Elasticsearch, InfluxDB, CloudWatch, Azure Monitor, etc.

### 17. Grafana + Prometheus integration?
**Answer:**
Grafana queries Prometheus using PromQL to visualize the data.

### 18. Does Grafana support alerting?
**Answer:**
**Yes**, Grafana supports alerting on panel queries.

### 19. Grafana alerts vs Prometheus alerts?
**Answer:**
*   **Prometheus alerts:** Metric-native, more reliable, defined in code.
*   **Grafana alerts:** Visualization-driven, easier to set up via UI.

### 20. What are Grafana variables?
**Answer:**
Variables allow dynamic dashboards (e.g., selecting a namespace, pod, or cluster from a dropdown).

### 21. How do you secure Grafana?
**Answer:**
RBAC, OAuth/LDAP integration, read-only dashboards, and folder permissions.

---

## 3️⃣ kube-state-metrics (Very Important 🔥)

### 22. What is kube-state-metrics?
**Answer:**
It exposes the state of Kubernetes objects (Deployments, Pods, Nodes) as metrics.

### 23. Does kube-state-metrics collect resource usage?
**Answer:**
❌ **No** — it only exposes cluster state (e.g., "How many replicas should exist?").

### 24. Examples of kube-state-metrics data?
**Answer:**
*   Pod status (Running, Pending).
*   Deployment replicas (Desired vs Available).
*   Node conditions.
*   Resource requests & limits.
*   *Example metric:* `kube_pod_status_phase`.

### 25. Difference between kube-state-metrics and metrics-server?
**Answer:**
*   **kube-state-metrics:** Desired & current state (metadata).
*   **metrics-server:** Actual resource usage (CPU/Memory stats).

### 26. Why is kube-state-metrics important?
**Answer:**
Prometheus cannot infer Kubernetes state (like "Pending" pods) by itself; it needs this exporter.

### 27. Where is kube-state-metrics deployed?
**Answer:**
Typically as a Deployment in the `kube-system` namespace.

---

## 4️⃣ Metrics Server

### 28. What is metrics-server?
**Answer:**
It collects CPU & memory usage from the Kubelet.

### 29. How does it collect metrics?
**Answer:**
It scrapes the Kubelet `/stats/summary` API.

### 30. What uses metrics-server?
**Answer:**
*   `kubectl top nodes`
*   `kubectl top pods`
*   **HPA (Horizontal Pod Autoscaler)**

### 31. Does Prometheus use metrics-server?
**Answer:**
❌ **No** — Prometheus scrapes exporters directly.

### 32. Limitations of metrics-server?
**Answer:**
*   Short retention (in-memory).
*   No historical data.
*   No custom metrics.

---

## 5️⃣ Comparison Summary

| Component | Purpose |
| :--- | :--- |
| **Prometheus** | Collects & stores metrics. |
| **Grafana** | Visualizes metrics. |
| **kube-state-metrics** | Exposes K8s object state. |
| **metrics-server** | Provides live CPU/memory usage. |

---

## 6️⃣ Common Scenario Questions

### 33. HPA not scaling – where do you check?
**Answer:**
Check `metrics-server` health, HPA events (`kubectl describe hpa`), Kubelet metrics, and ensure resource requests are defined.

### 34. Pod is Pending but CPU looks fine – how to debug?
**Answer:**
Check `kube-state-metrics` for pod status and scheduler events (taints, affinity, PVC binding).

---

## 7️⃣ Ultra-Crisp Memory Lines

*   **Prometheus** → Scrape + store metrics.
*   **Grafana** → Visualize.
*   **kube-state-metrics** → Cluster state.
*   **metrics-server** → Live resource usage.
