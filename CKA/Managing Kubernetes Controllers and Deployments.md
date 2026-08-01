# Managing Kubernetes Controllers and Deployments
> [!NOTE]
> This guide explains how Kubernetes controllers reconcile desired state and how to configure, update, troubleshoot, and safely operate Deployments, ReplicaSets, DaemonSets, Jobs, CronJobs, and StatefulSets.
## Reconciliation and controller architecture
Kubernetes uses declarative configuration. An object records a desired state in the API, while one or more controllers compare that state with the cluster's observed state and act to reduce the difference. Each controller runs a reconciliation loop. The loop watches relevant objects, evaluates their current condition, and submits changes through the API server. Reconciliation continues after the first successful action, so the cluster can respond to later failures, configuration changes, and scaling requests.

A controller does not normally issue instructions directly to every node. It reads and writes API objects, then other components respond to those objects. For example, a Deployment controller creates or updates a ReplicaSet. The ReplicaSet controller creates Pods. The scheduler assigns unscheduled Pods to nodes, and each node's kubelet starts the assigned containers. This separation allows several specialised control loops to cooperate through a common API.

Most built-in controllers run within `kube-controller-manager`. A highly available control plane can run several instances of this component. Leader election selects an active controller-manager instance for controller loops that require a single leader. This design differs from the claim that every cluster runs exactly one controller-manager process.

The cloud controller manager runs control loops that depend on a cloud provider, such as node lifecycle, routes, and external load balancers. Separating this component from the core controller manager lets a cloud integration evolve without changing the core Kubernetes control plane. Clusters without a supported cloud integration may not run it.

Controllers provide self-healing within the limits of their specifications. They can replace a failed Pod, restore a replica count, or create work for a missed state transition. They cannot repair an application that reports false health, preserve data that lacks durable storage, or guarantee capacity that the cluster does not possess. Effective reconciliation therefore depends on accurate selectors, suitable health probes, sufficient resources, and correct workload design.
## Choosing a workload controller
Each workload controller encodes a different lifecycle. A controller should match the identity, placement, duration, and scheduling needs of the workload.

| Controller | Primary purpose | Core behaviour |
| --- | --- | --- |
| Deployment | Stateless, continuously running applications | Manages ReplicaSets, supports scaling, and performs controlled rollouts and rollbacks |
| ReplicaSet | A stable count of interchangeable Pods | Creates or deletes Pods to reach the requested replica count |
| DaemonSet | One Pod on every eligible node, or on a selected group of nodes | Adds Pods as eligible nodes appear and removes Pods when they cease to qualify |
| Job | Finite work that must reach one or more successful completions | Retries failed execution within configured limits and records completion or failure |
| CronJob | Jobs that begin on a recurring schedule | Creates Jobs according to a cron expression and a concurrency policy |
| StatefulSet | Applications that need stable identity, ordered operations, or persistent storage | Gives each Pod a stable ordinal, network identity, and associated storage claim |

A Deployment normally provides the right abstraction for replicated stateless services. Administrators should not manage its ReplicaSets directly because the Deployment owns their lifecycle and revision history. A bare ReplicaSet remains useful for understanding reconciliation and for rare cases that do not require Deployment rollouts.
## Deployments and ReplicaSets
### Deployment specification
A Deployment uses the `apps/v1` API. Its specification defines a selector, a Pod template, and usually a replica count. The selector must match the labels in the Pod template. Kubernetes rejects a new `apps/v1` Deployment when these fields conflict. The replica count is optional and defaults to one, although an explicit value often makes operational intent clearer.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app.kubernetes.io/name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: web
  template:
    metadata:
      labels:
        app.kubernetes.io/name: web
    spec:
      containers:
      - name: web
        image: nginx:1.27
        ports:
        - name: http
          containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: http
          initialDelaySeconds: 2
          periodSeconds: 5
