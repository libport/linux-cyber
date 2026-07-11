## Workloads and scheduling for the CKA
The Workloads and Scheduling domain covers 15% of the Certified Kubernetes Administrator exam. It tests how Kubernetes runs applications, keeps pods healthy, rolls out change, injects configuration, places workloads on suitable nodes and scales applications during demand spikes.

Kubernetes administrators rarely create standalone pods for production workloads. A standalone pod has no controller to replace it after failure. Controllers watch cluster state and reconcile it with the desired state declared in manifests.
## Workload controllers
- Deployment runs stateless applications such as web servers and APIs. It manages ReplicaSets, supports rolling updates and provides the normal default for CKA workload tasks.
- StatefulSet runs stateful applications that need stable network identity and persistent storage. A restarted database pod keeps its identity and storage claim.
- DaemonSet runs one pod on each eligible node. It suits node-level agents such as log collectors, monitoring agents and networking components.
- Job runs finite work until a required number of successful completions occurs, then stops.
- CronJob creates Jobs on a schedule, such as a nightly backup or recurring report.

Labels and selectors connect controllers to their pods. Changing a controller-managed pod label can break that relationship, which may cause the controller to create a replacement and leave the edited pod orphaned. Administrators should treat controller selectors as stable after creation.

Jobs use `completions` to define how many successful runs the task needs and `parallelism` to cap how many pods may run at the same time. A CronJob wraps the same pattern in a schedule. Failed Job pods can retry according to the Job policy, while completed pods provide a clear record of task success. This distinction helps administrators choose a long-running controller for services and a finite controller for batch work.
## Deployments and rollouts
A Deployment update changes the pod template, commonly by changing an image tag, environment variable or label. Kubernetes creates a new ReplicaSet for the new template and keeps older ReplicaSets for rollback history. The default `revisionHistoryLimit` is 10.

A recreate strategy terminates existing pods before starting replacement pods. It avoids mixed versions but causes downtime. A rolling update replaces pods gradually and keeps the application available when readiness and capacity allow it.

Rolling updates use two key controls:
- `maxSurge` defines how many extra pods Kubernetes may run above the desired replica count during the update.
- `maxUnavailable` defines how many desired pods may be unavailable during the update.

The default for both fields is 25%. Increasing them can speed up a rollout but can reduce safety. Decreasing them can slow the rollout while protecting availability.

Administrators inspect rollout progress with `kubectl rollout status` and inspect revision history with `kubectl rollout history`. They verify the active image with `kubectl describe` or structured output, then confirm that old pods have terminated and new pods have become Ready. A failed image pull or broken pod template can stall a rollout. `kubectl rollout undo` restores the previous stable pod template by creating a new active revision from an older ReplicaSet.
## Configuration and secrets
Cloud native applications should separate configuration from container images. The same image should run across development, test and production while the environment supplies values such as domains, feature flags and credentials. This approach shortens build pipelines, reduces registry storage and prevents small environment changes from forcing new image builds.

ConfigMaps store non-sensitive configuration. Secrets store sensitive data such as passwords, tokens, SSH private keys and TLS certificates. Both can reach containers as environment variables or mounted files.

Environment variables work well for small values read during process start. A pod restart is required after those values change. Mounted ConfigMaps and Secrets expose data as files and can receive eventual updates from the kubelet. Mounting a single key with `subPath` prevents those updates from reaching the container.

Secrets improve workflow separation, but they are not encryption by themselves. Secret values are Base64 encoded and Kubernetes stores Secret resources unencrypted in etcd by default unless the cluster enables encryption at rest. Anyone with API or etcd access may retrieve or modify them, and anyone who can create a pod in a namespace can indirectly read Secrets in that namespace.

Administrators should enable encryption at rest, restrict `get`, `watch` and `list` access with RBAC, mount Secrets only into the containers that need them, and consider external secret stores for higher-security environments.
## Pod admission, requests and limits
Kubernetes uses resource requests and limits to control pod placement and runtime behaviour. Requests also improve cluster planning because the scheduler reserves capacity according to declared needs rather than momentary process usage. Limits protect neighbouring workloads from noisy containers, but overly tight limits can degrade performance or trigger restarts.

