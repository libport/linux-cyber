# CKA Advanced Cluster Configuration Cluster Architecture, Installation, and Configuration
Advanced cluster configuration combines strict authorisation, reusable deployment methods, stable extension interfaces, resilient control-plane design, and tested recovery procedures. Each layer requires explicit scope, controlled credentials, and operational verification.
## Role-based access control
Kubernetes RBAC grants permissions through additive rules. It contains no deny rules. After authentication, the API server evaluates the request against applicable authorisers. An RBAC rule grants an action only when the verb, resource or non-resource URL, API group, and scope match. An ungranted request normally receives an HTTP 403 response when RBAC controls the decision.

RBAC separates permissions from their assignment:

| Object | Permission scope | Binding scope |
| --- | --- | --- |
| `Role` | Resources in one namespace | `RoleBinding` assigns it within that namespace |
| `ClusterRole` | Cluster-scoped resources, non-resource URLs, or namespaced resources | `ClusterRoleBinding` assigns it across the cluster |
| `ClusterRole` with `RoleBinding` | Namespaced permissions from a reusable cluster-wide definition | The binding limits access to its namespace |

A subject can be a user, group, or ServiceAccount. Kubernetes stores ServiceAccounts as API objects, but an external identity provider usually manages human users and groups. Bindings should favour groups and workload-specific ServiceAccounts because this reduces repetitive configuration and simplifies review.

Least privilege starts with the narrowest verbs, resources, resource names, and namespaces. Wildcards can silently grant access to resources or verbs introduced by later releases. Cluster-wide bindings therefore require stronger justification than namespaced bindings. Built-in roles also need careful review. The `edit` role can access Secrets and run Pods as namespace ServiceAccounts, which may allow privilege escalation through those identities. Administrators should avoid changing default system roles because API-server reconciliation can restore missing permissions. Custom roles provide clearer ownership.

RBAC rules should name API groups explicitly. Core resources such as Pods use the empty group, while Deployments use `apps`. Subresources such as `pods/log` and `pods/exec` require separate entries. Permission to create Pods can indirectly expose mounted credentials or powerful node features, so admission policy and Pod Security controls must complement RBAC.

Aggregated ClusterRoles combine labelled rules into another ClusterRole. Aggregation supports extensible roles, but the controller manages the resulting `rules` field. Administrators should add or remove labelled source roles instead of editing aggregated rules directly.

Authorisation checks should begin with focused commands:

```bash
kubectl auth can-i get pods -n team-a
kubectl auth can-i create deployments.apps -n team-a --as=alice
kubectl auth can-i --list -n team-a --as=system:serviceaccount:team-a:builder
```

Impersonation requires its own RBAC permission. `kubectl auth can-i` evaluates authorisation, but it does not prove that admission controls, quotas, validation, or runtime conditions will accept an operation. Increased client verbosity can help diagnose request paths and responses, although high levels may expose credentials or sensitive payloads. Logs should use the lowest useful level and receive the same protection as other secrets.
## Reusable deployment configuration
### Helm
A Helm chart packages templates, default values, metadata, and optional dependencies. Installing a chart creates a release, and Helm stores release state in the cluster, normally in Secrets. A release is a Helm record rather than a native Kubernetes workload type.

Values from files and command-line flags override chart defaults. Teams should version environment values, inspect rendered manifests, and avoid placing unencrypted secrets in values files or command history. The following sequence supports review before mutation:

```bash
helm repo update
helm template payments ./chart -f values-production.yaml
helm upgrade --install payments ./chart -n payments --create-namespace \
  -f values-production.yaml --atomic
```

`helm history` lists revisions, and `helm rollback` reapplies an earlier release configuration. A rollback cannot reverse database migrations, persistent-volume changes, or effects produced outside Helm. `helm uninstall` removes resources tracked by the release, but hooks, persistent volume claims, and resources protected by retention annotations can remain. Safe charts document these lifecycle boundaries and use idempotent hooks.

Charts can place CRDs in a `crds` directory. Helm installs them before templated resources, but it does not upgrade or delete them automatically. Teams must manage CRD schema changes, stored versions, and removal as a separate lifecycle.
### Kustomize
Kustomize transforms plain Kubernetes manifests without a template language. A `kustomization.yaml` file declares resources, patches, name changes, labels, images, and generated ConfigMaps or Secrets. A shared base captures common configuration, while overlays express environment-specific differences.

```bash
kubectl kustomize overlays/production
kubectl diff -k overlays/production
kubectl apply -k overlays/production
kubectl delete -k overlays/production
```

Generators add a content hash to generated names by default. That change updates workload references and triggers a rollout when configuration changes. Teams should commit bases and overlays together, review rendered output, and keep patches small enough to reveal intent.

Remote bases and chart dependencies should be pinned to immutable revisions or verified packages. Floating references can change rendered resources without a local commit, weakening review, auditability, and rollback.

Helm suits packaged applications with release history and parameterised templates. Kustomize suits declarative variation across related manifests. A delivery pipeline can render a Helm chart and then apply Kustomize, but each additional transformation complicates provenance and debugging.
## Cluster extension interfaces and operators
Kubernetes relies on defined interfaces to separate orchestration from node implementations:

