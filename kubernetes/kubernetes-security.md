# Kubernetes Security Interview Questions and Answers

This document contains common interview questions related to Kubernetes Security and their answers.

## 1. How does Kubernetes handle Linux kernel level security?
**Answer:**

**1. Namespaces (Isolation)**

Namespaces are the fundamental building block of containers. They provide the illusion that a process is running in its own dedicated environment. The kernel ensures that a process in one namespace cannot see or affect processes in another.

**PID Namespace:** Prevents a pod from seeing processes in other pods or the host.

**Network Namespace:** Each pod gets its own IP address and routing table.

**Mount Namespace:** Restricts the pod’s view of the filesystem.

**2. Control Groups (cgroups)**

While Namespaces provide isolation, cgroups provide resource management. They prevent a single container from consuming all system resources (a "noisy neighbor" or DoS attack).

**Memory Limits:** The kernel will trigger an OOM (Out of Memory) kill if a container exceeds its limit.

**CPU Shares:** Ensures fair distribution of CPU cycles.

**Device Whitelisting:** Limits which physical devices (/dev/) a container can access.

**3. Seccomp (Secure Computing Mode)**

Seccomp restricts the system calls (syscalls) a process can make to the Linux kernel. Even if an attacker gains shell access to your container, Seccomp can prevent them from executing dangerous commands like mount or ptrace.

Implementation: In Kubernetes, you apply a Seccomp profile (usually a JSON file) via the securityContext.

The "**RuntimeDefault**" Profile: Kubernetes can automatically apply a default profile that blocks roughly 50 dangerous syscalls out of the 300+ available.

**4. AppArmor and SELinux (MAC)**

These are **Mandatory Access Control (MAC)** systems. Unlike standard Linux permissions (where the "owner" decides access), MAC allows the administrator to define system-wide security policies.

**AppArmor:** Uses program paths to restrict capabilities (e.g., "this application cannot write to /etc/"). It is common on Ubuntu/Debian.

**SELinux:** Uses labels (contexts) assigned to every file, process, and port. It is the default on RHEL/Fedora/OpenShift.

**Kubernetes usage:** You can label your Pods so the kernel prevents them from accessing sensitive host files, even if the pod is running as "root."

**5. Linux Capabilities**

Traditionally, Linux had a binary choice: you were either a normal user or the all-powerful root. Capabilities break "root" powers into small pieces.

Instead of giving a container full root access, Kubernetes allows you to grant only the specific privilege needed:

**CAP_NET_BIND_SERVICE:** Allows a process to bind to ports below 1024.

**CAP_CHOWN:** Allows changing file ownership.

Best Practice: Use allowPrivilegeEscalation: false in your Pod Security Standards to prevent processes from gaining more permissions than they started with.

## 2. Give me a YAML example of a "Hardened" Pod that implements Seccomp and drops Linux Capabilities? You can use Kubernetes documentation
**Answer:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened-web-server
spec:
  securityContext:
    # 1. Apply a Seccomp profile at the Pod level
    # 'RuntimeDefault' blocks dangerous syscalls like reboot or mount
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: web-app
    image: nginx:alpine
    securityContext:
      # 2. Prevent the process from gaining more privileges than its parent
      allowPrivilegeEscalation: false
      
      # 3. Run as a non-root user (Linux Namespace/UID isolation)
      runAsUser: 1000
      runAsNonRoot: true

      # 4. Linux Capabilities: Drop ALL 'root' powers, then add back only what's needed
      capabilities:
        drop:
          - ALL
        add:
          - NET_BIND_SERVICE # Allows binding to a port (if needed)

      # 5. Make the root filesystem read-only (Kernel Mount Namespace)
      # This prevents attackers from installing malware or modifying binaries
      readOnlyRootFilesystem: true
