# AWS EKS Interview Questions and Answers

This document contains common interview questions related to AWS EKS (Elastic Kubernetes Service) and their answers.

### 1. What is OIDC in EKS?
**Answer:**
OIDC (OpenID Connect) in EKS allows Kubernetes service accounts to authenticate to AWS IAM using JWT tokens issued by the EKS cluster.

**Flow:**
```text
Pod
 └── ServiceAccount (with IAM role annotation)
      └── Kubernetes issues JWT
           └── AWS IAM OIDC Provider trusts EKS
                └── STS: AssumeRoleWithWebIdentity
                     └── Temporary AWS credentials
```

### 2. What components are managed by AWS in EKS?
**Answer:**
AWS manages the **Control Plane**:
*   API Server
*   etcd
*   Scheduler
*   Controller Manager

The **User** manages:
*   Worker nodes
*   Networking (VPC CNI)
*   Add-ons (CoreDNS, kube-proxy, etc.)

### 3. Is the EKS control plane highly available?
**Answer:**
**Yes.** It runs across multiple Availability Zones (AZs) automatically, ensuring high availability and fault tolerance.

### 4. What is IRSA?
**Answer:**
**IRSA (IAM Roles for Service Accounts)** uses OIDC to allow pods to assume IAM roles securely. It enables fine-grained permission control at the Pod level rather than the Node level.

### 5. How do pods get IPs in EKS?
**Answer:**
Using the **AWS VPC CNI** plugin. Pods receive VPC-native IP addresses directly from the subnet, allowing them to be treated as first-class citizens in the VPC network.

### 6. What are ENIs and why are they important?
**Answer:**
**ENIs (Elastic Network Interfaces)** are virtual network cards attached to EC2 instances.
*   **Importance:** Pod IPs are allocated from the secondary IP addresses available on the node's ENIs. The number of ENIs and IPs per ENI determines the maximum number of pods a node can run.

### 7. How does EBS work with EKS?
**Answer:**
*   **AZ-bound:** EBS volumes are specific to an Availability Zone.
*   **Driver:** Requires the **EBS CSI Driver**.
*   **Scheduling:** The Pod must be scheduled in the **same AZ** as the EBS volume.

### 8. EBS vs EFS in EKS?
**Answer:**

| Feature | EBS (Elastic Block Store) | EFS (Elastic File System) |
| :--- | :--- | :--- |
| **Availability** | AZ-bound (Single AZ) | Multi-AZ (Regional) |
| **Type** | Block storage | File system (NFS) |
| **Access Mode** | RWO (ReadWriteOnce) | RWX (ReadWriteMany) |
| **Use Case** | Databases, High Performance | Shared storage, CMS, CI/CD |

### 9. How do you secure an EKS cluster?
**Answer:**
*   **Private Endpoint:** Restrict API server access to within the VPC.
*   **IRSA:** Use IAM Roles for Service Accounts for least privilege.
*   **Pod Security Standards:** Enforce security contexts (e.g., non-root).
*   **Network Policies:** Restrict pod-to-pod communication.
*   **Secrets Management:** Integrate with AWS Secrets Manager or use sealed secrets.

### 10. How do you enable logging in EKS?
**Answer:**
*   **Control Plane Logs:** Enable via EKS Console/CLI to send logs (API, Audit, Authenticator, Controller, Scheduler) to **CloudWatch Logs**.
*   **Pod Logs:** Use **Fluent Bit** or **OpenTelemetry** to ship container logs to CloudWatch, OpenSearch, or S3.

### 11. How to debug networking issues in EKS?
**Answer:**
*   Check `aws-node` (VPC CNI) logs.
*   Check `ipamd` logs for IP allocation errors.
*   Verify ENI and IP limits on the node.
*   Check Security Groups and NACLs.

### 12. Why does EKS separate control plane and data plane?
**Answer:**
This separation improves security, reliability, and operational simplicity. The control plane is isolated and protected by AWS, while customers retain flexibility over worker nodes. It also allows AWS to upgrade control planes independently, reducing the blast radius.

