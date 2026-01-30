# Jenkins Interview Questions and Answers

This document contains common interview questions related to Jenkins and their answers.

### 1. What is Jenkins and why is it used?
**Answer:**
Jenkins is an open-source CI/CD automation server that orchestrates build, test, and deployment workflows. It integrates with SCM, testing tools, container platforms, and cloud services to enable continuous delivery with minimal manual intervention.

### 2. Freestyle job vs Pipeline job?
**Answer:**
*   **Freestyle jobs:** UI-driven and suitable for simple tasks, but they lack reusability and version control.
*   **Pipeline jobs:** Defined as code using a `Jenkinsfile`, allowing complex workflows, better traceability, and Git-based versioning.

### 3. What is a Jenkinsfile?
**Answer:**
A `Jenkinsfile` defines the entire CI/CD pipeline using Groovy DSL. Since it’s stored in SCM, it enables pipeline versioning, peer review, and reproducibility across environments.

### 4. Declarative vs Scripted pipeline?
**Answer:**
*   **Declarative pipelines:** Follow a structured syntax, making them easier to maintain and less error-prone.
*   **Scripted pipelines:** Offer full Groovy flexibility and are used when complex logic or dynamic behavior is required.

### 5. What are Jenkins agents?
**Answer:**
Agents are worker nodes that execute pipeline steps. They allow Jenkins to distribute workloads, isolate builds, and run jobs on different OS or runtime environments.

---

## 🔸 Pipeline & Execution

### 6. What are stages and steps?
**Answer:**
*   **Stages:** Represent logical phases like `build` or `deploy`, improving pipeline visualization.
*   **Steps:** The actual commands executed inside a stage, such as running tests or building artifacts.

### 7. What is `agent any` vs `agent none`?
**Answer:**
*   `agent any`: Runs the entire pipeline on a single available agent.
*   `agent none`: Gives granular control by assigning different agents per stage, which is useful for multi-environment pipelines.

### 8. What is the purpose of the `post` block?
**Answer:**
The `post` block defines actions that run after pipeline execution, such as sending notifications or cleaning resources, regardless of pipeline success or failure.

### 9. How are parameters passed to Jenkins jobs?
**Answer:**
Parameters allow dynamic pipeline execution by accepting inputs at runtime, such as environment names or versions, making pipelines reusable across deployments.

### 10. How do you share variables across stages?
**Answer:**
Variables can be shared using environment variables or global Groovy variables, ensuring data consistency across pipeline stages.

---

## 🔸 Jenkins Security

### 11. How does Jenkins authentication and authorization work?
**Answer:**
*   **Authentication:** Verifies user identity using LDAP, Active Directory, or SSO.
*   **Authorization:** Defines permissions using RBAC or matrix-based strategies to enforce least-privilege access.

### 12. How does Jenkins handle secrets securely?
**Answer:**
Secrets are stored in the Jenkins Credentials Store and injected at runtime, preventing exposure in code or logs while supporting credential rotation.

### 13. Why should secrets not be hardcoded in Jenkinsfile?
**Answer:**
Since `Jenkinsfile`s are stored in SCM, hardcoding secrets risks exposure and violates security best practices and compliance requirements.

### 14. Explain Jenkins master–agent architecture.
**Answer:**
The Jenkins controller (master) manages scheduling, UI, and plugins, while agents handle execution. This separation improves scalability and keeps the controller lightweight.

### 15. What happens if the Jenkins controller goes down?
**Answer:**
All builds stop because scheduling is centralized. Running builds fail, highlighting the need for backups, monitoring, and high-availability setups.

### 16. How do you implement Jenkins high availability?
**Answer:**
By externalizing storage, backing up `JENKINS_HOME`, and running Jenkins on Kubernetes with persistent volumes or using warm-standby setups.

---

## 🔸 Jenkins with Docker & Kubernetes

### 17. How does Jenkins integrate with Docker?
**Answer:**
Jenkins can build Docker images and run pipeline steps inside containers, ensuring environment consistency across build stages.

### 18. What is the Docker-in-Docker issue?
**Answer:**
Docker-in-Docker introduces security risks and performance overhead. A safer approach is mounting the host Docker socket or using Kubernetes-based agents.

### 19. Jenkins on Kubernetes – how does it work?
**Answer:**
Jenkins dynamically provisions pods as agents using the Kubernetes plugin, allowing on-demand, isolated, and auto-scaled build environments.

### 20. Advantages of Kubernetes-based Jenkins agents?
**Answer:**
They provide ephemeral environments, better resource utilization, improved security isolation, and automatic scaling based on workload.

---

## 🔸 Performance & Reliability

### 21. Why do Jenkins builds slow down over time?
**Answer:**
Build history growth, excessive plugins, limited executors, and disk I/O constraints degrade performance if not managed.

### 22. How do you optimize Jenkins performance?
**Answer:**
By cleaning old builds, limiting plugins, using ephemeral agents, and offloading heavy tasks to scalable workers.

### 23. How do you troubleshoot a stuck Jenkins build?
**Answer:**
Check executor availability, agent health, logs, thread dumps, and external dependencies such as SCM or artifact repositories.

---

## 🔸 CI/CD Design & Scenarios

### 24. How do you prevent concurrent deployments?
**Answer:**
By using locks, milestones, or disabling concurrent builds to ensure only one deployment runs for a given environment.

### 25. How is rollback implemented in Jenkins pipelines?
**Answer:**
By deploying versioned artifacts and triggering rollback stages using blue-green or canary deployment strategies.

### 26. How can Jenkins jobs be triggered?
**Answer:**
Through SCM webhooks, scheduled jobs, upstream pipelines, or REST API calls for event-driven automation.

### 27. How do you manage Jenkins configuration as code?
**Answer:**
Using Jenkins Configuration as Code (JCasC) to define system settings in YAML, ensuring consistency and repeatability.

### 28. Jenkins vs GitHub Actions?
**Answer:**
*   **Jenkins:** Offers deep customization and enterprise flexibility.
*   **GitHub Actions:** Provides managed, GitHub-native CI/CD with less infrastructure overhead.

### 29. Biggest challenges with Jenkins?
**Answer:**
Plugin dependency management, maintenance overhead, and ensuring secure, scalable architecture.

### 30. Why is Jenkins still widely used?
**Answer:**
Its maturity, vast plugin ecosystem, and flexibility make it suitable for complex enterprise CI/CD workflows.
