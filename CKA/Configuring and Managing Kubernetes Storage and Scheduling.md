# Kubernetes Persistent Storage and Pod Scheduling
> [!NOTE]
> This guide explains how Kubernetes storage and scheduling work together, covering persistent volumes, dynamic provisioning, resource-based placement, affinity, taints, topology constraints, disruption budgets, and troubleshooting.

Kubernetes separates application state from Pod lifecycles and assigns new Pods to suitable nodes. Storage topology, resource requests, placement rules, and disruption controls must work together. A valid storage request can still leave a Pod pending when no node satisfies every scheduling constraint.
## Persistent storage
A container's writable layer is ephemeral and can disappear when Kubernetes replaces the container or Pod. An `emptyDir` volume survives a container crash but disappears when the Pod leaves its node. Applications that require durable state use a storage service designed for the required failure, recovery, and consistency model.

| Resource | Scope | Role |
| --- | --- | --- |
| PersistentVolume (PV) | Cluster | Represents provisioned storage and its capacity, class, access modes, topology, and reclaim policy |
| PersistentVolumeClaim (PVC) | Namespace | Requests storage and gives a Pod a namespaced reference to mount |
| StorageClass | Cluster | Defines a provisioner and implementation-specific storage policy |

The control plane binds one PVC exclusively to one compatible PV. Matching considers the storage class, requested capacity, access modes, volume mode, and any selector. A PV may provide more capacity than the claim requests because Kubernetes does not split one PV among claims. A `Bound` PVC confirms selection or provisioning, not successful attachment or mounting.

A Pod references the PVC under `spec.volumes` and mounts that Pod volume through `volumeMounts`. Data can survive Pod replacement while the claim and backing asset remain available. Persistence does not guarantee replication, backup, integrity, or cross-zone recovery.

`ReadWriteOnce` permits a read-write mount from one node, not one Pod. Several Pods on that node may use the volume. `ReadWriteOncePod` constrains a supported CSI volume to one Pod across the cluster. Backend capabilities determine support for `ReadOnlyMany`, `ReadWriteMany`, and other modes.

`hostPath` exposes a directory on one node. It can demonstrate mounting in a controlled single-node environment, but it does not provide portable multi-node persistence and can expose sensitive host data. Production clusters normally use supported Container Storage Interface drivers, network storage, or topology-aware local PVs.
### StorageClasses and binding
Dynamic provisioning requires a functioning provisioner, commonly a CSI driver. The provisioner creates a backing asset and PV for a matching PVC. By contrast, `provisioner: kubernetes.io/no-provisioner` explicitly disables automatic provisioning and supports pre-created local PVs. A static PV does not become dynamically provisioned because a StorageClass names it.

Omitting `storageClassName` allows the default StorageClass to apply. Setting `storageClassName: ""` disables dynamic provisioning for the claim and restricts binding to a PV with no class. Naming a class requests that class.

| Binding mode | Behaviour |
| --- | --- |
| `Immediate` | Binds or provisions when the PVC appears, before the scheduler knows the consuming Pod's placement |
| `WaitForFirstConsumer` | Delays binding or provisioning until scheduling can account for node constraints and storage topology |

`WaitForFirstConsumer` reduces the risk of selecting zone-bound storage that the Pod cannot use. A Pod should express placement through scheduler constraints rather than `spec.nodeName`, which bypasses the scheduler and can leave this type of claim pending.

The StorageClass reclaim policy commonly uses `Delete` or `Retain`. `Delete` can remove both the PV and backing asset when the driver supports deletion. `Retain` preserves the asset for manual recovery, sanitisation, or reuse. When a StorageClass omits `reclaimPolicy`, Kubernetes assigns `Delete` to dynamically provisioned PVs. `Recycle` remains deprecated and should not support new designs. Storage protection finalisers delay removal of a PVC that a Pod still uses and of a PV that remains bound. Operators should inspect the policy before deleting a PVC. Persistent storage also needs separate backup, restore, encryption, and retention controls.
## Scheduling decisions
The scheduler handles an unscheduled Pod in three broad stages. It filters out nodes that cannot satisfy hard requirements, scores the feasible nodes against preferences, and binds the Pod to the highest-scoring choice. A Pod remains pending when no feasible node exists.

