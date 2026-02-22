# SRE Interview Questions and Answers

This document contains common interview questions related to Site Reliability Engineering (SRE) and their answers.

### 1. What is SRE?
**Answer:**
Applying software engineering principles to operations to improve reliability and scalability.

### 2. How is SRE different from DevOps?
**Answer:**
DevOps is a culture/practice; SRE is an implementation of DevOps using engineering and SLIs/SLOs.

### 3. What is reliability?
**Answer:**
Probability that a system performs correctly over a given period.

### 4. What is an incident?
**Answer:**
An unplanned interruption or degradation of service.

### 5. What is on-call?
**Answer:**
Rotation where engineers respond to incidents outside normal hours.

### 6. What is monitoring?
**Answer:**
Collecting and alerting on system metrics and logs.

### 7. Difference between monitoring and observability?
**Answer:**
Monitoring tells **what** is wrong; observability explains **why**.

### 8. What are the Golden Signals?
**Answer:**
The four key indicators of service health:
*   **Latency:** Response time.
*   **Traffic:** Request volume.
*   **Errors:** Failed requests.
*   **Saturation:** Resource utilization.

### 9. What is alert fatigue?
**Answer:**
Too many alerts causing important ones to be ignored.

### 10. What is an SLI?
**Answer:**
A **Service Level Indicator (SLI)** is a quantitative metric that measures service behavior.
*   **Examples:** Request success rate, 95th percentile latency, Error rate.

### 11. What is an SLO?
**Answer:**
A **Service Level Objective (SLO)** is the target value for an SLI.
*   **Example:** 99.9% of requests must succeed over 30 days.
*   **Purpose:** SLOs define how reliable is “reliable enough.”

### 12. What is an SLA?
**Answer:**
A **Service Level Agreement (SLA)** is a contract with customers that includes:
*   SLOs.
*   Penalties or credits if breached.
*   👉 SREs design systems around SLOs, not SLAs.

### 13. Why are SLOs important?
**Answer:**
They define acceptable reliability and guide engineering tradeoffs.

### 14. What is an error budget?
**Answer:**
Allowed amount of failure = `1 − SLO`.

### 15. How do you use error budgets?
**Answer:**
Balance feature velocity vs reliability.

### 16. What happens when error budget is exhausted?
**Answer:**
Freeze releases and focus on reliability.

### 17. What are the stages of incident management?
**Answer:**
Detection → Mitigation → Resolution → Postmortem.

### 18. What is MTTR?
**Answer:**
Mean Time To Recovery.

### 19. What is a blameless postmortem?
**Answer:**
Focuses on system failures, not individuals.

### 20. Why is automation critical for SRE?
**Answer:**
Reduces toil and human error.

### 21. What is toil?
**Answer:**
Manual, repetitive, automatable operational work.

### 22. How much toil is acceptable in SRE?
**Answer:**
Less than 50% of engineering time.

### 23. How do you design a highly available system?
**Answer:**
Redundancy, stateless services, load balancing, multi-AZ.

### 24. Why is “100% availability” a bad goal?
**Answer:**
Infinite cost, diminishing returns, hides real priorities.

### 25. How do you reduce blast radius?
**Answer:**
Canary deployments, cell-based architecture, rate limiting.

### 26. What is graceful degradation?
**Answer:**
Partial functionality instead of total failure.

### 27. How do you design good alerts?
**Answer:**
Alert on symptoms, not causes.

### 28. What should you alert on?
**Answer:**
User-impacting conditions tied to SLOs.

### 29. Difference between RED and USE metrics?
**Answer:**
*   **RED:** Service metrics (Rate, Errors, Duration).
*   **USE:** Resource metrics (Utilization, Saturation, Errors).

### 30. How do you plan capacity?
**Answer:**
Historical trends + growth + safety margin.

### 31. What is saturation?
**Answer:**
Resource utilization nearing limits.

### 32. What deployment strategies reduce risk?
**Answer:**
Canary, blue-green, rolling updates.

### 33. How do you detect bad deployments quickly?
**Answer:**
SLO-based alerts, automated rollback.

### 34. What is RTO and RPO?
**Answer:**
*   **RTO:** Recovery Time Objective (how fast to recover).
*   **RPO:** Recovery Point Objective (how much data loss is acceptable).

### 35. How do you test disaster recovery?
**Answer:**
Chaos testing, game days, failover drills.

### 36. What is chaos engineering?
**Answer:**
Intentionally injecting failures to test resilience.

### 37. Tools used for chaos testing?
**Answer:**
Chaos Mesh, Gremlin, Litmus.

### 38. Your latency increased but CPU is low — why?
**Answer:**
I/O wait, locks, network latency, downstream dependency.

### 39. System is up but users report failures — what do you check?
**Answer:**
Error rates, saturation, dependencies, DNS, certificates.

### 40. How do you debug a memory leak in production?
**Answer:**
Heap dumps, profiling, gradual rollout, restart strategy.

### 41. What does “reliability” mean to you?
**Answer:**
Reliability is the probability that a service delivers correct results within an acceptable latency over time, as defined by user-focused SLIs and SLOs.

