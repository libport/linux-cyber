# Storage for CKA
Kubernetes separates storage consumption from storage implementation. Applications request capacity and access characteristics, while cluster components and storage drivers provision, attach, mount, resize, and reclaim the backing resource. This separation allows Pod replacement without tying application data to a container's writable layer.
## Storage lifecycles and resources
A container's writable layer is ephemeral and unsuitable for durable application state. An `emptyDir` volume lets containers in one Pod share data and survives container crashes, but Kubernetes deletes it when the Pod leaves its node. Persistent storage uses a lifecycle independent of an individual Pod.

| Resource | Scope | Function |
| --- | --- | --- |
| PersistentVolume (PV) | Cluster | Describes provisioned storage, capacity, access modes, class, volume mode, and reclaim policy |
| PersistentVolumeClaim (PVC) | Namespace | Requests storage and provides the reference that a Pod mounts |
| StorageClass | Cluster | Defines a provisioner and implementation-specific policy for dynamic provisioning |

The control plane binds one PVC exclusively to one suitable PV. A static match considers storage class, requested capacity, access modes, volume mode, and any selector. The PV can exceed the requested capacity because Kubernetes does not divide one PV among claims. A claim remains `Pending` when no matching PV exists.

A `Bound` claim confirms selection or provisioning, but it does not prove that a node can attach and mount the volume. A Pod can remain `Pending` or `ContainerCreating` because of topology conflicts, multi-attach restrictions, missing CSI node components, credentials, permissions, or file-system faults. Events distinguish these stages and identify the responsible component.

A PV commonly moves through `Available`, `Bound`, `Released`, and `Failed` phases. Deleting and recreating a Pod does not delete its PVC, so a replacement can mount the same bound storage. Protection finalisers delay deletion of a PVC that a Pod still uses and of a PV that remains bound.
### Using and reserving claims
A Pod declares a PVC under `spec.volumes` and mounts the named Pod volume through a container's `volumeMounts`. The Pod and PVC must occupy the same namespace, while the PV remains cluster-scoped. A mount path exposes the volume inside the container and does not identify the storage location on a node or provider.

A StatefulSet can create one PVC for each replica through `volumeClaimTemplates`. Each replacement Pod then reuses the claim associated with its stable ordinal. This pattern preserves per-replica storage identity, but application consistency still depends on the storage system, shutdown behaviour, replication design, and recovery process.

Static provisioning can target a specific PV by setting `spec.volumeName` in the PVC. An administrator can reserve that PV with `claimRef` so another claim does not bind first. Pre-binding still requires compatible storage class, access modes, and requested capacity. A PV with `Retain` also needs careful data sanitisation and a new binding definition before reuse.

Modern clusters normally integrate external storage through Container Storage Interface drivers. Several legacy in-tree cloud drivers, including the original AWS EBS and Google Persistent Disk plugins, no longer ship in current Kubernetes releases. Driver capabilities and supported Kubernetes versions therefore require verification.

`hostPath` mounts a directory from one node. It does not provide portable, multi-node persistence and can expose sensitive host files or privileged sockets. It suits controlled single-node tests or narrowly governed system workloads, not a general stateful application design. A supported CSI-backed service, network file system, or topology-aware local PV provides a safer production basis.
## Access modes and volume modes
The storage backend determines which access modes a volume supports.

| Access mode | Meaning |
| --- | --- |
| `ReadWriteOnce` (RWO) | One node can mount the volume read-write. Several Pods on that node may use it |
| `ReadOnlyMany` (ROX) | Many nodes can mount the volume read-only |
| `ReadWriteMany` (RWX) | Many nodes can mount the volume read-write |
| `ReadWriteOncePod` (RWOP) | One Pod across the cluster can mount the volume read-write. This mode requires CSI support |

Access modes guide binding and attachment. Apart from the RWOP constraint, they do not independently enforce write protection after a mount. Workloads that require a read-only view should also configure the volume mount and underlying storage permissions accordingly. A volume uses only one access mode at a time.

`volumeMode: Filesystem` is the default and mounts a formatted file system at a directory. `volumeMode: Block` presents a raw block device, so the application must manage it correctly. The PV and PVC must request compatible modes.
## Static and dynamic provisioning
Static provisioning requires an administrator to create a PV before a claim binds. Dynamic provisioning creates a PV for a PVC through the requested StorageClass. A StorageClass commonly defines a CSI `provisioner`, driver-specific `parameters`, `reclaimPolicy`, `allowVolumeExpansion`, and `volumeBindingMode`.

Storage class selection requires precise YAML:
- Omitting `storageClassName` allows the cluster's default StorageClass to apply.
- Setting `storageClassName: ""` disables dynamic provisioning for that claim and restricts it to PVs with no class.
- Naming a class requests only PVs from that class or triggers its provisioner.

Omission does not guarantee static provisioning. Kubernetes can assign a default class later to an unclassified PVC when the cluster gains a default StorageClass.

`Immediate` binding, the default, provisions or binds storage when the PVC appears. This can select storage in a zone that conflicts with the eventual Pod placement. `WaitForFirstConsumer` delays binding or provisioning until the scheduler evaluates a Pod that uses the claim. It can then consider resource requests, node selectors, affinity, anti-affinity, taints, tolerations, and storage topology.

A Pod using `WaitForFirstConsumer` should express placement through scheduler constraints rather than `spec.nodeName`. Directly setting `nodeName` bypasses the scheduler and can leave the PVC pending.
## Reclaiming and expanding storage
The PV reclaim policy controls the result after PVC deletion:

| Policy | Result |
| --- | --- |
| `Retain` | Leaves the PV in `Released` state and preserves the backing asset for manual recovery, sanitisation, or reuse |
| `Delete` | Removes the PV and, when the driver supports it, permanently deletes the backing storage asset |

Dynamically provisioned PVs inherit their StorageClass policy. A StorageClass that omits `reclaimPolicy` defaults to `Delete`, so operators should inspect the policy before deleting a claim. `Recycle` remains deprecated and should not support new designs.

Expansion requires `allowVolumeExpansion: true` and driver support. The operator increases the PVC request, and Kubernetes resizes the existing backing volume. Editing PV capacity directly can prevent the resize workflow. Kubernetes supports growth, not shrinking. File-system expansion can complete during Pod start or online when the driver and file system support it.

Persistent storage does not replace backups. Reclaim policy, snapshots, replication, backup, restore testing, encryption, and retention address different failure and recovery needs.
## Operational checks
1. Inspect available StorageClasses, their defaults, provisioners, binding modes, reclaim policies, and expansion settings.
2. Confirm that the PVC requests the intended class, capacity, access mode, and volume mode.
3. Compare a pending claim with PV capacity, class, modes, selectors, node affinity, and topology.
4. Inspect Pod and PVC events for provisioning, scheduling, attachment, mounting, permission, and resize failures.
5. Verify CSI controller and node components before changing workload manifests.
6. Confirm data recovery and deletion consequences before removing a PVC or PV.