| Interface | Primary relationship | Responsibility |
| --- | --- | --- |
| CRI | kubelet to container runtime | Creates Pod sandboxes and manages containers and images |
| CNI | runtime integration to network plugins | Connects Pods to networks and configures interfaces and routes |
| CSI | Kubernetes to storage drivers | Provisions, attaches, mounts, resizes, and snapshots supported storage |

A conforming CRI runtime does not remove the need for planned node migration. Runtime changes can alter configuration, image storage, cgroups, and operational behaviour, so administrators should cordon, drain, change, test, and return nodes gradually. CNI capabilities also vary. NetworkPolicy enforcement works only when the selected network plugin supports it.

A CustomResourceDefinition extends the Kubernetes API with a new resource schema and storage endpoint. A custom resource alone stores desired state. A controller watches that state and reconciles external or native resources. An operator applies this pattern to specialised operational knowledge such as installation, upgrades, backup, failover, or certificate renewal.

Controllers should reconcile idempotently, report conditions through `status`, retry transient failures with backoff, and avoid assuming that events arrive once or in order. Leader election allows multiple controller replicas to provide availability without performing the same mutation concurrently. Metrics and events should expose stalled reconciliation without leaking secret values.

CRD schemas should use structural OpenAPI validation, appropriate status subresources, clear versioning, and conversion when versions diverge. Finalisers can protect cleanup, but a stalled finaliser can also block deletion. Deleting a CRD removes its stored custom resources, so removal requires a backup and migration plan.

cert-manager illustrates the pattern. `Issuer` and `ClusterIssuer` resources describe certificate authorities, `Certificate` resources request certificates, and controllers store issued material in Secrets. Workloads must reference or mount those Secrets explicitly. They do not receive certificates automatically.
## Highly available control planes
Control-plane availability depends on redundant API servers, a stable endpoint, healthy network paths, and an etcd cluster that retains quorum. The voting-member count, not the API-server count, determines etcd quorum.

| etcd members | Quorum | Member failures tolerated |
| ---: | ---: | ---: |
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |

An odd number of voting members provides the same failure tolerance as the next even number with fewer machines. Three members suit many clusters, while five can tolerate two failures at greater cost. Loss of quorum prevents etcd from committing state changes and stops normal control-plane progress. It should not be treated as a general read-only operating mode.

A stacked topology runs an etcd member beside each control-plane node. It reduces infrastructure and bootstrap complexity, although failure of a node removes both an API server and an etcd member. An external topology separates etcd from control-plane nodes and can isolate failure domains, but it adds machines, certificates, networking, monitoring, and recovery work.

A load balancer presents one stable `controlPlaneEndpoint` to clients and joining nodes. It should perform TCP or suitable health checks against API servers, exclude failed backends, and avoid becoming a single point of failure. DNS, virtual IPs, firewall rules, and load-balancer redundancy form part of the design.

Nodes should span independent failure domains where the platform permits. Capacity planning must allow the surviving control plane and worker pool to absorb a planned drain or an unplanned loss. Time synchronisation, certificate expiry monitoring, component alerts, and tested administrative access support the same availability objective.

For kubeadm, the first control-plane node must initialise the cluster with the stable endpoint and shared certificate workflow:

```bash
kubeadm init \
  --control-plane-endpoint "api.cluster.example:6443" \
  --upload-certs
```

The administrator then configures `kubectl`, installs a compatible CNI plugin, and verifies core Pods. Each additional control-plane node uses the generated join command with both `--control-plane` and `--certificate-key`. Worker nodes use the separate worker join command. Control-plane nodes should join one at a time, with component and etcd health checked after each addition.

The uploaded certificates reside temporarily in the `kubeadm-certs` Secret, encrypted by the certificate key. The key grants sensitive access and the uploaded data expires after two hours by default. If that window closes, `kubeadm init phase upload-certs --upload-certs` creates a new key and upload. Production automation should protect join tokens, certificate keys, and discovery information.
## etcd backup and recovery
An etcd snapshot captures Kubernetes API state, including Secrets. It does not capture container images, external databases, application-level state, or persistent-volume contents. Backup design must cover those systems separately.

Snapshots should run regularly, transfer to encrypted off-cluster storage, and use strict access controls. They should not remain inside the active etcd data directory. The snapshot interval should follow the recovery-point objective, while restore exercises should measure the recovery-time objective. Retention should include multiple generations because corruption or accidental deletion may remain unnoticed until the newest snapshot already contains the damage.

A kubeadm-managed member can create and verify a snapshot with its client certificates:

```bash
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /secure-backup/etcd.db

etcdutl snapshot status /secure-backup/etcd.db
```

Recovery requires a planned maintenance window. Administrators should stop API-server writes, preserve existing data directories, verify the snapshot, and restore with `etcdutl snapshot restore`. A restore creates new member data and a new logical etcd cluster. Every member in an HA cluster must restore from the same snapshot with the correct unique member name, peer URL, initial cluster, and data directory. Editing one static Pod on one member does not restore a multi-member cluster.

Kubernetes restores benefit from a revision bump and `--mark-compacted` because controllers may otherwise retain stale watch caches after the apparent revision moves backwards. After restoration, administrators should update static Pod manifests or service configuration, restart members and API servers in the required order, verify endpoint health and alarms, confirm node and workload state, and test a representative API change. Regular recovery exercises validate certificates, commands, storage access, timings, and staff readiness before an incident.