The scheduler uses resource requests, not observed short-term utilisation, to test capacity. CPU uses cores or millicores, such as `250m`, while memory commonly uses binary units such as `256Mi`. Limits constrain runtime consumption. CPU limits can cause throttling, and memory use beyond a limit can trigger an out-of-memory kill. Limits do not replace realistic requests for placement.

The scheduler does not continuously relocate running Pods to improve balance. `IgnoredDuringExecution` means that a Pod continues running when a relevant node label later changes. Controllers can create replacement Pods after failure, scaling, or rollout, but each replacement receives a new scheduling decision.
### Placement controls
| Control | Effect |
| --- | --- |
| `nodeSelector` | Requires all specified node labels |
| Required node affinity | Filters out nodes that do not satisfy the expression |
| Preferred node affinity | Adds a weighted scoring preference without blocking placement |
| Pod affinity | Places a Pod near matching Pods within a labelled topology domain |
| Pod anti-affinity | Avoids or forbids placement near matching Pods, depending on rule strength |
| Taints and tolerations | Repel Pods from nodes unless their tolerations match |
| Topology spread constraints | Control replica skew across labelled domains such as nodes or zones |

Hard affinity and anti-affinity rules can leave Pods pending. Preferred rules preserve scheduling flexibility. Inter-Pod rules also add scheduler work and depend on consistent labels and selectors.

A toleration permits placement on a tainted node but does not attract the Pod to that node. A dedicated node pool therefore commonly combines a taint with a trusted node label and required node affinity. `NoSchedule` blocks new placement, `PreferNoSchedule` creates a soft avoidance, and `NoExecute` also evicts running Pods without a matching toleration. `tolerationSeconds` can delay that eviction.
## Availability during disruption
A PodDisruptionBudget selects Pods by label and defines either `minAvailable` or `maxUnavailable`. The Eviction API uses the budget to limit voluntary disruptions such as `kubectl drain`. The PDB status reports current health, desired health, expected Pods, and allowed disruptions.

With three desired replicas and `minAvailable: 2`, one voluntary eviction is possible only while all three selected Pods are healthy. If one Pod is already unavailable, the budget normally allows no additional disruption. A percentage or `maxUnavailable` can adapt more naturally when a controller changes replica count.

A PDB does not guarantee that replicas remain available. It cannot prevent node failure, direct Pod deletion, workload deletion, or every other involuntary event. Deployment rolling updates use the Deployment strategy's `maxUnavailable` and `maxSurge` settings. A strict PDB can block maintenance indefinitely when too few selected Pods are healthy.

Topology spread constraints compare matching Pods across eligible domains identified by `topologyKey`. `maxSkew` sets the allowed imbalance. `DoNotSchedule` makes the constraint hard, while `ScheduleAnyway` scores placements that reduce skew. The label selector must identify the intended replica set, and relevant nodes need consistent topology labels. Four replicas can normally occupy two eligible nodes as two and two when capacity and other constraints allow it.

PDBs and topology spread constraints address different risks. A PDB limits selected voluntary evictions, while spread constraints influence new placement across failure domains. Neither control creates capacity, repairs an unhealthy application, or guarantees service availability.
## Operational checks
1. Inspect Pod events and scheduler messages before changing placement rules.
2. Compare requests with node allocatable capacity, taints, labels, affinity, and topology.
3. Inspect the PVC, PV, StorageClass, CSI components, and attachment events for storage failures.
4. Test drain, node loss, rollout, and zone failure separately because each follows different controls.
5. Monitor pending Pods, skew, disruption allowance, storage attachment, and recovery time.