- Requests reserve scheduling capacity. The scheduler places a pod only on a node whose allocatable capacity can satisfy the pod's requested resources after accounting for existing requests.
- Limits cap runtime use. CPU above the limit gets throttled. Memory above the limit can trigger termination with an `OOMKilled` status.

These values affect Quality of Service classes:
- Guaranteed applies when every container has CPU and memory requests and limits set, and each request equals its matching limit.
- Burstable applies when at least one CPU or memory request or limit exists, but the pod does not meet Guaranteed criteria.
- BestEffort applies when no container defines CPU or memory requests or limits.

During memory pressure, Kubernetes evicts lower QoS pods before higher QoS pods. BestEffort pods face eviction first, while Guaranteed pods receive the strongest protection.
## Scheduling controls
The Kubernetes scheduler matches unscheduled pods to nodes. It uses requested resources, node conditions and scheduling rules. Administrators can guide placement when workloads need specific hardware, isolation or high availability.

`nodeSelector` provides the simplest hard placement rule. A pod with `nodeSelector: {disk: ssd}` can run only on nodes with the exact `disk=ssd` label. If no node matches, the pod remains Pending.

Taints and tolerations create an exclusion model. A node taint repels pods that do not tolerate it. `NoSchedule` blocks new pods, `PreferNoSchedule` asks the scheduler to avoid the node where possible, and `NoExecute` can evict existing pods without a matching toleration. A toleration permits scheduling onto a tainted node but does not force placement there.

Node affinity extends `nodeSelector` with richer operators and hard or preferred rules. Preferred rules let the scheduler try a placement without blocking the pod when the ideal node is unavailable.

Pod affinity and anti-affinity place pods according to other pods already running in the topology. Anti-affinity supports high availability by spreading replicas across nodes so one node failure does not remove every replica of an application. Affinity can also keep tightly coupled components near each other, for example when a cache and application benefit from node-local communication. Administrators should select the weakest rule that satisfies the design, since hard rules can leave pods Pending when the cluster lacks matching capacity.
## Pod Disruption Budgets
Cluster maintenance often uses voluntary disruption. Examples include node drains, upgrades and planned node pool changes. Pod Disruption Budgets protect applications from losing too many available replicas during these planned operations.

A PDB defines an availability floor with `minAvailable` or a disruption ceiling with `maxUnavailable`. For a three-replica application, `minAvailable: 2` allows one voluntary eviction but blocks another until Kubernetes schedules and readies a replacement pod.

A strict PDB can block `kubectl drain`. Setting `minAvailable` equal to the replica count or `maxUnavailable` to zero allows no voluntary evictions. PDBs do not protect against every disruption. Deleting a Deployment or deleting pods directly bypasses PDB protection, and unplanned failures can still remove pods.
## Workload autoscaling
Horizontal Pod Autoscaler scales replicas for scalable targets such as Deployments and StatefulSets. It compares observed metrics, such as CPU or memory utilisation, with a target value and adjusts replica count within configured minimum and maximum bounds. HPA commonly needs resource requests and a metrics pipeline such as Metrics Server.

Vertical Pod Autoscaler changes CPU and memory requests rather than replica count. Cluster Autoscaler changes node count when pending pods cannot schedule or nodes remain underused. Both usually require separate installation and provider integration, so CKA work focuses mainly on HPA.

HPA uses a control loop. If the target CPU utilisation is 50% and pods average 100%, HPA recommends more replicas to reduce average utilisation. When load falls, HPA avoids rapid scale-down through a stabilisation window. The default scale-down stabilisation window is 300 seconds. Administrators should test HPA under load, confirm that metrics become available and set sensible minimum replicas so the application can absorb sudden demand before new pods finish starting.
## Exam practice guidance
CKA tasks reward speed and precision. Candidates should switch to the context shown in each task before changing resources. Aliasing `kubectl` to `k` and using `--dry-run=client -o yaml` to generate manifests saves time and reduces syntax errors.

Candidates should learn short resource names such as `cm`, `ds`, `deploy`, `po`, `svc` and `hpa`. They should use `kubectl get -o wide`, `kubectl describe`, events and rollout commands to verify outcomes.

The exam permits official documentation. Effective preparation includes practising with the Kubernetes docs, locating examples quickly and moving past time-consuming tasks for later review. The Workloads and Scheduling domain rewards practical fluency with controllers, rollout safety, configuration injection, resource controls, scheduling rules, disruption protection and HPA.