### 13. How does authentication work in EKS?
**Answer:**
*   **Authentication:** Done using **AWS IAM**.
*   **Authorization:** Handled by **Kubernetes RBAC**.
*   **Mapping:** IAM identities (Users/Roles) are mapped to Kubernetes users or groups via the `aws-auth` ConfigMap (or Access Entries in newer versions). This allows centralized identity management using IAM while still following Kubernetes-native access controls.

### 14. Explain IRSA and why it’s critical in production.
**Answer:**
IAM Roles for Service Accounts (IRSA) uses OIDC federation to allow pods to assume IAM roles using short-lived credentials. Each pod gets least-privilege AWS access without storing secrets.
**Critical in production to avoid:**
*   Node role over-permission (giving all pods on a node access to S3).
*   Static AWS credentials (security risk).
*   Credential leakage risks.

### 15. How does pod networking work in EKS?
**Answer:**
EKS uses the **AWS VPC CNI**, which assigns VPC-native IP addresses directly to pods. This allows pods to communicate with AWS services without NAT or overlays and enables native integration with Security Groups and NACLs.
**Trade-off:**
*   **Pros:** Native networking, simpler routing, high performance.
*   **Cons:** IP exhaustion risk in large clusters (consumes VPC IPs).

### 16. How do you design EKS to avoid IP exhaustion?
**Answer:**
*   **Larger CIDR:** Use larger CIDR ranges (e.g., `/16`) for the VPC.
*   **Prefix Delegation:** Enable VPC CNI prefix delegation to assign `/28` prefixes to ENIs instead of individual IPs.
*   **Secondary CIDR:** Add a secondary CIDR block (e.g., `100.64.0.0/16`) specifically for pods.
*   **Spread Workloads:** Distribute workloads across multiple AZs.

### 17. When would you choose EFS over EBS?
**Answer:**
**Use EFS when you need:**
*   Multi-AZ access (pods in different zones sharing data).
*   RWX (ReadWriteMany) volumes (multiple pods writing simultaneously).
*   Shared data between pods (e.g., web content, CI artifacts).

**Use EBS for:**
*   High-performance databases (low latency).
*   Single-writer workloads (RWO).

### 18. How do you design a secure EKS cluster?
**Answer:**
*   **Private API Endpoint:** Disable public access or restrict via CIDR.
*   **IRSA Everywhere:** Never use node roles for application permissions.
*   **Network Policies:** Default deny all, allow specific traffic.
*   **Encrypted Secrets:** Use KMS envelope encryption for Kubernetes secrets.
*   **Restrictive Node IAM Roles:** Only allow necessary permissions (CNI, registry pull).
*   **Audit Logging:** Enable and monitor control plane audit logs.

### 19. Explain the EKS cluster upgrade process.
**Answer:**
EKS upgrades are done in phases: control plane first, then worker nodes, and finally cluster add-ons—while ensuring application availability through rolling updates and compatibility checks.

**Step-by-Step Process:**

1.  **Pre-upgrade Planning:**
    *   Check Kubernetes version skew (Control plane can be +1 version ahead of nodes).
    *   Review API deprecations.
    *   Verify compatibility of add-ons (VPC CNI, CoreDNS, kube-proxy, CSI drivers).

2.  **Upgrade the Control Plane (AWS-managed):**
    *   Triggered via AWS Console/CLI.
    *   AWS upgrades API Server, etcd, etc., AZ by AZ.
    *   **No downtime** for workloads (Control plane upgrade does not restart pods).

3.  **Upgrade Cluster Add-ons:**
    *   Upgrade `aws-vpc-cni`, `coredns`, `kube-proxy`, and CSI drivers to match the new version.
    *   **Best Practice:** Upgrade CNI before nodes to avoid networking issues.

4.  **Upgrade Worker Nodes (Data Plane):**
    *   **Managed Node Groups:** EKS performs rolling upgrades (drains nodes, terminates old ones, launches new ones).
    *   **Self-Managed Nodes:** Create new node group, drain old nodes manually, delete old group.

5.  **Application Safety Controls:**
    *   Use **PodDisruptionBudgets (PDBs)** to ensure availability during node drains.
    *   Ensure Readiness & Liveness probes are configured.

6.  **Post-upgrade Validation:**
    *   Verify node versions, add-on health, and run smoke tests.

