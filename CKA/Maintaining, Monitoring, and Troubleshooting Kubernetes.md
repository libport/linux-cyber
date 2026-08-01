# Maintaining, Monitoring, and Troubleshooting Kubernetes
## Helm release management
Helm packages related Kubernetes resources as a chart. `Chart.yaml` defines chart metadata, including the chart version and optional application version. `values.yaml` supplies defaults, additional values files override them, and files under `templates/` generate Kubernetes manifests. Helm renders those templates and submits the resulting resources to the API server. It does not rewrite the chart's source YAML.

A release records the installed chart, configuration, and revision. `helm upgrade --install` applies a repeatable installation or upgrade, `helm history` lists revisions, and `helm rollback` restores the configuration from an earlier revision as a new revision. Helm history supports operational recovery, but version control must retain chart source, reviewed values, and change history.

Production workflows should run `helm lint`, inspect `helm template` output, validate it against the target API, protect secrets, and pin chart and image versions. Application source code should reside in built images rather than ConfigMaps.
## Availability and maintenance
Replicas reduce the effect of a Pod failure only when remaining Pods have enough capacity and readiness probes keep unhealthy instances out of Service endpoints. A single database replica still creates a stateful failure point even when front-end and API tiers have several replicas.

A PodDisruptionBudget, or PDB, limits concurrent voluntary evictions through the Eviction API. `kubectl drain` respects a PDB and waits when an eviction would breach `minAvailable` or `maxUnavailable`. A PDB does not block involuntary failures, direct Pod or controller deletion, Deployment updates, or scaling by the workload controller. It therefore supports maintenance but cannot guarantee uninterrupted service.

Pod anti-affinity can separate replicas across nodes or zones. A preferred rule influences scheduling without guaranteeing placement. A required rule enforces separation but can leave Pods Pending when the cluster lacks suitable capacity. Topology spread constraints provide more direct control over balance across failure domains. Maintenance plans must combine replicas, accurate probes, spare capacity, disruption budgets, placement rules, graceful termination, and tested dependency resilience.
## kubeadm cluster upgrades
An upgrade plan should confirm supported version skew, deprecated APIs, release-specific changes, cluster health, available capacity, and rollback or replacement procedures. It should also record responsibilities, success criteria, verification steps, maintenance timing, and escalation contacts. An etcd snapshot requires validation and secure storage. A complete recovery plan also preserves relevant certificates, configuration, manifests, and external dependencies.

A kubeadm-managed cluster upgrades its control plane before its workers. A highly available control plane upgrades one node at a time. The first control-plane node runs `kubeadm upgrade plan` and `kubeadm upgrade apply`. Additional control-plane nodes run `kubeadm upgrade node`. Each worker follows a controlled sequence: cordon and drain the node, upgrade `kubeadm`, run `kubeadm upgrade node`, upgrade and restart the kubelet, verify health, and uncordon the node. Administrators should follow the documentation for the exact source and target versions rather than reuse old package commands.

Each phase should stop when control-plane health, system Pods, nodes, workloads, or service indicators fail acceptance checks. Redundant workloads can preserve service during a node upgrade, but capacity limits, PDB conflicts, storage attachment, topology rules, or application faults can still cause disruption.
## Monitoring and observability
Observability combines metrics, logs, and traces. Metrics quantify behaviour over time, logs describe discrete events, and traces follow requests across services. Useful telemetry should connect infrastructure signals with user-visible outcomes.

Metrics Server aggregates recent CPU and memory measurements for the resource metrics API. It supports autoscaling and `kubectl top` spot checks. It does not provide a historical monitoring system, and live utilisation from `kubectl top` does not represent the scheduler's remaining allocatable capacity.

Prometheus scrapes configured metric endpoints, stores time series, evaluates PromQL queries, and runs recording and alerting rules. Alertmanager groups, suppresses, and routes notifications from firing alerts. Grafana queries Prometheus and other data sources to visualise cluster, node, Pod, and application signals. A production design must also address retention, persistent storage, high availability, access control, alert ownership, and capacity for the monitoring system itself.

Kubernetes keeps container logs on nodes, where rotation, Pod deletion, or node loss can remove them. `kubectl logs --previous` can retrieve the preceding container instance while the kubelet still retains it, and label selectors can retrieve logs from several Pods. Cluster-level logging gives logs a lifecycle independent of Pods and nodes.