```

The top-level metadata identifies the Deployment. The template metadata labels each Pod that the Deployment ultimately creates. The selector determines which Pods the controller treats as members of the workload. The template specification defines containers, images, ports, probes, resources, volumes, security settings, and other Pod-level properties.

Selectors form part of a controller's identity. A Deployment selector is immutable after creation. An administrator who needs a different selector normally creates a new Deployment and migrates traffic deliberately. Broad or overlapping selectors can cause one controller to adopt Pods that another process created, so each controller should use distinctive labels.

`matchLabels` expresses equality requirements. `matchExpressions` adds set-based operators such as `In`, `NotIn`, `Exists`, and `DoesNotExist`. Requirements within one selector combine with logical AND. A ReplicaSet can therefore select across several labels and expressions. The older ReplicationController also supports multiple equality-based requirements, but it does not support the ReplicaSet's set-based selector syntax.
### Ownership and reconciliation
Creating a Deployment produces an ownership chain:

```text
Deployment -> ReplicaSet -> Pods
```

The Deployment controller calculates a hash from the Pod template and places it in the generated ReplicaSet name and the `pod-template-hash` label. A changed Pod template produces a new hash and therefore a new ReplicaSet. The label keeps revisions distinct and prevents their selectors from overlapping.

Owner references record the controlling relationship. A Pod created by a ReplicaSet points to that ReplicaSet as its controller. The ReplicaSet points to the Deployment. Kubernetes garbage collection can use these references when deleting dependants, subject to the selected deletion propagation policy.

The ReplicaSet controller continuously compares the number of matching Pods with `.spec.replicas`. If too few Pods match, it creates replacements from the Pod template. If too many match, it selects excess Pods for deletion. The controller counts matching Pods rather than relying only on objects it originally created. It may adopt compatible orphan Pods that match its selector and lack another controller owner.

Changing or removing a selected label from one Pod can isolate that Pod from its ReplicaSet. The ReplicaSet then detects a shortfall and creates another Pod. The isolated Pod continues to run until an administrator or another controller deletes it. This behaviour demonstrates why direct Pod edits can create unexpected capacity and why workload-level changes should target the owning controller.

A controller never moves an existing Pod to another node. Pods remain bound to their assigned nodes for their lifetime. When a node or Pod fails, the relevant controller creates a different Pod, and the scheduler chooses a node for that replacement. The new Pod receives a new name and unique identifier, even when it runs the same application configuration.
### Deletion and ownership boundaries
Deleting a controller normally starts cascading garbage collection of the objects it owns. Deleting a Deployment therefore removes its owned ReplicaSets and their Pods under the usual background propagation policy. A Service remains because it is a separate object and does not belong to the Deployment.

Propagation policy changes the sequence. Background deletion removes the owner from the API and lets garbage collection remove dependants. Foreground deletion keeps the owner in a terminating state until blocking dependants disappear. Orphan propagation removes the owner while preserving dependants and clearing the relevant owner references.

```bash
kubectl delete deployment web --cascade=background
kubectl delete deployment web --cascade=foreground
kubectl delete deployment web --cascade=orphan
```

Orphaning a ReplicaSet can preserve running Pods during a deliberate ownership transfer, but it also removes the Deployment's rollout control. A new controller with an overlapping selector may adopt compatible orphans. Operations should plan label changes and ownership transfer so two controllers do not compete for the same Pods.

Finalizers can delay deletion while a controller performs required cleanup. An object with a deletion timestamp and remaining finalizers is terminating rather than fully deleted. Removing an unfamiliar finalizer by force can strand cloud resources, storage, network configuration, or other external state. Diagnosis should identify the responsible controller and its failure before any manual removal.

Deletion and scaling do not have identical storage effects across controllers. Deployment Pods usually reference shared or externally defined volumes. StatefulSet Pods often own stable claims through their ordinal identities, and those claims commonly survive Pod or StatefulSet deletion. An administrator must inspect the retention policy, PersistentVolume reclaim policy, and backup status before removing stateful resources.

`kubectl delete pod` tests replacement behaviour but does not update desired state. A ReplicaSet, Deployment, DaemonSet, or StatefulSet can recreate the Pod immediately. To stop the workload, operations should scale or delete the owner, suspend the relevant batch controller, or change node eligibility according to the intended lifecycle.
### Declarative and imperative management
Declarative management stores desired state in a manifest and applies it through the API:

```bash
kubectl apply -f web-deployment.yaml
```

The command creates the object when it does not exist and patches the live object when it does. Version-controlled manifests provide review, repeatability, and an audit trail. Server-side apply can add managed-field ownership when several actors manage different fields.

Imperative commands make focused changes without first editing a file. They remain useful for examination, incident response, and time-limited operational actions:

```bash
kubectl create deployment web --image=nginx:1.27 --replicas=3
kubectl scale deployment/web --replicas=5
kubectl set image deployment/web web=nginx:1.28
```

An imperative change can drift from the repository that should define production state. A subsequent declarative apply may reverse that change. Operational practice should therefore record intended changes in source control and reconcile emergency changes back into the maintained manifests.

The obsolete `--record` workflow should not support change history. Current practice can associate a change with a source-control revision, deployment system record, or ticket. An optional `kubernetes.io/change-cause` annotation can provide a human-readable note, but it should complement rather than replace a durable audit system.
### Inspection and diagnosis
Several views expose different parts of the ownership chain and current state:

```bash
kubectl get deployment web
kubectl get replicaset -l app.kubernetes.io/name=web
kubectl get pods -l app.kubernetes.io/name=web -o wide
kubectl describe deployment web
kubectl get deployment web -o yaml
kubectl get pod POD_NAME -o jsonpath='{.metadata.ownerReferences}'
```

`kubectl get` gives a compact status view. `kubectl describe` combines fields, conditions, and recent events. YAML or JSON output exposes the complete stored object. Events can explain scheduling failures, image pull errors, probe failures, quota rejection, and other transitions, but event retention is limited. Long-term operations require central logging and monitoring.

Deployment status does not reduce to one universal phase. An administrator should inspect replica counts and conditions such as `Available`, `Progressing`, and `ReplicaFailure`. Desired, current, updated, available, and unavailable replica counts answer different questions. A Deployment can report progress while some new Pods remain unready, and it can have all replicas created before any become available.
## Updating Deployments
### What triggers a rollout
A Deployment starts a new revision when `.spec.template` changes. Common triggers include a new image, an environment variable, a label, a resource request, a volume, or a probe. Changing `.spec.replicas` alone scales the current ReplicaSet and does not create a new revision.

An image update can target one named container:

```bash
kubectl set image deployment/web web=nginx:1.28
kubectl rollout status deployment/web
```

The Pod template should use immutable image digests when release reproducibility is essential. A mutable tag can resolve to different image content at different times, which weakens rollback and audit guarantees. A registry policy and admission controls can enforce approved image sources and signatures.
### RollingUpdate and Recreate
`RollingUpdate` is the default Deployment strategy. The controller creates a new ReplicaSet, increases its replica count, and decreases the old ReplicaSet in controlled steps. It continues until the new ReplicaSet supplies the desired replicas and the old one reaches zero. Old ReplicaSets normally remain with zero replicas so the Deployment can preserve revision history.

Two fields control rollout capacity:

| Field | Meaning | Default | Percentage rounding |
| --- | --- | --- | --- |
| `maxUnavailable` | Maximum number of desired replicas that may be unavailable during the update | 25% | Down |
| `maxSurge` | Maximum number of Pods that may exist above the desired replica count | 25% | Up |

For a Deployment with 20 replicas, `maxUnavailable: 25%` allows five unavailable replicas, not four. `maxSurge: 25%` allows five additional Pods. At least one of the two values must be greater than zero. Integer values set exact limits, while percentages scale with the desired replica count.

A rollout does not guarantee uninterrupted service by itself. Availability also depends on enough replicas, spare cluster capacity, accurate readiness probes, graceful termination, suitable Service selectors, application compatibility, and external dependencies. A single-replica Deployment can experience interruption if the cluster cannot run a surge Pod or if the new Pod becomes ready only after the old one stops.

`Recreate` scales old Pods down before it creates new ones. This strategy suits workloads that cannot run old and new versions together, but it introduces an intentional service gap unless another layer supplies capacity. Manually deleting old Pods can still produce a brief overlap, so `Recreate` should not serve as a strict mutual-exclusion mechanism.
### Readiness and availability
The Deployment controller uses readiness to determine whether new Pods can receive traffic and whether a rollout can proceed within its availability limits. A readiness probe should test the conditions required to serve requests. It should not duplicate every deep dependency check if a temporary dependency failure would remove all replicas and worsen an incident.

A liveness probe answers a different question. It asks whether the kubelet should restart the container. A startup probe protects slow-starting containers from premature liveness and readiness failures. Misconfigured probes can create restart loops or route traffic too early.

`minReadySeconds` specifies how long a newly ready Pod must remain ready without a container crash before Kubernetes counts it as available. The default is zero. A positive value can expose early failures before the controller removes more old replicas, though it also lengthens the rollout.

`progressDeadlineSeconds` defines how long the controller waits for progress before it reports a `ProgressDeadlineExceeded` condition. The default is 600 seconds. This condition reports a stalled rollout. Kubernetes does not automatically roll the Deployment back, and the controller can continue retrying. Monitoring or a delivery system must decide whether to alert, pause, fix, or roll back.

Typical causes of a stalled rollout include an invalid image name, missing registry credentials, insufficient CPU or memory, an unsatisfied affinity rule, a taint without a toleration, a failing admission policy, a missing ConfigMap or Secret, and a readiness probe that never succeeds.
### Rollout status and history
The following commands inspect a rollout and its revisions:

```bash
kubectl rollout status deployment/web
kubectl rollout history deployment/web
kubectl rollout history deployment/web --revision=3
kubectl describe deployment web
```

`kubectl rollout status` watches the latest rollout by default and returns a non-zero status if its watch encounters a failure or timeout. A script should set an explicit timeout that fits the release process. A separate monitoring system should continue checking service health after Kubernetes reports completion.

The Deployment stores historical revisions as old ReplicaSets. `.spec.revisionHistoryLimit` controls how many old ReplicaSets Kubernetes retains for rollback and defaults to 10. A value of zero removes rollback history after cleanup. History consumes API objects rather than active application replicas because retained old ReplicaSets normally stay scaled to zero.

The change-cause annotation can add context to rollout history:

```bash
kubectl annotate deployment/web \
  kubernetes.io/change-cause='Release 2026-08-01, image nginx:1.28' \
  --overwrite
