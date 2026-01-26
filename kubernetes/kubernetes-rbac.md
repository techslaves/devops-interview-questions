# Kubernetes RBAC Interview Questions

This document contains common interview questions related to Kubernetes RBAC (Role-Based Access Control) and their answers.

### 1. What is a ServiceAccount?
**Answer:**
A ServiceAccount provides an identity for Pods to authenticate to the Kubernetes API.

### 2. How is a ServiceAccount different from a User?
**Answer:**

| Feature | ServiceAccount | User |
| :--- | :--- | :--- |
| **Target** | For workloads (Pods) | For humans |
| **Scope** | Namespaced | Cluster-wide |
| **Management** | Managed by Kubernetes | External identity (LDAP, OIDC, Cloud IAM) |
| **Credentials** | Uses tokens (JWT) | Uses certs / OIDC tokens |

### 3. Explain the components of RBAC.
**Answer:**
There are four main objects:
*   **Role:** Defines permissions (verbs like `get`, `list`) within a specific namespace.
*   **ClusterRole:** Defines permissions at the cluster level (across all namespaces or for cluster-wide resources like Nodes).
*   **RoleBinding:** Links a Role to a subject (User/Group/ServiceAccount) within a namespace.
*   **ClusterRoleBinding:** Links a ClusterRole to a subject across the entire cluster.

### 4. How does a Pod use a ServiceAccount?
**Answer:**
*   A token is auto-mounted at: `/var/run/secrets/kubernetes.io/serviceaccount`.
*   The application inside the Pod uses this token to call the `kube-apiserver`.

### 5. Can multiple Pods use the same ServiceAccount?
**Answer:**
✅ **Yes** — this is a common pattern for workloads that share the same permission requirements.

### 6. What is RBAC in Kubernetes?
**Answer:**
**Role-Based Access Control (RBAC)** is a method of regulating access to computer or network resources based on the roles of individual users within your organization. In Kubernetes, it defines **who** (Subject) can do **what** (Verbs) on **which resources** (Objects).

### 7. What is the principle of least privilege in RBAC?
**Answer:**
You should grant:
*   **Minimum verbs:** Only the actions needed (e.g., `get`, `list` but not `delete`).
*   **Minimum resources:** Only the specific resources needed (e.g., `pods` but not `secrets`).
*   **Minimum scope:** Use `Role` (namespace-scoped) instead of `ClusterRole` whenever possible.

### 8. How do ServiceAccounts get permissions?
**Answer:**
ServiceAccounts get permissions by being bound to roles via bindings:
*   **Role** → **RoleBinding** (Permissions in a specific namespace).
*   **ClusterRole** → **RoleBinding** (Permissions in a specific namespace, reusing a cluster-wide role definition).
*   **ClusterRole** → **ClusterRoleBinding** (Permissions across the entire cluster).

### 9. How do you grant a Pod permissions to list Secrets in its own namespace?
**Answer:**
1.  Create a **ServiceAccount**.
2.  Create a **Role** with the `list` verb for the `secrets` resource.
3.  Create a **RoleBinding** to tie the ServiceAccount to the Role.
4.  Set `serviceAccountName` in the Pod's YAML to match the ServiceAccount.

### 10. What is a kubeconfig file and what are its main sections?
**Answer:**
It is a YAML file that stores cluster connection details.
**Main Sections:**
*   **clusters:** API endpoint URL and Certificate Authority (CA) data.
*   **users:** Credentials (client certificates, tokens, or auth provider args).
*   **contexts:** The mapping that ties a **user** to a **cluster** (and optionally a default namespace).

### 11. How do you troubleshoot a "Forbidden" error for a user who can log in?
**Answer:**
1.  Use `kubectl auth can-i <verb> <resource> --as <user>` to verify permissions.
    *   If it returns "no," the issue is in the **RBAC Bindings**.
    *   If it returns "yes" but they still fail, check for **NetworkPolicies** or **Admission Controllers** (like OPA/Gatekeeper) that might be blocking the request.

### 12. How did we traditionally give IAM users access to EKS, and what is the new recommended way?
**Answer:**
*   **Traditionally:** We used the `aws-auth` ConfigMap in the `kube-system` namespace to map IAM ARNs to Kubernetes groups.
*   **New Recommended Way:** **EKS Access Entries**. This allows managing access via the AWS API/Console directly, without editing a Kubernetes ConfigMap, preventing syntax errors and lockouts.

### 13. What are EKS "Access Policies"?
**Answer:**
They are AWS-managed policies (e.g., `AmazonEKSAdminPolicy`) that you attach to an **Access Entry**. This simplifies RBAC because you don't have to manually create Kubernetes `ClusterRoles` and `ClusterRoleBindings` for standard administrative tasks.

### 14. If a user has the AdministratorAccess IAM policy, do they automatically have access to the EKS cluster?
**Answer:**
**No.** By default, only the IAM entity (user/role) that **created** the cluster has access (system:masters). Other IAM users, even with full AWS Admin permissions, must be explicitly added to the cluster's RBAC (via Access Entries or `aws-auth` ConfigMap) to interact with the Kubernetes API.

### 15. What is IRSA (IAM Roles for Service Accounts)?
**Answer:**
It allows a Kubernetes **ServiceAccount** to assume an AWS **IAM Role**.
*   **Mechanism:** It uses an **OIDC provider** to trust the Kubernetes cluster.
*   **Benefit:** It enables **"Least Privilege"** because you grant AWS permissions (e.g., S3 access) to a specific **Pod** rather than attaching the policy to the entire Worker Node (which would expose permissions to all pods on that node).
