# Kubernetes Networking Interview Questions

### 1. Which k8s component assigns IP addresses to Pods, Services, and Worker Nodes?
**Answer:**

**Pod IP address**
*   **Assigned by:** CNI plugin (Container Network Interface).
*   **Examples:** Calico, Flannel, Weave, Cilium.
*   **Process:**
    1.  The kubelet calls the CNI plugin when creating a Pod.
    2.  The CNI plugin allocates the Pod IP and configures networking (routes, veth pairs, etc.).
    3.  Each Pod gets one unique IP that is routable within the cluster.
*   **Key point:** 👉 CNI plugin is responsible for Pod IP assignment.

**Service IP address (ClusterIP)**
*   **Assigned by:** kube-apiserver.
*   **Process:**
    1.  Service IPs come from a predefined Service CIDR.
    2.  The API server allocates the ClusterIP (and ExternalIPs if specified).
    3.  kube-proxy then programs rules to route traffic to backend Pods.
*   **Key point:** 👉 kube-apiserver assigns Service IPs.

**Worker Node IP address**
*   **Assigned by:** External infrastructure (not Kubernetes).
*   **Process:**
    *   Provided by the Cloud provider (AWS, GCP, Azure), DHCP, or Static network configuration.
    *   The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes in case of on-prem.
    *   Kubernetes only detects and registers the node IP via kubelet.
*   **Key point:** 👉 Node IPs are assigned outside Kubernetes.

---

### 2. How does DNS resolution work in Kubernetes?
**Answer:**
When a Pod tries to resolve a service name (e.g., Application in Pod A tries to reach another service named `orders`):

1.  **Pod initiates DNS lookup:**
    *   The query goes to CoreDNS.
    *   kubelet injects the CoreDNS Service IP into every Pod when the pod is scheduled/started.
    *   The OS resolver checks `/etc/resolv.conf` inside the Pod.

    ```yaml
    # Inside Pod resolv.conf points to coreDNS service with IP 10.96.0.10
    [ /etc ]$ cat resolv.conf
    search demo-namespace.svc.cluster.local svc.cluster.local cluster.local
    nameserver 10.96.0.10
    options ndots:5
    ```

2.  **CoreDNS consults Kubernetes API:**
    *   CoreDNS watches Services & Endpoints.
    *   It finds the `orders` Service in the same namespace.

3.  **CoreDNS returns ClusterIP:**
    *   The DNS response contains the Service ClusterIP (e.g., `10.96.34.21`).

4.  **Traffic is load-balanced:**
    *   The Pod connects to the ClusterIP.
    *   kube-proxy routes traffic to a healthy backend Pod.

---

### 3. What if CoreDNS can’t resolve the name?
**Answer:**
The request is forwarded to the upstream DNS, like the cloud provider’s DNS, using CoreDNS forwarding rules (which are configured in the CoreDNS configmap).

---

### 4. DNS returns an IP, but the service is still unreachable. Why?
**Answer:**
DNS resolution succeeding doesn’t guarantee reachability. If a service has no endpoints or is blocked by a NetworkPolicy, DNS will still return the ClusterIP, but the traffic connection will fail.

---

### 5. Do short service names work if we are not using the complete FQDN?
**Answer:**
Yes, short service names rely on DNS search domains (configured in `/etc/resolv.conf`) and work only within the same namespace. For cross-namespace communication, the full service name (FQDN) is required (e.g., `service.namespace.svc.cluster.local`).

---

### 6. How does a Headless Service work compared to a normal Service?
**Answer:**
Headless services don’t get a ClusterIP. Instead, CoreDNS returns the individual Pod IPs directly. This is useful for StatefulSets and databases where clients need direct access to specific Pods.

---

### 7. Is traffic also routed by CoreDNS?
**Answer:**
No. DNS resolves names to service IPs, but the actual traffic routing to Pods is handled by **kube-proxy** (iptables/IPVS) or **eBPF-based CNIs** (like Cilium).

---