```
### Pause, resume, rollback, and restart
Pausing a Deployment lets several Pod-template changes accumulate without starting an intermediate rollout for each change:

```bash
kubectl rollout pause deployment/web
kubectl set image deployment/web web=nginx:1.28
kubectl set resources deployment/web --containers=web --limits=cpu=500m,memory=256Mi
kubectl rollout resume deployment/web
```

While paused, the Deployment controller does not begin a new rollout for Pod-template changes. Resume causes the controller to reconcile the accumulated template as one revision. A paused Deployment cannot roll back until it resumes.

A rollback copies the selected old Pod template into the Deployment as a new active revision. It does not turn time backwards or restore cluster-wide dependencies, data, ConfigMaps, Secrets, Services, or database schemas. Compatibility planning must cover those resources separately.

```bash
kubectl rollout undo deployment/web
kubectl rollout undo deployment/web --to-revision=3
kubectl rollout status deployment/web
```

The first command selects the preceding revision. The second selects a retained revision explicitly. Before rollback, an administrator should inspect the revision and confirm that its image, configuration references, and data expectations remain valid.

A restart leaves the Pod template's application settings intact but patches the template annotation so that the Deployment creates a new revision and replaces all Pods through its configured strategy:

```bash
kubectl rollout restart deployment/web
kubectl rollout status deployment/web
```

A restart can refresh Pods after an external dependency or mounted configuration has changed. It does not fix an invalid template, increase capacity, or guarantee that an application reloads data from an external service.
### Scaling
The Deployment's replica count can change declaratively or imperatively:

```bash
kubectl scale deployment/web --replicas=8
```

Scaling changes the current desired count without creating a rollout revision. Scaling down deletes Pods and can terminate in-flight work. Pod disruption budgets constrain some voluntary disruptions, but they do not prevent every controller-driven deletion. Applications should handle termination signals, expose readiness accurately, and use a sufficient termination grace period.

A HorizontalPodAutoscaler can manage a Deployment's scale subresource from resource or custom metrics. Manual changes to `.spec.replicas` can then conflict with autoscaling and may be overwritten during the next reconciliation. A Git-managed manifest that continually reapplies a fixed replica count can create the same conflict. Operational ownership of the replica field should remain clear.
## Node failure and Pod replacement
The node controller monitors node health. When a node becomes unreachable or not ready, Kubernetes represents that condition with node taints. Ordinary Pods receive default tolerations for `node.kubernetes.io/not-ready` and `node.kubernetes.io/unreachable`, commonly with `tolerationSeconds: 300`. This grace period can prevent unnecessary replacement during a short network interruption.

After the relevant tolerance expires, the control plane can evict affected Pods and their workload controllers can create replacements. The exact outcome depends on the node condition, taints, tolerations, Pod type, and controller behaviour. Static Pods and some DaemonSet Pods follow different rules. An administrator should not treat an old `pod-eviction-timeout` explanation as the complete current model.

Replacement Pods do not schedule onto a node that the scheduler regards as unavailable. If no healthy node satisfies resource, affinity, topology, volume, and taint constraints, the replacement remains Pending. The controller has restored the desired object count, but the cluster has not restored service capacity.

Persistent storage introduces additional recovery constraints. A volume may remain attached to the failed node, support only one attachment, or reside in a topology zone unavailable to other nodes. Stateful applications require storage and failure-domain planning beyond controller replica counts.
## Diagnosing failed reconciliation
Troubleshooting should follow the ownership chain from the highest controller to the Pod rather than begin with repeated Pod deletion. Each layer can reach its own desired state while a lower layer remains blocked. A Deployment can create the correct ReplicaSet, the ReplicaSet can create the correct number of Pod objects, and every Pod can still remain Pending because no node can accept it.

| Observation | Likely layer | Useful evidence |
| --- | --- | --- |
| No new ReplicaSet appears after an intended update | Deployment specification or paused rollout | Deployment generation, observed generation, template, pause state, conditions, and events |
| A new ReplicaSet exists but has too few Pods | ReplicaSet reconciliation or admission | ReplicaSet replica counts, owner references, namespace quota, admission events, and API errors |
| Pods remain Pending | Scheduling or storage | Pod scheduling events, requests, node capacity, affinity, taints, topology, and volume binding |
| Pods show `ImagePullBackOff` | Image reference or registry access | Image name, tag or digest, pull secret, registry reachability, and kubelet events |
| Containers restart repeatedly | Application, configuration, or health checks | Previous container logs, exit code, probe results, limits, and mounted configuration |
| Pods run but never become Ready | Readiness or dependency health | Readiness probe output, endpoint membership, application logs, and network policy |
| Updated Pods become Ready but clients fail | Service path or application compatibility | Service selector, EndpointSlices, ingress or gateway state, protocol compatibility, and client telemetry |

`.metadata.generation` increases when a desired-state field changes. A controller records the latest generation it has observed in `.status.observedGeneration`. A lower observed value can indicate that the controller has not processed the newest specification. Equal values show observation, not success, so conditions and replica counts still require inspection.

Events provide concise reasons attached to objects. Sorting namespace events by timestamp can reveal admission rejection, failed scheduling, mount errors, image failures, and probe problems:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod POD_NAME
kubectl logs POD_NAME -c CONTAINER_NAME --previous
```

