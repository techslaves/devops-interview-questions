## 🔥 Questions

<details>
<summary>Which k8s component assign IP address to Pod, services and worker nodes in k8s</summary>

1. Pod IP address

Assigned by: CNI plugin (Container Network Interface)
Examples: Calico, Flannel, Weave, Cilium
The kubelet calls the CNI plugin when creating a Pod
The CNI plugin:
Allocates the Pod IP
Configures networking (routes, veth pairs, etc.)
Each Pod gets one unique IP that is routable within the cluster
Key point:
👉 CNI plugin is responsible for Pod IP assignment

2. Service IP address (ClusterIP)
Assigned by: kube-apiserver
Service IPs come from a predefined Service CIDR
The API server allocates:
ClusterIP
ExternalIPs (if specified)
kube-proxy then programs rules to route traffic to backend Pods
Key point:
👉 kube-apiserver assigns Service IPs

3. Worker Node IP address
Assigned by: External infrastructure (not Kubernetes), The kubelet or the cloud-controller-manager is configured to assign IP addresses to Nodes in case of on-prem.
Provided by:
Cloud provider (AWS, GCP, Azure)
DHCP
Static network configuration
Kubernetes only detects and registers the node IP via kubelet

Key point:
👉 Node IPs are assigned outside Kubernetes
</b></details>
