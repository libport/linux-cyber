# Configuring and Managing Kubernetes Storage and Scheduling
## Persistent storage
Kubernetes treats containers as disposable. When a pod restarts, moves, or is replaced, data written to the container filesystem does not provide durable application state. Stateless front ends and APIs can usually recreate runtime state, but databases, upload services, log collectors, and batch jobs need storage that outlives pod lifecycle events.

PersistentVolumes (PVs) separate storage from compute. A PV represents cluster storage provisioned by an administrator or by a StorageClass. It can map to network storage, cloud disks, CSI drivers, local volumes, or a hostPath in a local test environment. A PersistentVolumeClaim (PVC) represents a workload's request for storage. The claim declares capacity, access mode, storage class, and optionally volume mode or selectors. Kubernetes binds a compatible PV to the PVC, and pods mount the claim as a volume.

This abstraction gives platform teams control over storage back ends while developers request storage through a stable API. It also avoids tying application data to a single container image or pod instance.

Important corrections for PVs and PVCs:
- ReadWriteOnce means a volume can mount read-write on one node, not one pod. Multiple pods on the same node may access it, depending on the volume and application behaviour.
- ReadWriteOncePod is the access mode that restricts read-write access to one pod across the cluster, when supported by CSI.
- hostPath suits local learning and single-node testing. It binds data to a node and does not provide portable production storage.
- For production, teams normally use CSI-backed storage, network filesystems, managed cloud disks, backup policies, and explicit reclaim policies.

A basic validation pattern creates a PV, creates a matching PVC, mounts that PVC into a pod, writes a file under the mount path, deletes the pod, recreates it, and checks that the file remains available. This verifies persistence across pod replacement, but it does not prove resilience across node failure unless the storage backend supports that outcome.
## StorageClasses and dynamic provisioning
Manual PV creation becomes unreliable as teams add services. StorageClasses define reusable storage profiles. They set the provisioner, parameters, reclaim policy, and volume binding mode. A PVC refers to a StorageClass name, and Kubernetes provisions or selects storage according to that class.

Dynamic provisioning needs a real provisioner, usually a CSI driver such as an AWS EBS CSI driver, a VMware vSphere CSI driver, or another storage vendor driver. The field `provisioner: kubernetes.io/no-provisioner` does not create storage dynamically. It identifies storage that Kubernetes cannot provision automatically, such as local volumes that require pre-created PVs. For local volumes, `WaitForFirstConsumer` is usually the safer binding mode because the scheduler can consider pod node constraints before binding a claim.

Reclaim policy controls what happens after a PVC releases a PV:
- Delete removes the associated storage asset when the provisioner supports deletion.
- Retain preserves the data for manual recovery or clean-up.
- Development clusters often prefer Delete.
- Production stateful systems often prefer Retain or explicit backup and deletion workflows.
## Scheduling workloads deliberately
The Kubernetes scheduler places new or unscheduled pods onto nodes. It first filters out nodes that cannot satisfy requirements, then scores feasible nodes and binds the pod to the highest-ranked option. It considers resource requests, node labels, affinity rules, taints, tolerations, volume topology, and policy constraints.

The scheduler does not continuously rebalance already-running pods by itself. It makes placement decisions when a pod needs scheduling. Rebalancing needs controllers, rollout actions, the descheduler, or operator intervention.
## Taints, tolerations, and node isolation
Taints let a node repel pods. Tolerations let selected pods schedule onto tainted nodes, but they do not force placement. A dedicated analytics node might receive `dedicated=analytics:NoSchedule`. Analytics pods can tolerate that taint. A `nodeSelector` or node affinity rule can then target nodes labelled `dedicated=analytics`.

This combination matters. The toleration grants permission, while the selector or affinity rule directs placement. Without a placement rule, the scheduler may choose any feasible node.

Taints and tolerations support:
- Workload isolation for analytics, security scanning, GPUs, or production services.
- Protection of specialised nodes from general workloads.
- Cleaner multi-tenant clusters when combined with labels and policies.
## Affinity, anti-affinity, and resources
Node affinity attracts pods to nodes with matching labels. It can express hard requirements or preferences. A pod can require `role=analytics` for data processing workloads, or prefer a low-cost node pool when performance allows.

Pod affinity and anti-affinity describe relationships between pods. Affinity co-locates related pods, such as an application and a local cache. Anti-affinity spreads replicas apart, which reduces the chance that one node failure takes down all instances of a service.

Resource requests and limits make scheduling safer. Requests describe the CPU and memory the scheduler reserves for placement. Limits cap runtime consumption. CPU uses cores or millicores, such as `250m`. Memory uses bytes or suffixes such as `Mi` or `Gi`. A realistic request helps the scheduler avoid overpacking nodes. A limit protects the node, but a tight CPU limit can throttle workloads and a tight memory limit can trigger termination.
## Availability controls
PodDisruptionBudgets (PDBs) protect replicated applications from too many voluntary disruptions at once. A PDB uses a selector and either `minAvailable` or `maxUnavailable`. For example, `minAvailable: 2` allows an eviction only when at least two selected pods remain healthy after that eviction.

PDBs apply to voluntary disruptions that use the Eviction API, such as many node drains. They do not prevent involuntary failures, and they do not replace application-level rollout settings such as Deployment update strategy. Cluster operators also need enough replicas, suitable probes, and healthy nodes for a PDB to help.

Topology spread constraints distribute matching pods across topology domains such as nodes, zones, or regions. `topologyKey: kubernetes.io/hostname` spreads pods across nodes. `maxSkew: 1` limits imbalance. `whenUnsatisfiable: DoNotSchedule` keeps the scheduler from breaking the rule when it cannot maintain the required spread.

PDBs and topology spread constraints complement each other. Topology spread reduces concentrated failure risk. PDBs reduce disruption during maintenance. Together, they improve availability for production workloads, especially when clusters have enough topology domains and enough replicas to satisfy both constraints.
## Operational pattern
A resilient Kubernetes platform should:
- Use PVs and PVCs for stateful workloads.
- Use StorageClasses to standardise provisioning, reclaim policy, and performance classes.
- Prefer CSI-backed storage for production durability and portability.
- Apply labels, taints, tolerations, affinity rules, and topology spread constraints to align placement with workload needs.
- Set realistic CPU and memory requests.
- Use limits carefully, based on observed behaviour.
- Use PDBs for replicated services that must stay available during maintenance.
- Test pod deletion, node drain, and topology failure scenarios before relying on scheduling policy.

Kubernetes becomes more predictable when storage and scheduling policies express real workload intent. Persistent storage preserves state beyond container lifecycles. Scheduling controls place pods where they can run reliably. Availability controls keep services running through change.