`--previous` retrieves logs from the preceding terminated container instance when the kubelet still retains them. It helps diagnose crash loops in which the current container has only just restarted. Central logging remains necessary because Pod replacement, node loss, log rotation, and retention limits can remove local evidence.

EndpointSlice inspection distinguishes a controller problem from a traffic-selection problem. A healthy Pod that lacks the Service's labels will not enter its EndpointSlices. A matching but unready Pod normally remains unavailable for ordinary Service traffic. The following commands compare the intended selector with actual endpoints:

```bash
kubectl get service web -o yaml
kubectl get endpointslice -l kubernetes.io/service-name=web
kubectl get pods --show-labels
```

Admission controllers can reject or mutate resources before storage. Policy failures often appear in the command response and events rather than in a workload controller condition. Server-side dry run can validate API acceptance without persisting the object:

```bash
kubectl apply --server-side --dry-run=server -f web-deployment.yaml
```

A successful dry run does not prove that the scheduler can place the Pods or that the application will start. It confirms API validation, admission, and compatible field ownership at that moment.

Repeated manual deletion can conceal the original symptom and consume evidence. An administrator should first capture YAML, conditions, events, logs, revision history, and relevant metrics. Remediation should change the owning template, capacity, policy, or dependency that caused the failure. The controller can then perform the replacement under its normal safety limits.
## DaemonSets
A DaemonSet ensures that every eligible node, or every node selected by its scheduling rules, runs a copy of a Pod. When an eligible node joins, the controller creates a Pod for it. When a node ceases to qualify, the controller removes the associated Pod. Typical uses include log collectors, node monitoring agents, storage daemons, networking components, and security agents.