### 42. How do you decide what to monitor?
**Answer:**
Start from user experience, define SLIs that represent it, then monitor symptoms (latency, error rate, availability) instead of internal causes.
*   **Bad answer:** “CPU, memory, disk”.
*   **Good answer:** “Requests failing or getting slow”.

### 43. What is the relationship between SLOs and velocity?
**Answer:**
SLOs define the acceptable level of unreliability, and error budgets convert that into a policy for how fast we can release changes without harming users.
*   👉 If error budget is healthy → ship faster.
*   👉 If exhausted → slow down, stabilize.

### 44. Production system fails at 2 AM. What do you do first?
**Answer:**
1.  **Mitigate user impact immediately.**
2.  Roll back or reduce blast radius.
3.  Restore service.
4.  Collect data.
5.  Write blameless postmortem later.
*   **Note:** Google values restoration over root cause during incidents.

### 45. Why does Google prefer fewer, high-quality alerts?
**Answer:**
Because every alert should require human action. Too many alerts increase MTTR and cause alert fatigue, making real incidents harder to detect.

### 46. Why is 99.9% availability not “just 0.1% downtime”?
**Answer:**
Because downtime is often clustered, not evenly distributed, and impacts users disproportionately during peak traffic.

### 47. How do you justify downtime for maintenance?
**Answer:**
Downtime is acceptable if it fits within the error budget and is clearly communicated. Reliability without planned maintenance leads to worse failures later.

### 48. What is toil and why is it dangerous?
**Answer:**
Toil is manual, repetitive operational work that scales linearly with service growth. It diverts engineering time from improving system reliability.
*   **Google expectation:** Toil < 50%.

### 49. What does a good postmortem look like?
**Answer:**
*   Timeline.
*   What failed (system, not people).
*   Why defenses failed.
*   Concrete action items.
*   Shared learning.
*   👉 “Blameless” does not mean “accountability-free”.

### 50. What’s worse: one big outage or many small ones?
**Answer:**
Depends on user impact and SLOs, but many small failures often indicate systemic design issues and burn error budget silently.

### 51. How do you scale a system safely?
**Answer:**
*   Horizontal scaling.
*   Stateless services.
*   Load balancing.
*   Backpressure.
*   Gradual rollouts.
*   Continuous monitoring of SLIs.

### 52. Latency is high but CPU is low. Explain.
**Answer:**
Latency isn’t CPU-bound. Possible causes:
*   Network or disk I/O.
*   Lock contention.
*   Downstream dependency latency.
*   Garbage collection pauses.
*   **Note:** Google wants multi-layer reasoning, not guessing.

### 53. What is the difference between Monitoring and Observability?
**Answer:**
*   **Monitoring:** Tracking "known-unknowns" (detecting when a predefined threshold is crossed, e.g., CPU > 90%).
*   **Observability:** Understanding the internal state of a system from its external outputs (telemetry), allowing you to debug "unknown-unknowns" without shipping new code.

### 54. Explain the "Three Pillars of Observability."
**Answer:**
1.  **Metrics:** Aggregated numbers (trends).
2.  **Logs:** Discreet events (details).
3.  **Traces:** Request journeys (context).

### 55. What is a "Span" and how does it differ from a "Trace"?
**Answer:**
*   **Trace:** Represents the end-to-end journey of a request.
*   **Span:** A single unit of work within that trace (e.g., a database query or a specific API call).

### 56. How does context propagation work in distributed tracing?
**Answer:**
It involves the injection and extraction of metadata (like `trace-id` and `span-id`) into headers (e.g., W3C Trace Context) as the request moves across service boundaries.

### 57. What is the difference between Head-based and Tail-based sampling?
**Answer:**
*   **Head-based:** The decision to trace is made at the start (e.g., trace 5% of all traffic). High risk of missing rare errors.
*   **Tail-based:** The decision is made *after* the trace finishes. This allows you to keep 100% of errors and only 1% of successful traces, making it more efficient for debugging.

### 58. Explain the difference between a Counter, Gauge, and Histogram.
**Answer:**
*   **Counter:** Always increases (e.g., total requests).
*   **Gauge:** Can go up or down (e.g., memory usage).
*   **Histogram:** Samples observations and counts them in buckets (e.g., request latency).

### 59. What is "Cardinality" and why is it a problem for metrics?
**Answer:**
Cardinality refers to the number of unique label-value pairs. High cardinality (e.g., putting a `user_id` as a label in Prometheus) can crash your time-series database because it creates a unique time series for every single user.

### 60. Scenario: Latency spiked for 5 minutes, but CPU and Memory on all pods looked normal. How do you find the root cause?
**Answer:**
1.  **Check Traces:** Use a tool like Jaeger or Tempo to find traces from that 5-minute window.
2.  **Identify the Bottleneck:** Look for a "long span" in the trace waterfall. It might be a downstream 3rd-party API or a lock contention in the database.
3.  **Inspect Logs:** Use the `trace_id` from the slow trace to look at the logs for that specific request to see if there were any hidden errors or warnings.
4.  **Check Metrics:** Look for "Golden Signals" (Latency, Traffic, Errors, Saturation) specifically for the service identified in the trace.