### 20. Scenario: After control plane upgrade, CoreDNS pods started crashing or were in CrashLoopBackOff. What happened, and how did you resolve it?
**Answer:**
**Symptoms:**
*   Pods cannot resolve Kubernetes services or external domains.
*   Errors: `nslookup: connection timed out`, `SERVFAIL`.
*   CoreDNS pods status: `CrashLoopBackOff` or `Pending`.

**Root Cause:**
*   CoreDNS version incompatible with the new Kubernetes version.
*   Insufficient CPU/memory limits after upgrade.
*   ConfigMap syntax mismatch.

**Resolution (Production Fix):**
1.  **Check Version:** `kubectl get deployment coredns -n kube-system -o wide`
2.  **Upgrade Add-on:**
    ```bash
    aws eks update-addon --cluster-name prod-cluster --addon-name coredns --resolve-conflicts OVERWRITE
    ```
3.  **Increase Resources:** If OOMKilled, increase memory limits.
    ```yaml
    resources:
      limits:
        memory: "170Mi"
    ```

**Architect Takeaway:** Always upgrade CoreDNS to the recommended version compatible with the new Kubernetes release.

### 21. Scenario: After control plane upgrade, Pods stayed in ContainerCreating, PodInitializing, or Init:0/1. Why?
**Answer:**
**Root Cause:**
*   `aws-vpc-cni` version incompatible with upgraded control plane.
*   IPAM (`ipamd`) crashes.
*   Prefix delegation disabled or misconfigured.
*   ENI/IP exhaustion.

**Impact:**
*   New pods can’t get IPs.
*   Cluster appears healthy but workloads fail to start.

**Resolution:**
1.  **Upgrade CNI:** Upgrade `aws-vpc-cni` add-on to the matching version.
2.  **Enable Prefix Delegation:**
    ```bash
    kubectl set env daemonset aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true
    ```
3.  **Check Limits:** Verify ENI/IP limits and restart `aws-node` DaemonSet.

**Architect Takeaway:** Networking failures often manifest as pod startup issues, but the root cause is usually the infrastructure layer (CNI).

### 22. Scenario: Pods stuck in Pending or ContainerCreating with `AttachVolume.Attach failed` events.
**Answer:**
**Root Cause:**
*   EBS CSI driver version incompatible.
*   Missing IAM permissions (IRSA) after upgrade.
*   Controller pods failing.
*   AZ mismatch scheduling (Pod scheduled in AZ where volume doesn't exist).

**Impact:**
*   Stateful workloads (DBs, queues) down.
*   PersistentVolumeClaims (PVCs) stuck.

**Resolution:**
1.  **Upgrade Driver:** Upgrade EBS CSI add-on.
2.  **Verify Permissions:** Check IRSA permissions for the CSI controller and Node IAM role policies.
3.  **Restart:** Restart CSI controller pods.
4.  **Check Topology:** Verify Pod and Volume AZ alignment.

**Architect Takeaway:** Storage drivers are control-plane dependent components, not just optional add-ons.

### 23. Scenario: Deprecated APIs caused workloads to fail after upgrade. How did you handle it?
**Answer:**
**Symptoms:**
*   Deployments failed with errors like: `no matches for kind "Ingress" in version "extensions/v1beta1"`.

**Root Cause:**
*   Removal of older Kubernetes APIs (e.g., `extensions/v1beta1`, `networking.k8s.io/v1beta1`) in newer versions.

**Impact:**
*   Helm charts fail, controllers crash, CI/CD pipelines break.

**Resolution Strategy:**
1.  **Detection:**
    *   Use tools like **kubent (Kube No Trouble)** or **pluto** before upgrading to scan for deprecated APIs.
    *   Check `kubectl api-resources`.
2.  **Fix Manifests:**
    *   Update YAMLs (e.g., change `apiVersion: extensions/v1beta1` to `apiVersion: networking.k8s.io/v1` for Ingress).
    *   Update Helm charts and CI/CD templates.
3.  **Validation:**
    *   Test in staging cluster.
    *   Run GitOps dry-runs and Helm template validation.
4.  **Rollout:** Controlled rollout via Canary deployment.