Eligibility does not mean worker nodes only. Node selectors, affinity, taints, and tolerations determine placement. Control-plane nodes commonly carry a `NoSchedule` taint, so a DaemonSet needs a matching toleration before it can run there. The DaemonSet controller also adds tolerations for some node conditions so node-local agents can remain present when ordinary workloads leave.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
  namespace: operations
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: node-agent
  template:
    metadata:
      labels:
        app.kubernetes.io/name: node-agent
    spec:
      nodeSelector:
        node-role.example.com/observed: "true"
      containers:
      - name: agent
        image: example.invalid/agent:2.4.0
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
```

As with a Deployment, the selector must match the template labels and becomes immutable after creation. A DaemonSet normally omits `replicas` because eligible nodes determine the count. Desired, current, ready, available, and unavailable figures can differ while nodes join, leave, or fail health checks.

Many clusters deploy `kube-proxy` as a DaemonSet, but Kubernetes does not require every networking implementation to do so. Some environments replace it or implement service networking by other means.
### DaemonSet updates
`RollingUpdate` replaces DaemonSet Pods gradually and is the default strategy. `maxUnavailable` defaults to one. Current DaemonSet rolling updates can also use `maxSurge`, which defaults to zero. The two values cannot both be zero. Surge support depends on the running Kubernetes version and workload constraints, so cluster policy should define suitable values.

`OnDelete` updates the template but leaves existing Pods running. New Pods receive the new template only after an administrator or another event deletes the old Pods. This strategy allows manual sequencing, but it also lets a fleet remain indefinitely mixed if operations stop partway through.

```bash
kubectl set image daemonset/node-agent -n operations agent=example.invalid/agent:2.5.0
kubectl rollout status daemonset/node-agent -n operations
kubectl rollout history daemonset/node-agent -n operations
kubectl rollout undo daemonset/node-agent -n operations
```

Before a broad node-agent update, operations should test compatibility with each node operating system, architecture, kernel, container runtime, and security policy. A faulty privileged DaemonSet can affect every node, so staged node selection or a canary DaemonSet can reduce blast radius.
## Jobs
A Job manages finite work and tracks successful completion. It creates one or more Pods, retries failures within policy, and records a terminal `Complete` or `Failed` condition. Suitable tasks include data processing, schema migration, report generation, backups, and administrative batch work. A continuously running service belongs in a Deployment or StatefulSet instead.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: report
spec:
  backoffLimit: 4
  activeDeadlineSeconds: 1800
  ttlSecondsAfterFinished: 86400
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: report
        image: example.invalid/report:3.1.0
        args: ["generate", "--date", "2026-08-01"]
```

