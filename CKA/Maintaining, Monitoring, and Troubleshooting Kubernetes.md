# Maintaining, monitoring and troubleshooting Kubernetes
Production Kubernetes work requires repeatable deployment, safe maintenance, useful observability and disciplined fault diagnosis. A production-like cluster normally separates the control plane from several worker nodes, runs more than one replica of important workloads and spreads those replicas across nodes so that maintenance or failure does not remove all capacity at once.

A simple guestbook application can demonstrate these practices. The frontend serves the web interface, the backend handles requests and Redis stores application state. The application stays deliberately simple so that deployment strategy, monitoring, logging and troubleshooting remain the focus.
## Helm for repeatable application deployment
Kubernetes applications often need several manifests, including Deployments, Services, ConfigMaps, ServiceMonitors and PodDisruptionBudgets. Managing raw YAML across environments creates duplication and increases the risk of inconsistent changes.

Helm packages related Kubernetes resources into a chart. `Chart.yaml` stores chart metadata, `values.yaml` stores configurable settings such as image tags, replica counts and resource limits, and the `templates` directory stores parameterised Kubernetes manifests. Helm renders those templates with the supplied values and creates a release in the cluster.

Helm release history supports controlled upgrades and rollbacks. `helm upgrade` applies a new configuration or image version, and `helm rollback` creates a new release revision from an earlier revision. This is not source control, but it gives operators a useful deployment history inside the cluster.
## High availability during maintenance
High availability starts with replicas. Multiple frontend and backend pods handle the same workload behind Services that distribute traffic. If one pod crashes, the remaining pods continue serving requests whilst Kubernetes replaces the failed pod.

A PodDisruptionBudget, or PDB, protects availability during voluntary evictions handled through the Kubernetes Eviction API, such as node drains. A PDB sets `minAvailable` or `maxUnavailable`, which prevents an administrator from evicting too many selected pods at once. It does not protect against every disruption, and it does not stop direct deletion of unmanaged resources. It gives Kubernetes a clear availability constraint during planned maintenance.

Pod anti-affinity reduces placement risk. Without placement rules, the scheduler can place several replicas on the same node. Anti-affinity asks Kubernetes to spread matching pods across nodes, so a single worker outage removes only part of the application capacity.

Together, replicas, PDBs and anti-affinity support safer upgrades. Replicas provide redundancy, PDBs constrain voluntary evictions and anti-affinity spreads risk across the cluster.
## Cluster upgrade strategy
A safe Kubernetes upgrade begins with health checks. Administrators check node readiness, `kube-system` pod health, resource pressure, workload status and application availability. They back up etcd before changing the control plane because etcd stores cluster state, including Deployments, Services, ConfigMaps and other API objects.

Kubernetes version-skew rules drive the upgrade order. The control plane upgrades before worker nodes because supported kubelets can run at older minor versions than the API server during a controlled upgrade. Worker nodes then upgrade one at a time. Each worker should be cordoned to prevent new scheduling, drained to evict pods safely, upgraded, restarted, verified and uncordoned before the next worker changes.

A maintenance plan should record the exact upgrade sequence, validation commands, success criteria, rollback options, communications, milestones, risks and escalation contacts. If a worker upgrade fails, the operator can repair and retry or replace the node, especially in cloud environments. If the control plane becomes unstable, worker upgrades should stop until the API server and core components return to a healthy state.
## Metrics and observability
Observability gives operators evidence about system behaviour. Metrics record numerical measurements over time, such as CPU usage, memory use, request rate, latency and error rate. Logs record timestamped events and provide diagnostic context. Traces show request flow through distributed services. Metrics and logs usually deliver the first operational value in Kubernetes environments.

The Metrics Server provides recent CPU and memory data for nodes and pods. It enables commands such as `kubectl top nodes` and `kubectl top pods`. It suits current resource visibility and autoscaling inputs, but it does not provide long-term storage, advanced queries or application-specific reliability views.

Prometheus adds production-grade metrics. It scrapes configured targets, stores time-series data, supports PromQL and evaluates alerting rules. Grafana visualises metrics from Prometheus and other data sources through dashboards. Cluster dashboards show overall capacity and health. Node dashboards identify overloaded servers. Pod dashboards reveal restart counts, container states and resource use. Application dashboards track request rates, duration, errors and other business-relevant indicators.
## Centralised logging with Loki and Grafana Alloy
`kubectl logs` remains useful for local diagnosis, but it has production limits. It focuses on one pod at a time, loses context after pod churn, provides limited search and does not naturally connect log events with dashboard metrics.

Loki centralises log storage and query. Grafana Alloy can run as a DaemonSet on every node, collect container logs and send them to Loki. Structured JSON logs improve querying because each entry can include a timestamp, level, event type, pod name and contextual fields. Grafana can then display logs beside metrics, which helps operators correlate a latency spike, error burst or pod crash with the application events that occurred at the same time.
## Reliability targets and chaos validation
Service level indicators, or SLIs, measure reliability. Common SLIs include availability, latency and error rate. Availability measures the proportion of time a service can handle traffic. Latency measures response time, often at P95 or P99. Error rate measures failed requests as a percentage of total requests.

Service level objectives, or SLOs, set internal targets for those indicators, such as 99.5% availability, P95 latency below 200 ms or error rate below 0.5%. Service level agreements, or SLAs, expose selected commitments to customers and usually attach consequences, such as service credits.

Error budgets convert SLOs into operational decision-making. A 99.5% availability target allows 0.5% failure time. When the budget shrinks, teams should reduce release risk and prioritise reliability work. When the budget remains healthy, teams can accept more change.

Chaos engineering validates whether observability detects real failure modes. Injected latency should affect the latency SLO. Injected application errors should change the error-rate SLO and appear in logs. Crashing backend pods should reduce availability and produce matching pod events, metrics and logs. The goal is controlled proof that dashboards and alerts reveal customer-impacting failure.
## Troubleshooting workflow
Kubernetes troubleshooting should follow a steady sequence:
- Observe pod and node state with `kubectl get`.
- Describe the affected resource with `kubectl describe`.
- Read current or previous container logs with `kubectl logs`.
- Check cluster events, Services, Endpoints, labels and resource requests.
- Apply the smallest safe fix through configuration, manifests or Helm values.
- Verify that pods run, Services route traffic and the application behaves correctly.

CrashLoopBackOff usually means a container starts, fails and restarts repeatedly. `kubectl describe pod` shows restart counts and exit codes, while `kubectl logs --previous` often reveals the application error. In the guestbook scenario, an incorrect Redis hostname caused backend crashes. Restoring the correct service name fixed the loop.

ImagePullBackOff means Kubernetes cannot pull the configured image. The pod events normally expose an invalid repository, tag or credential issue. The scenario used a nonexistent Python tag, and redeploying with a valid image tag resolved the failure.

Pending pods indicate that the scheduler cannot place the pod. Common causes include insufficient CPU, insufficient memory, taints, node selectors or affinity rules. The scenario requested eight CPUs on nodes with far less allocatable capacity, so lowering the request allowed scheduling.

Networking failures can occur even when pods look healthy. A Service selector must match pod labels before Kubernetes creates usable Endpoints. In the scenario, the backend Service selected the wrong app label, so the frontend could not reach the backend. Correcting the selector restored traffic.

OOMKilled means the container exceeded its memory limit and the kernel terminated it. Logs may be absent if the process died abruptly, so `kubectl describe pod` and resource limits become critical. The scenario set the backend memory limit to 32 Mi, which was too low. Restoring realistic limits returned the pod to a healthy state.