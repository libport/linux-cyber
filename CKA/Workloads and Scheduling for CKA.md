# Workloads and Scheduling for CKA
> [!NOTE]
> This guide covers Kubernetes workloads and scheduling, including controllers, rollouts and rollbacks, configuration, resource management, Pod placement, disruption controls, autoscaling, and operational verification.

The CKA curriculum assigns 15% of the examination to workloads and scheduling. This domain covers application controllers, rolling updates and rollbacks, configuration, autoscaling, resource management, Pod placement, and disruption control.
## Workload controllers
A Kubernetes controller continually compares actual state with desired state and acts to reconcile any difference. A standalone Pod may restart its containers on the same node, but no higher-level controller replaces the Pod after deletion or node loss. Most applications therefore use a workload controller.

| Controller | Primary use | Behaviour |
| --- | --- | --- |
| Deployment | Usually stateless applications | Manages ReplicaSets and replaces interchangeable Pods through controlled updates |
| StatefulSet | Stateful applications | Preserves each Pod's stable identity, ordering, and association with persistent storage |
| DaemonSet | Node-local agents | Runs one Pod on every eligible node, or on a selected subset, and follows changes in eligible nodes |
| Job | Finite work | Creates Pods until the required number of successful completions occurs |
| CronJob | Scheduled finite work | Creates Jobs according to a cron schedule |

Job `completions` defines the required successes, while `parallelism` limits concurrent Pods. Completed Jobs and their Pods normally remain available for status and log inspection until deletion or TTL cleanup. CronJob tasks should tolerate duplicate or missed executions because scheduling is not exact.

A CronJob's concurrency policy controls whether a new run may overlap the preceding run. History limits and `ttlSecondsAfterFinished` prevent finished Jobs from accumulating. Job programs should handle retries safely because node loss, eviction, and controller recovery can cause work to run again.

Selectors connect controllers to their Pods. A controller replaces a Pod that no longer matches its selector, so administrators must keep controller selectors and Pod-template labels aligned. Deleting a controller normally removes its dependent Pods through ownership references.

Self-healing operates at two layers. The kubelet follows a Pod's restart policy when a container exits. A workload controller creates a replacement when a managed Pod disappears or becomes unsustainable on its node. Readiness probes keep an unready replacement out of Service endpoints, while liveness and startup probes can trigger container restarts when configured correctly.
## Deployment lifecycle
A Deployment creates a new ReplicaSet when its Pod template changes. Scaling the replica count alone does not create a revision. The Deployment retains old ReplicaSets for rollback, with `revisionHistoryLimit` defaulting to 10 old revisions.

`RollingUpdate` is the default strategy. It gradually scales up the new ReplicaSet and scales down the old one. `maxSurge` sets the number of extra Pods allowed above the desired count and rounds percentages up. `maxUnavailable` sets the permitted shortfall in available Pods and rounds percentages down. Both default to 25%. Readiness probes and realistic resource requests determine whether a rollout can preserve service.

`Recreate` terminates the old revision's Pods before creating the new revision's Pods during an update. It suits applications that cannot run two versions concurrently, but it introduces downtime.

The principal rollout commands are:
- `kubectl set image deployment/<name> <container>=<image>`
- `kubectl rollout status deployment/<name>`
- `kubectl rollout history deployment/<name>`
- `kubectl rollout undo deployment/<name> --to-revision=<number>`

A rollback copies an earlier Pod template into a new active revision. It does not reverse changes outside the Pod template. `kubectl describe`, Pod status, events, and logs reveal failures such as `ImagePullBackOff`, failed probes, insufficient quota, or an exceeded progress deadline.

Deployment selectors are immutable in `apps/v1`, and they must match the Pod-template labels. Image tags should identify known artefacts rather than changing content in place. A successful `kubectl set image` response confirms only that the API accepted the change. `kubectl rollout status` and application health checks establish whether the new revision became available.
## Configuration and Secrets
ConfigMaps separate non-confidential configuration from container images. Secrets hold small confidential values such as passwords, tokens, TLS material, SSH keys, and image-pull credentials. Applications can consume either object through environment variables or mounted volumes.

Environment variables receive values when a container starts. A changed ConfigMap or Secret therefore requires a Pod rollout before the process receives new environment values. Mounted ConfigMap and Secret volumes update with eventual consistency after kubelet synchronisation. A `subPath` mount does not receive automated updates. File projection also does not force the application to reload its configuration, so the application must watch, poll, or otherwise reload changed files.

Base64 encoding does not encrypt Secret data. Kubernetes stores Secrets unencrypted in etcd by default unless the cluster enables encryption at rest. Strong protection requires least-privilege RBAC, restricted Pod-creation rights, encryption at rest, access only from the containers that need each Secret, and, where appropriate, an external secret store. Administrators should avoid exposing Secret values through logs, command histories, or broad environment dumps.
## Requests, limits, admission, and QoS
API admission and scheduling perform different functions. Admission controls such as ResourceQuota and LimitRange can reject a Pod or supply defaults. The scheduler then compares Pod resource requests with each node's allocatable capacity. It does not use limits or momentary utilisation to decide placement.