Job Pod templates allow `restartPolicy: Never` or `OnFailure`. With `OnFailure`, the kubelet can restart a failed container in the same Pod. With `Never`, a failed container causes the Pod to fail, and the Job controller can create another Pod. `Never` does not make the Job fail immediately. The Job still applies its retry and deadline policy.

`backoffLimit` bounds retries after failure. Failure accounting depends on Pod failures and, with `OnFailure`, container restarts in active Pods. Kubernetes uses exponential backoff between retries. Once failure accounting reaches the configured limit, the Job receives a `Failed` condition. Administrators should not infer a fixed total Pod count from the limit because restart policy, controller timing, and newer per-index policies affect the observed count.

`activeDeadlineSeconds` limits the Job's active duration and takes precedence over the backoff limit. When the deadline expires, Kubernetes terminates running Pods and marks the Job failed. A suspended Job handles this timer specially, so operations should consult the active cluster version before relying on pause duration semantics.

`ttlSecondsAfterFinished` makes a finished Job eligible for automatic cleanup after the specified delay. Without it, the Job and its Pods remain until another retention mechanism removes them. Retaining selected objects can aid diagnosis, but unlimited completed Jobs consume API storage and clutter operational views.
### Completion and parallelism
Without explicit `completions` or `parallelism`, a Job normally runs one Pod to one successful completion. `completions` sets the required number of successful Pods. `parallelism` limits how many Pods may run at once.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-convert
spec:
  completions: 12
  parallelism: 3
  completionMode: Indexed
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: converter
        image: example.invalid/converter:5.0.0