```
**seccompProfile:** Without this, a compromised container can attempt to exploit vulnerabilities in the kernel itself by sending "malformed" system calls. By using the RuntimeDefault profile, you are effectively putting a filter between your app and the kernel's CPU.

**capabilities: drop: - ALL:** By default, even "non-root" containers often carry certain capabilities that could be abused. Dropping all of them ensures that even if an attacker finds a way to execute code, they cannot change file ownership (CHOWN), kill other processes, or bypass file permissions.

**readOnlyRootFilesystem:** This leverages the kernel's mount isolation. If an attacker tries to run apt-get install or download a script to /tmp, the kernel will block the write operation at the system level, making the attack much harder to persist.

**runAsNonRoot:** This ensures the process is mapped to a high-numbered UID in the kernel's user namespace. Even if the process "escapes" the container, it remains a powerless user on the host node.

## 3. How does Security context AppArmor and SELinux operate in Kubernetes?
**Answer:**

They operate differently at the kernel level.

- **AppArmor** is path-based (e.g., "deny writing to /etc/").

- **SELinux** is label-based (e.g., "only processes with the web_t label can touch files with the web_data_t label").

| Feature | AppArmor | SELinux |
| :--- | :--- | :--- |
| **Logic** | **Path-based:** Rules are tied to file paths. | **Label-based:** Rules are tied to object labels. |
| **Default Distros** | Ubuntu, Debian, SUSE. | RHEL, CentOS, Fedora. |
| **Complexity** | Easier to write/read profiles. | Steeper learning curve, highly granular. |
| **K8s config** | Refers to a profile name on the host. | Defines specific security context labels. |

## 4. Give me an example of **AppArmor** implementation in Kuberbetes
**Answer:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apparmor-locked-down
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      securityContext:
        # Modern K8s (v1.30+) uses this field
        appArmorProfile:
          type: Localhost
          localhostProfile: k8s-apparmor-example-deny-write
      containers:
      - name: nginx
        image: nginx
```
To use AppArmor, the profile must first be loaded onto the worker node itself (usually via a DaemonSet or manual SSH). Kubernetes simply tells the container runtime to use a profile that already exists on the host.

**Note:** In versions older than v1.30, AppArmor was controlled via annotations like container.apparmor.security.beta.kubernetes.io/nginx: localhost/k8s-apparmor-example-deny-write.