### 8. How is CoreDNS different from a normal DNS server?
**Answer:**
CoreDNS is Kubernetes-aware. It dynamically watches the API server to resolve Service and Pod records automatically, while still forwarding external queries to an upstream DNS.

---

### 9. Explain how Pods get IP addresses in EKS with VPC CNI.
**Answer:**

**Step-by-Step Explanation:**

1.  **Pod is scheduled:** The Scheduler places the Pod on an EKS worker node.
2.  **kubelet invokes CNI:** The kubelet calls the AWS VPC CNI plugin (aws-node DaemonSet).
3.  **IP allocation happens:**
    *   The VPC CNI attaches Elastic Network Interfaces (ENIs) to the node if needed.
    *   It pre-allocates secondary IPs from the subnet to these ENIs.
4.  **Pod gets VPC IP:**
    *   One secondary IP is assigned to the Pod.
    *   It is bound to the Pod’s network namespace.
5.  **Pod becomes VPC-routable:**
    *   The Pod can talk directly to AWS services and other VPC resources.
    *   There is no overlay network and no NAT required for VPC communication.

**Follow-up Questions:**

*   **What limits the number of Pods per node?**
    *   ENI limits and secondary IP limits per EC2 instance type.
*   **What happens if IPs are exhausted?**
    *   Pod scheduling fails until more IPs or larger subnets are available.
*   **How is traffic routed?**
    *   Traffic is routed using native VPC routing tables, not encapsulation.

---

### 10. Explain the authentication flow when an EKS Pod pulls an image from ECR.
**Answer:**

**Step-by-Step Flow:**

1.  **Pod is scheduled:**
    *   Scheduler assigns the Pod to a worker node.
    *   Pod references an ECR image (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/app:tag`).

2.  **kubelet initiates image pull:**
    *   kubelet instructs the container runtime (containerd).
    *   **Note:** kubelet handles the image pull, not the Pod itself.

3.  **kubelet gets AWS credentials:**
    *   kubelet uses the **node IAM role**.
    *   Credentials are retrieved via the EC2 Instance Metadata Service (IMDS).

4.  **kubelet requests ECR auth token:**
    *   Calls `ecr:GetAuthorizationToken`.
    *   Receives a base64-encoded token (valid for ~12 hours).

5.  **Container runtime authenticates:**
    *   Token is passed to containerd.
    *   containerd logs into the ECR registry endpoint.

6.  **Image is pulled:**
    *   Image layers are downloaded over HTTPS.
    *   Pod starts once the image pull succeeds.

**Required IAM Permissions (Critical Detail):**
The node IAM role must allow:
```json
{
  "Effect": "Allow",
  "Action": [
    "ecr:GetAuthorizationToken",
    "ecr:BatchGetImage",
    "ecr:GetDownloadUrlForLayer"
  ],
  "Resource": "*"
}
```
Without these, Pods will fail with `ImagePullBackOff`.

---

### 11. Can we use IRSA (IAM Roles for Service Accounts) for image pull?
**Answer:**
**No, IRSA is NOT used for image pulls.**

*   **Reason:** Image pulling happens *before* the Pod starts. IRSA applies to the application running *inside* the Pod to access AWS services.
*   **Mechanism:** Only the **node IAM role** (attached to the worker node) is used by the kubelet to authenticate with ECR.
*   **Key Takeaway:** Pods do NOT authenticate to ECR; the node's kubelet does.

---

### 12. What could be the reasons for `ErrImagePull` or `ImagePullBackOff` in K8s?
**Answer:**
*   ❌ **Missing IAM permissions:** The node IAM role lacks `ecr:GetAuthorizationToken` or `ecr:BatchGetImage`.
*   ❌ **IMDS blocked:** The kubelet cannot retrieve credentials from the metadata service.
*   ❌ **Cross-account ECR:** The ECR repository policy does not allow the node role from the other account to pull images.
*   ❌ **Network Issues:** Node cannot reach ECR endpoint (missing NAT Gateway or VPC Endpoint).
*   ❌ **Typo:** Incorrect image name or tag.