```

An indexed Job gives each completion a stable index, allowing each Pod to process a distinct partition. A non-indexed Job requires cooperating workers or an external work queue when tasks must remain unique. Kubernetes can occasionally start more than one Pod for the same work after node, kubelet, or controller disruptions. Batch operations should therefore be idempotent or protect side effects with transactional coordination.

Parallelism is a maximum rather than a promise. Quotas, scheduling constraints, resource shortages, or controller throttling can reduce actual concurrency. Lowering parallelism does not necessarily terminate existing Pods immediately. Suspending a Job stops new Pod creation and can remove active Pods, so applications should tolerate interrupted execution.
### Observing and cleaning up Jobs
```bash
kubectl get job report
kubectl describe job report
kubectl get pods -l job-name=report
kubectl logs job/report
kubectl wait --for=condition=complete job/report --timeout=30m
kubectl delete job report
```

The Job condition type is `Complete`, while a successfully terminated Pod has phase `Succeeded`. Some `kubectl` tables display a human-readable status such as `Completed`, but scripts should query structured fields rather than parse display text.

Logs belong to Pods and containers, not to the Job object. `kubectl logs job/report` selects a Pod for convenience. Parallel or retried Jobs can have several Pods, so diagnosis may require listing each Pod, inspecting attempts, and collecting logs centrally. Deleting a Job normally deletes its dependent Pods according to garbage-collection policy.
## CronJobs
A CronJob creates Jobs on a repeating schedule. It uses `apiVersion: batch/v1`. The removed `batch/v1beta1` API should not appear in current manifests.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-report
spec:
  schedule: "0 2 * * *"
  timeZone: "Australia/Sydney"
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 900
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 3
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: report
            image: example.invalid/report:3.1.0
            args: ["generate-nightly"]
```

The schedule uses five cron fields for minute, hour, day of month, month, and day of week. Without `.spec.timeZone`, the controller interprets the schedule in its configured local time zone. An explicit IANA time-zone name prevents ambiguity when cluster components use a different zone. Daylight-saving transitions can still repeat or omit local clock times, so critical processes should account for those transitions.

`suspend: true` prevents future scheduled Jobs but does not stop Jobs that already started. Unsuspending can make missed schedules eligible to start, subject to the deadline and the controller's missed-schedule rules.

`startingDeadlineSeconds` limits how late the controller may start a missed occurrence. A short deadline may skip work during a brief control-plane outage. An absent deadline permits broader catch-up, although the controller limits excessive missed schedules. The correct value follows the business tolerance for delayed execution.
### Concurrency and retention
The concurrency policy applies only to Jobs created by the same CronJob:

| Policy | Behaviour when the previous Job still runs |
| --- | --- |
| `Allow` | Starts the new Job and permits overlap |
| `Forbid` | Skips that scheduled occurrence |
| `Replace` | terminates the active Job and starts a replacement |

`Forbid` does not mark the running Job as failed. It leaves that Job running and omits the overlapping occurrence. `Replace` requires a task that can safely stop and restart. `Allow` requires independent executions or coordination that prevents harmful concurrent side effects.

`successfulJobsHistoryLimit` and `failedJobsHistoryLimit` count finished Job objects, not Pods. Their defaults are three and one. Setting either value to zero removes the corresponding finished Jobs promptly. A separate log and audit system should preserve required evidence before cleanup.

CronJob scheduling provides operational scheduling rather than an exact-once transaction guarantee. A controller interruption can delay a run, a missed deadline can omit one, and unusual races can create duplicate work. Each Job should therefore tolerate repetition, identify the schedule interval it processes, and commit side effects idempotently.

