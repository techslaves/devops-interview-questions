What is a StorageClass?

Answer:
Defines:

Provisioner (CSI driver)

Volume type

Parameters (IOPS, encryption)

Reclaim policy

Binding mode

Difference between static and dynamic provisioning?

Static → Admin pre-creates PVs

Dynamic → StorageClass + CSI auto-provision

 Dynamic provisioning is standard in production

What happens when a PVC is deleted?

Answer:
Depends on the PV reclaim policy:

Delete → PV & storage deleted

Retain → PV remains (manual cleanup)

Recycle → deprecated

What is volumeBindingMode: WaitForFirstConsumer?

Answer:
Delays volume provisioning until:

Pod is scheduled

Node & AZ are known

Why do Pods get stuck in Pending with PVCs?

Top reasons:

No matching StorageClass

Insufficient storage quota

AZ / node affinity conflict

CSI driver not installed

Volume binding mode mismatch

What is CSI and why is it important?

Answer:
CSI (Container Storage Interface):

Standardizes storage plugins

Decouples storage from Kubernetes core

Enables cloud & vendor innovation

10. Explain Access Modes (RWO, RWX, ROX)

Interview trap answer:

Access mode is capability, not guarantee

Depends on storage backend

Example:

EBS → RWO only

EFS → RWX

### What is volumeClaimTemplate?
A volumeClaimTemplate is a template used by a StatefulSet to automatically create a unique PVC per Pod, ensuring stable storage that follows the Pod across restarts.

```yaml
volumeClaimTemplates:
- metadata:
    name: data
  spec:
    accessModes: ["ReadWriteOnce"]
    storageClassName: gp3
    resources:
      requests:
        storage: 10Gi
```

### What is the difference between volumeClaimTemplate vs normal PVC?
| Feature         | volumeClaimTemplate | Normal PVC       |
| --------------- | ------------------- | ---------------- |
| Who creates PVC | StatefulSet         | User             |
| PVC per Pod     | Yes                 | Shared or single |
| Pod identity    | Bound to ordinal    | Not guaranteed   |
| Typical use     | Databases           | Shared storage   |

### What is the difference between persistent volume vs emepheral vs projected volume?
| Volume Type           | Purpose               | Data Lifetime         | Backed By        |
| --------------------- | --------------------- | --------------------- | ---------------- |
| **Persistent Volume** | Long-term state       | Survives Pod restarts | External storage |
| **Ephemeral Volume**  | Temporary data        | Pod lifetime          | Node / memory    |
| **Projected Volume**  | Inject config/secrets | Pod lifetime          | API objects      |

### What is Projected volumes?
A special ephemeral volume that combines multiple sources into a single mount.

Sources it can project

Secrets

ConfigMaps

Downward API

ServiceAccount tokens

Example:

```yaml
volumes:
- name: config
  projected:
    sources:
    - configMap:
        name: app-config
    - secret:
        name: app-secret
```