Grafana Alloy can discover and ship Kubernetes logs to Loki. Loki indexes stream labels and stores compressed log chunks. Structured JSON logs improve filtering when they use consistent fields, timestamps, severity, service identity, and request correlation. Low-cardinality labels should identify stable dimensions such as cluster, namespace, and service. Retention, object storage, tenancy, encryption, and access policies require explicit configuration.
## Reliability objectives and chaos engineering
A service level indicator, or SLI, measures user-relevant performance over a defined window. Many SLIs use the ratio of good events to valid events. Examples include successful requests, requests completed below a latency threshold, and correct responses. A service level objective, or SLO, sets the internal target. A service level agreement, or SLA, expresses an external commitment and may attach consequences.

An SLO of 99.5% permits an error budget of 0.5% bad events during the measurement window. A documented error-budget policy can slow risky changes and prioritise reliability work when consumption becomes excessive.

Chaos engineering tests a measurable steady-state hypothesis by introducing realistic failure conditions. Authorised experiments should start with a narrow blast radius, defined safeguards, live monitoring, abort criteria, and a restoration plan. Latency, error, and process-failure injection can reveal weaknesses, but uncontrolled experiments can create an outage rather than useful evidence.
## Systematic troubleshooting
Diagnosis should proceed from symptom to evidence: identify affected objects, inspect detailed state and events, collect current and previous logs, test the relevant dependency, correct the owning manifest or configuration, and verify both Kubernetes status and application health. Structured output and timestamps provide safer evidence than table text alone.

The first pass should preserve evidence before a restart or redeployment changes it:

```bash
kubectl get pods -A -o wide
kubectl describe pod POD_NAME -n NAMESPACE
kubectl get events -n NAMESPACE --sort-by=.metadata.creationTimestamp
kubectl logs POD_NAME -n NAMESPACE -c CONTAINER_NAME --previous
kubectl get pod POD_NAME -n NAMESPACE -o yaml
```

An administrator should then follow the ownership chain from Pod to ReplicaSet, Deployment, StatefulSet, Job, or DaemonSet. Node conditions, kubelet logs, control-plane component logs, and audit records become relevant when several unrelated workloads fail together. A change that clears the displayed status does not prove recovery. Verification should exercise the affected request path, confirm ready endpoints, compare current telemetry with the prior baseline, and watch long enough to cover the original failure pattern. Incident records should retain timestamps, commands, relevant output, the root cause, the corrective change, and any follow-up control.

| Symptom | Evidence and likely causes | Corrective direction |
| --- | --- | --- |
| `CrashLoopBackOff` | Inspect container state, exit code, events, and `kubectl logs --previous`. Common causes include invalid commands, missing configuration, failed dependencies, and probe-driven restarts. | Repair the owning workload or Helm values, then confirm a stable restart count and successful readiness. |
| `ImagePullBackOff` | Pod events identify a missing tag, denied registry access, absent pull credentials, rate limiting, or network failure. | Correct the immutable image reference, registry authentication, or connectivity. |
| `Pending` | Scheduler events expose insufficient requested CPU or memory, affinity conflicts, taints, unbound storage, quota, or topology limits. The scheduler compares requests with node allocatable resources and existing reservations. | Reduce justified requests, add capacity, or correct the specific scheduling constraint. |
| Service unavailable | Compare the Service selector with Pod labels, then inspect ports, readiness, EndpointSlices, DNS, and NetworkPolicy. A selector mismatch produces no controller-managed endpoints. | Align selectors and labels, correct port mapping or policy, and verify traffic from a diagnostic Pod. |
| `OOMKilled` | Container `lastState` commonly records `reason: OOMKilled` and exit code 137. Logs may end abruptly. Metrics, heap data, and limits distinguish an undersized limit from a leak or burst. | Fix memory behaviour or set a measured request and limit that the cluster can support. |

Repeated Pod deletion can hide evidence because the controller recreates the same faulty template. The durable fix belongs in the Deployment, StatefulSet, Job, chart values, Secret, ConfigMap, Service, or policy that owns the failed state. Verification should include readiness, endpoints, logs, metrics, SLO impact, and recurrence over an appropriate observation period.