## 5. Give me an example of **SELinux** implementation in Kuberbetes
**Answer**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: selinux-deployment
spec:
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      securityContext:
        seLinuxOptions:
          user: "system_u"
          role: "system_r"
          type: "container_t"
          # 'level' is used for Multi-Category Security (MCS) 
          # to isolate pods from each other
          level: "s0:c123,c456" 
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "sleep 1h"]
```
## 6.  How to create a simple AppArmor profile to block all write access to a specific directory?

**Answer:**

To implement AppArmor, you first need to define a profile on your worker nodes. The Linux kernel uses this profile to monitor the process and block any action not explicitly permitted.
1. The AppArmor Profile (On the Host)

Save this file on your Kubernetes worker nodes (e.g., at /etc/apparmor.d/k8s-deny-write):

```bash
# This profile allows reading, but denies writing to /var/www/
profile k8s-deny-write flags=(attach_disconnected) {
  # Include default abstractions (network, base system)
  include <abstractions/base>

  # Allow all read/execute
  file,
  
  # Specifically deny any write access to /var/www/ and its subdirectories
  deny /var/www/** w,
}
```
After saving, you must load it into the kernel: apparmor_parser -q << 'EOF' (paste profile) 

2. The Kubernetes Deployment

Now, we tell Kubernetes to attach this specific profile to our container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hardened-app
spec:
  selector:
    matchLabels:
      app: web-server
  template:
    metadata:
      labels:
        app: web-server
    spec:
      securityContext:
        # This tells the kernel to look for the 'k8s-deny-write' 
        # profile we just loaded on the host.
        appArmorProfile:
          type: Localhost
          localhostProfile: k8s-deny-write
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

3. Testing the Kernel Enforcement

If you try to bypass this security from within the pod, the Linux kernel will block the operation, even if you are the "root" user inside the container.

Try to write to the protected directory:

```bash
kubectl exec -it <pod-name> -- touch /var/www/test.txt
```
Result: The command will fail with: touch: /var/www/test.txt: Permission denied.

Check the Kernel Logs: On the worker node host, you will see the violation recorded by the kernel in dmesg or syslog:

```text
audit: type=1400 audit(1641234567.890): apparmor="DENIED" operation="mknod" profile="k8s-deny-write" name="/var/www/test.txt" pid=12345 comm="touch" requested_mask="c" denied_mask="c"
```

## 7. Followup question to last one: Why this is better than standard permissions

**Answer:**

Standard Linux permissions (chmod 555) can sometimes be bypassed if a process manages to escalate to root or if a volume is misconfigured. AppArmor is a "Mandatory Access Control" (MAC), meaning even the root user inside the container is bound by the profile. The kernel checks the AppArmor policy after standard permissions, adding an inescapable layer of security.

## 8. Explain how to automate the loading of these AppArmor profiles across a large cluster?
**Answer:**
We use a DaemonSet.

A DaemonSet ensures that a specific Pod runs on every single node. This "Loader" Pod carries your AppArmor profiles and uses a script to load them into the host's kernel.
The AppArmor Loader Pattern
1. Store Profiles in a ConfigMap

First, put your AppArmor profile text into a ConfigMap so it's accessible to the cluster.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: apparmor-profiles
data:
  k8s-deny-write: |
    profile k8s-deny-write flags=(attach_disconnected) {
      include <abstractions/base>
      file,
      deny /var/www/** w,
    }
```
2. The Loader DaemonSet

This Pod runs on every node, mounts the host's AppArmor directory, and runs apparmor_parser to sync the profiles.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: apparmor-loader
spec:
  selector:
    matchLabels:
      name: apparmor-loader
  template:
    metadata:
      labels:
        name: apparmor-loader
    spec:
      containers:
      - name: apparmor-loader
        image: google/apparmor-loader:latest # Community image or custom busybox
        securityContext:
          privileged: true # Needs high privileges to talk to the host kernel
        volumeMounts:
        - name: profiles
          mountPath: /etc/apparmor.d
        - name: sys
          mountPath: /sys # Access to the kernel's AppArmor interface
      volumes:
      - name: profiles
        configMap:
          name: apparmor-profiles
      - name: sys
        hostPath:
          path: /sys
```
**Why this is a "Production Grade" setup**

**Auto-Scaling:** When a new node joins the cluster, the DaemonSet automatically spins up on it and loads the profile before your apps arrive.

**Consistency:** Every node is guaranteed to have the exact same security policy.

**Centralized Updates:** If you need to allow a new directory, you update the ConfigMap, and the DaemonSet (with a small restart or watch script) updates the kernel across the entire fleet.

## 8. How would you enforce that no developer can deploy a pod unless it has these AppArmor settings enabled?
**Answer:**
To enforce these kernel-level protections at scale, you can use Pod Security Admission (PSA). PSA is the successor to Pod Security Policies (PSP); it is a built-in controller that evaluates every Pod request against a set of standards.

1. The Three Policy Levels

PSA provides three "off-the-shelf" levels of security. As you move from Baseline to Restricted, the kernel-level requirements become mandatory:

**Privileged:** No restrictions (for system infra).

**Baseline:** Prevents known privilege escalations. It enforces the RuntimeDefault AppArmor profile if the host supports it.

**Restricted:** The highest level. It requires all containers to drop capabilities and enforces specific Seccomp and AppArmor profiles.

2. Enforcing AppArmor via Namespace Labels

You don't write complex YAML for the policy itself. Instead, you label your namespace. Once labeled, the Kubernetes API will reject any Pod that doesn't include the security settings we discussed earlier.

Apply enforcement to a namespace:

```bash
# This command tells the kernel-level admission controller to 
# BLOCK any pod that doesn't meet the "Restricted" standard.
kubectl label --overwrite ns production-apps \
  pod-security.kubernetes.io/enforce=restricted
```
 3. What happens if a Pod is "Non-Compliant"?

If a developer tries to deploy a standard, unhardened Nginx Pod into that namespace, the Kubernetes API will reject it immediately:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
  namespace: production-apps
spec:
  containers:
  - name: nginx
    image: nginx
```
The Response (Rejected by PSA):

```bash
Error from server (Forbidden): pods "insecure-pod" is forbidden: violates PodSecurity "restricted:latest": 
- allowPrivilegeEscalation != false
- runAsNonRoot != true
- seccompProfile.type != "RuntimeDefault" or "Localhost"
- appArmorProfile.type != "RuntimeDefault" or "Localhost"
```

4. How PSA and AppArmor Work Together

While PSA enforces that your YAML must ask for an AppArmor profile, the Linux Kernel on the node is what actually executes that request.

**PSA (Admission Layer):** Checks the YAML. "Does this pod have an AppArmor profile? Yes? Proceed."

**Kubelet (Node Layer):** Receives the Pod. "The YAML asks for k8s-deny-write. Does my kernel have that profile loaded?"

**AppArmor (Kernel Layer):** "I have the profile. I will now monitor this container and block any forbidden write syscalls."

## Summary Table: Security Responsibility

| Layer | Responsibility | Component |
| :--- | :--- | :--- |
| **Cluster** | Enforcement (The "Law") | Pod Security Admission (PSA) |
| **Node** | Distribution (The "Carrier") | DaemonSet (Profile Loader) |
| **Kernel** | Execution (The "Enforcer") | AppArmor / SELinux / Seccomp |

## 9. How PSA enforce apparmor?
**Answer:**

Pod Security Admission (PSA) enforces AppArmor, as a "Security Border Control" that happens before any container actually starts.

PSA doesn't "run" AppArmor itself; instead, it enforces that your Pod's YAML includes the correct "orders" for the Linux Kernel.

**The Enforcement Workflow**

The enforcement happens in three distinct stages, moving from the Kubernetes API down to the physical CPU.

**1. The Validation Stage (API Server)**

When you run kubectl apply, the request hits the PSA Admission Controller. It looks at the namespace labels (e.g., enforce=restricted).

The Rule: Under the "Restricted" profile, a Pod must explicitly set an AppArmor profile.

The Action: If the Pod spec is missing the appArmorProfile field, or if it sets it to Unconfined, PSA rejects the request with a 403 Forbidden error. The Pod is never even created in the database.

**2. The Verification Stage (Kubelet)**

Once a Pod passes the API check, it is scheduled to a node. The Kubelet (the agent on the node) takes over.

The Check: Kubelet looks at the Pod spec and sees localhostProfile: k8s-deny-write.

The Validation: Kubelet checks the node's internal AppArmor registry. If that profile hasn't been loaded into the kernel (by your DaemonSet or manual script), the Kubelet will refuse to start the container and mark the Pod as CreateContainerConfigError.

**3. The Enforcement Stage (Kernel)**

If the profile exists, the Kubelet tells the Container Runtime (containerd/CRI-O) to launch the process with that profile attached.

The Action: The Linux Kernel now wraps the process in the AppArmor sandbox.

The Result: Every time the app tries to touch a file, the Kernel checks the "Law" (the profile). If the app tries to write to a forbidden path, the Kernel blocks the CPU instruction and logs a "DENIED" message.

## PSA Enforcement Levels for AppArmor

| PSA Level | AppArmor Requirement | Resulting Kernel State |
| :--- | :--- | :--- |
| **Privileged** | No requirements. | Often Unconfined (Kernel protection is OFF). |
| **Baseline** | Disallows explicit `Unconfined` setting. | Defaults to `RuntimeDefault` (Kernel protection is ON). |
| **Restricted** | Requires `RuntimeDefault` or a specific `Localhost` profile. | Maximum Isolation (Strict Kernel filtering). |

Summary of the "Chain of Command"

PSA says: "You must have a shield."

Kubelet says: "Let me check if I have that specific shield on this node."

AppArmor (Kernel) says: "I am now holding the shield and will block anything that tries to hit the container."