```bash
kubectl get cronjob nightly-report
kubectl get jobs --sort-by=.metadata.creationTimestamp
kubectl create job --from=cronjob/nightly-report nightly-report-manual
kubectl patch cronjob nightly-report --type=merge -p '{"spec":{"suspend":true}}'
```

Creating a one-time Job from the CronJob template supports testing without changing the recurring schedule. Test inputs and outputs should remain isolated from production work, particularly when the template performs destructive or externally visible actions.
## StatefulSets
A StatefulSet manages Pods that need a stable identity. Each Pod receives an ordinal name such as `database-0`, `database-1`, and `database-2`. Replacing a failed Pod preserves its ordinal even though the replacement receives a new unique identifier and may run on another node.

StatefulSet identity combines three properties:
- A stable ordinal and Pod name
- A stable network identity, normally through a headless Service
- Stable storage claims created from `volumeClaimTemplates`

A headless Service uses `clusterIP: None`. It supports DNS records for individual StatefulSet Pods rather than distributing traffic through one virtual IP. An application can use these names for peer discovery and role assignment. A separate normal Service can provide load-balanced client access where the application permits it.

Each `volumeClaimTemplate` produces a PersistentVolumeClaim for each ordinal. Deleting or scaling down the StatefulSet does not automatically erase those claims under the default retention behaviour. This protects data but requires deliberate cleanup. Storage class, access mode, topology, reclaim policy, backup, and restore design determine whether stable claims produce durable recovery.

StatefulSets default to ordered Pod management. They create Pods from the lowest ordinal upward and normally require each predecessor to become ready before proceeding. They terminate Pods in reverse order. `Parallel` Pod management relaxes creation and deletion ordering but does not remove stable identities.

Rolling updates normally proceed from the highest ordinal down. A partition can restrict an update to Pods at or above a selected ordinal, which supports staged releases. Stateful application upgrades must also respect membership protocols, quorum, data format compatibility, and leader placement. Kubernetes controls Pod lifecycle, but the application must preserve distributed-system safety.

A StatefulSet does not automatically configure replication or copy data between Pods. It supplies stable building blocks. The application, operator, or external system still manages membership, replication, failover, backups, and recovery validation.
## Operational safeguards
Controller operations become safer when the surrounding configuration supports clear ownership and observable health.

- Version control should hold production manifests and record reviewed changes.
- Selectors should use stable, distinctive labels and should not overlap unintentionally.
- Readiness, liveness, and startup probes should express different health decisions.
- Resource requests should allow the scheduler to place replacement and surge Pods.
- Pod disruption budgets should reflect quorum and minimum service capacity.
- Termination handling should stop new work, drain active work, and exit within the grace period.
- Metrics, events, and central logs should reveal failed reconciliation and stalled rollouts.
- Images and configuration should use traceable, immutable release identifiers where practical.
- Batch tasks and scheduled work should tolerate retries and duplicate execution.
- Persistent workloads should test backup restoration and node or zone failure recovery.

Direct Pod changes rarely provide a durable fix because a controller can replace the Pod from its template. An administrator should identify the owner, update the highest appropriate controller, and observe reconciliation through status, events, and application telemetry.
## Command reference
```bash
# Inspect ownership and status
kubectl get deploy,rs,pods
kubectl describe deployment web
kubectl get pod POD_NAME -o jsonpath='{.metadata.ownerReferences}'
# Apply and scale
kubectl apply -f web-deployment.yaml
kubectl scale deployment/web --replicas=5
# Update and observe
kubectl set image deployment/web web=nginx:1.28
kubectl rollout status deployment/web --timeout=10m
kubectl rollout history deployment/web
# Pause, resume, restart, and roll back
kubectl rollout pause deployment/web
kubectl rollout resume deployment/web
kubectl rollout restart deployment/web
kubectl rollout undo deployment/web --to-revision=3
# Inspect node-wide and batch controllers
kubectl get daemonset -A
kubectl get jobs
kubectl get cronjobs
kubectl get statefulsets
```

Every command acts within the current kubeconfig context and namespace unless flags select another target. Operational procedures should verify the context, namespace, object name, and intended change before mutation. Role-based access control should grant only the verbs and resources required for the task.