A request establishes the amount considered for scheduling and resource allocation. A container may use more than its request when capacity remains available. A CPU limit causes kernel throttling. A memory limit acts reactively, and the kernel may terminate a process with an out-of-memory kill after excessive allocation.

Kubernetes assigns a Pod one of three QoS classes:
- `Guaranteed` requires every application and init container to specify positive CPU and memory requests and limits, with each request equal to its corresponding limit.
- `Burstable` applies when the Pod does not qualify as `Guaranteed`, but at least one container has a CPU or memory request or limit.
- `BestEffort` applies when no container has a CPU or memory request or limit.

QoS influences node-pressure eviction order, but it is not Pod priority. Kubernetes considers resource use relative to requests, Pod priority, and other conditions during eviction. `kubectl top` reports current usage only when the cluster provides the resource Metrics API.
## Scheduling and availability
The scheduler filters out nodes that fail a Pod's hard requirements, scores the remaining nodes, and binds the Pod to a suitable node. Several controls modify this process:

| Control | Effect |
| --- | --- |
| `nodeSelector` | Requires every specified node label to match |
| Required node affinity | Expresses hard label rules with richer operators |
| Preferred node affinity | Adds a weighted preference without blocking placement elsewhere |
| Pod affinity | Co-locates Pods according to labels and a topology domain |
| Pod anti-affinity | Separates Pods according to labels and a topology domain |
| Taints and tolerations | Repel Pods unless they tolerate the relevant taint |

`IgnoredDuringExecution` affinity rules govern scheduling only. A later node-label change does not evict an already running Pod. Hard pod anti-affinity can spread replicas across nodes or zones, but all nodes need consistent topology labels, and the cluster needs enough eligible topology domains.

Taint effects have distinct meanings. `NoSchedule` blocks new Pods without a matching toleration. `PreferNoSchedule` asks the scheduler to avoid the node when possible. `NoExecute` also evicts existing Pods that lack a matching toleration. A toleration grants permission to use a tainted node but does not attract the Pod there. A node selector or affinity rule must provide that attraction when the workload requires placement on the tainted node.

A PodDisruptionBudget limits simultaneous unavailability from voluntary evictions that use the Eviction API. `kubectl drain` respects a PDB and retries an eviction that would breach `minAvailable` or `maxUnavailable`. A PDB cannot prevent node failure, direct Pod or controller deletion, or a workload controller's rolling update. Those disruptions may count against the budget even though the PDB does not control them. Replicas, topology spread, readiness, update strategy, and PDBs must work together to provide availability.

An unschedulable Pod remains `Pending`. `kubectl describe pod` and namespace events expose causes such as unmatched labels, untolerated taints, insufficient requested resources, volume topology, or hard anti-affinity. `kubectl get pods -o wide` confirms the chosen node after binding. Administrators should remove temporary node labels and taints carefully because those changes can alter future placement without relocating Pods already running under ignored-during-execution rules.
## Workload autoscaling
A HorizontalPodAutoscaler periodically changes the replica count of a scalable workload such as a Deployment or StatefulSet. It cannot scale a DaemonSet. The stable `autoscaling/v2` API supports CPU, memory, custom, and external metrics.

For percentage-based CPU or memory utilisation, the HPA compares observed use with the relevant resource request. Missing requests leave utilisation undefined for that metric. The cluster must expose the corresponding metrics API, commonly through Metrics Server for resource metrics.

The core calculation is `desired replicas = ceil(current replicas x current metric / target metric)`. Tolerance suppresses small fluctuations, readiness handling dampens misleading startup data, and scaling policies limit the rate of change. The default scale-down stabilisation window considers recommendations from the previous 300 seconds, which reduces repeated removal and recreation of replicas.

An administrator can create a basic HPA with `kubectl autoscale deployment/<name> --cpu=<percentage> --min=<n> --max=<n>` and inspect it with `kubectl get hpa`. An `unknown` metric commonly indicates unavailable metrics or missing requests. Load tests must verify both scale-up and scale-down behaviour, while workload readiness must prevent new replicas from receiving traffic before they can serve it.

Vertical Pod Autoscaler is an optional add-on that adjusts resource assignments. Node autoscaling adds or removes cluster capacity when workload placement requires it. These mechanisms solve different constraints and require coordinated minimums, maximums, requests, and capacity.
## Operational checks
Before changing a workload, an administrator confirms the current context and namespace, inspects the live object, and records its existing state. Client-side dry runs can generate editable YAML with `--dry-run=client -o yaml`. After each change, status, events, logs, selected nodes, resource use, and controller ownership confirm the result. Temporary aliases can reduce typing, but explicit context checks prevent changes to the wrong cluster or namespace.