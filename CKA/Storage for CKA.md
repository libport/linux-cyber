# Storage for CKA
Kubernetes schedules pods as disposable units. Container write layers do not provide durable application storage. When a container crashes or a pod moves, Kubernetes can replace the workload, but local container changes can disappear. Stateless front ends can tolerate that behaviour. Stateful systems such as databases, queues and content stores need storage that survives pod replacement, node scheduling and routine updates.

Kubernetes storage separates compute from data through a small set of API objects. A volume gives containers in a pod access to a directory. Ephemeral volume types follow the pod lifecycle, so Kubernetes destroys them when the pod ends. Persistent storage exists outside that lifecycle and allows pods to reconnect to the same data after restart or replacement.
## Core storage objects
- PersistentVolume, or PV, represents a piece of storage in the cluster. An administrator can create it manually, or Kubernetes can create it through a StorageClass.
- PersistentVolumeClaim, or PVC, requests storage for an application. The claim specifies requirements such as size, access mode and storage class without exposing backend details.
- StorageClass defines a storage tier and the provisioner that creates matching PVs. It can encode disk type, performance tier, expansion support, reclaim policy and binding timing.

The control plane binds a PVC to a matching PV. A binding is exclusive and maps one claim to one volume. A PV can be Available, Bound, Released or Failed. A Released PV no longer has an active claim, but Kubernetes does not automatically make it safe for reuse when old data remains.
## Static provisioning
Static provisioning suits exams, labs and tightly controlled environments. The administrator creates a PV with capacity, an access mode, a backing volume such as `hostPath`, NFS or iSCSI, and a reclaim policy. The application then creates a PVC with compatible requirements. A PVC that sets `storageClassName: ""` disables dynamic provisioning and can bind only to a PV with no storage class.

A basic demonstration creates a 2 GiB PV and a matching PVC, mounts the claim into an NGINX pod at `/usr/share/nginx/html`, writes an `index.html` file, deletes the pod and recreates it. The file remains because the PV and PVC outlive the pod. `hostPath` works for a local Minikube exercise, but it ties data to a node and does not offer portable production storage.
## Access modes
Access modes describe how Kubernetes may mount a volume. They also help Kubernetes match PVs and PVCs. The storage backend determines the modes it can support.

| Mode | Meaning |
| --- | --- |
| ReadWriteOnce | The volume can mount read-write on one node. Multiple pods on that node may use it. |
| ReadOnlyMany | The volume can mount read-only on many nodes. |
| ReadWriteMany | The volume can mount read-write on many nodes. Distributed filesystems such as NFS commonly support it. |
| ReadWriteOncePod | The volume can mount read-write by one pod across the cluster. It is stable and supported for CSI volumes. |

Access modes do not replace application-level locking or filesystem permissions. Except for ReadWriteOncePod, they do not guarantee write protection after a volume has mounted. A ReadWriteOnce claim can fail when pods on different nodes attempt to mount the same backend at the same time.
## Reclaim policies
Reclaim policies define what happens after a PVC releases a PV.

- Retain keeps the PV and the underlying data. An administrator must inspect, recover, back up or clean the volume before reuse.
- Delete removes the PV and, when the plugin supports it, deletes the external storage asset. Dynamically provisioned PVs inherit the StorageClass reclaim policy, and a StorageClass defaults to Delete when the field is omitted.
- Recycle is deprecated. Dynamic provisioning has replaced its old scrub-and-reuse behaviour.

Retain protects important data, especially databases and audit stores. Delete fits temporary environments, scratch workloads and cases where automatic cleanup matters more than recovery.
## Dynamic provisioning and StorageClasses
Dynamic provisioning removes the need to create every PV by hand. A developer creates a PVC that names a StorageClass, or omits `storageClassName` to use the default class when one exists. Kubernetes reads the StorageClass, calls its provisioner and creates a new PV that matches the claim.

A StorageClass centres on these fields:
- `provisioner`, which identifies the volume plugin or CSI driver.
- `parameters`, which pass backend settings such as disk type or replication.
- `reclaimPolicy`, which applies to dynamically created PVs.
- `allowVolumeExpansion`, which allows users to grow a PVC when the driver supports expansion.
- `volumeBindingMode`, which controls when binding and provisioning occur.

`Immediate` provisions and binds storage as soon as the PVC appears. This reserves storage early, but it can ignore pod scheduling constraints for topology-bound backends. `WaitForFirstConsumer` waits until a pod uses the claim, then provisions storage that respects the pod's scheduling requirements. This improves placement for zonal, node-local and topology-aware storage. Pods should use selectors or affinity for that case, not `nodeName`, because `nodeName` bypasses the scheduler and can leave the PVC pending.
## Practical CKA patterns
CKA candidates should practise both static and dynamic flows until manifest edits feel routine.

For a static task, create a PV such as `task-pv` with `200Mi`, `ReadWriteOnce`, `Retain` and `hostPath: /opt/data`. Then create a PVC such as `task-pvc` in the required namespace with matching capacity, access mode and no storage class. Validate with `kubectl get pv,pvc` and confirm that both objects show Bound.

For a dynamic task, create a PVC such as `dynamic-pvc` with `50Mi` and the default StorageClass. Then create a pod such as `data-pod` using `busybox:latest`, mount the claim at `/data`, and provide a long-running command such as `sleep` so the container stays Running. If a pod crashes, `kubectl describe pod` and event messages usually reveal missing commands, failed mounts or scheduling conflicts.
## Exam focus
The current CKA exam is online, proctored and performance-based. Candidates solve command-line tasks in Kubernetes within 2 hours. The passing score is 66 percent. The exam currently uses Kubernetes v1.35, with updates aligned to recent Kubernetes minor releases.

Storage accounts for 10 percent of the CKA curriculum. The wider weighting is Cluster Architecture, Installation and Configuration at 25 percent, Workloads and Scheduling at 15 percent, Services and Networking at 20 percent, Storage at 10 percent and Troubleshooting at 30 percent.

Efficient candidates reduce typing and context errors. They set an alias such as `alias k=kubectl`, learn short names such as `po`, `pv`, `pvc` and `sc`, generate starter YAML with `--dry-run=client -o yaml`, and confirm the active context and namespace before each task. The official Kubernetes documentation and quick reference remain essential exam tools. Fast lookup, accurate YAML editing and repeated validation with `get`, `describe` and events matter more than memorising every manifest field.