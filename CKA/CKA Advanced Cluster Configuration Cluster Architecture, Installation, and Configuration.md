# Advanced Kubernetes cluster configuration
## Secure API access with RBAC
Kubernetes administrators control API access through role-based access control, or RBAC. RBAC answers three questions: which subject is acting, which action it wants to perform and which resource it wants to affect. Subjects include users, groups and service accounts. Actions map to verbs such as `get`, `list`, `watch`, `create`, `update`, `patch` and `delete`. Resources include API objects such as Pods, Deployments, Secrets, Nodes and PersistentVolumes.

RBAC grants permissions additively. Kubernetes does not use deny rules in RBAC. When no RoleBinding or ClusterRoleBinding grants a requested permission, the API server rejects the request with `Forbidden`. This default supports least privilege when administrators grant only the permissions that a role needs. A permission review should start from the intended task and add only the verbs and resources required for that task.

Rules combine API groups, resources and verbs. Core resources, such as Pods and Services, use the empty API group. Workloads such as Deployments use the `apps` API group. `get` retrieves one object. `list` retrieves a collection. `watch` streams changes. A read-only Pod role in one namespace should grant `get`, `list` and `watch` on `pods`, not update or delete permissions. A role that manages Deployments must include the correct API group and resource name, otherwise the API server will not match the request.

Scope determines the correct RBAC object:
- A Role grants permissions inside one namespace.
- A ClusterRole defines permissions at cluster scope.
- A RoleBinding grants a Role or ClusterRole inside one namespace.
- A ClusterRoleBinding grants a ClusterRole across the cluster.

ClusterRoles suit non-namespaced resources such as Nodes and PersistentVolumes. They also suit reusable permission sets, including the built-in `view`, `edit`, `admin` and `cluster-admin` roles. Binding a ClusterRole with a RoleBinding limits those ClusterRole rules to the RoleBinding namespace. Binding the same ClusterRole with a ClusterRoleBinding grants the rules globally. This distinction prevents accidental cluster-wide access.

ClusterRole aggregation lets Kubernetes combine several ClusterRoles into a parent ClusterRole by label selector. This keeps permission libraries modular. For example, a monitoring platform can add permissions for its own custom resources by labelling its ClusterRoles so the monitoring administrator role inherits them automatically.

Imperative `kubectl create role` and `kubectl create rolebinding` commands can generate manifests quickly. `--dry-run=client -o yaml` is safer for exam and production work because it prints the object without creating it. Administrators can then inspect the API group, resource, verb, subject and namespace before applying the manifest. Deleting a namespace removes the namespaced Roles and RoleBindings inside it.
## Validate and troubleshoot access
Administrators should verify RBAC with `kubectl auth can-i`. The command sends a SubjectAccessReview to the API server and returns whether the nominated subject can perform the action. Impersonation with `--as` allows an administrator to test a user without using that user's credentials. The same approach works for service accounts, which often need tightly scoped access for Pods and controllers.

Effective checks include:

```bash
kubectl auth can-i get pods --namespace test-space --as test-user
kubectl auth can-i delete pods --namespace test-space --as test-user
kubectl auth can-i --list --namespace test-space --as test-user
kubectl auth can-i get /healthz
```

`kubectl` verbosity exposes how the client talks to the API server. `-v=6` shows request URLs and response codes, which helps identify namespace, API group and endpoint mistakes. `-v=8` shows request and response headers and more raw HTTP detail. That level can reveal credential and kubeconfig problems, but logs from high verbosity can contain sensitive authentication material and need careful handling.

An RBAC denial normally identifies the user, verb, resource, API group and namespace. Administrators should use those fields to grant the smallest useful permission rather than broad access. A `Forbidden` response points to authorisation. A `NotFound` response may point to a missing resource, wrong API group or wrong namespace. High verbosity separates those cases from local command syntax errors and network failures.
## Manage configuration with Helm and Kustomize
Production Kubernetes administration requires repeatable configuration, not hand-edited YAML. Helm and Kustomize solve different parts of that problem.

Helm packages Kubernetes resources into charts. A chart contains metadata in `Chart.yaml`, templates, default values in `values.yaml` and optional dependencies. A release is a running instance of a chart. Helm renders templates with supplied values, sends the resulting manifests to Kubernetes and stores release history. That history supports upgrades and rollbacks when a new application version or configuration fails.

A typical Helm workflow adds a repository, updates the local index, searches for a chart, inspects values, installs a named release and later uninstalls or rolls it back. Chart version and application version are different. Chart version tracks packaging and templates. Application version tracks the software deployed by the chart. Values files let teams adjust replica counts, image tags, Service types, resource requests and other settings without editing chart templates.

Helm improves lifecycle management because it tracks all resources that belong to a release. `helm uninstall` removes the release objects as a set. `helm history` lists revisions. `helm rollback` returns a release to a previous revision when a failed upgrade causes crash loops or incorrect configuration. The package model suits third-party applications and shared internal platforms.

Kustomize manages environment differences without templating syntax. A base contains ordinary Kubernetes manifests that describe the shared application. An overlay points to the base and applies changes such as name prefixes, replica counts, labels, images, ConfigMaps, Secrets or environment-specific patches. `kubectl kustomize` renders the final manifest. `kubectl apply -k` applies it.

Kustomize uses patching rather than template substitution. An overlay can change the replica count for development, set a production hostname or add labels while leaving the base files untouched. Kustomize generators can create ConfigMaps and Secrets from local files. Kustomize appends hashes to generated names, which lets Kubernetes roll workloads when configuration changes. Small overlays make differences between development, staging and production easy to review.
## Extend the platform
Kubernetes stays portable because it delegates runtime, networking and storage work to standard interfaces.

- The Container Runtime Interface lets the kubelet talk to runtimes such as containerd or CRI-O.
- The Container Network Interface lets compatible plugins implement the Kubernetes network model, assign Pod networking and support features such as network policy.
- The Container Storage Interface lets storage vendors expose storage systems to Kubernetes workloads through drivers.

This modularity points troubleshooting in the right direction. A container that will not start often involves the runtime, image or kubelet. A Pod that cannot reach another Pod often involves CNI, routing, DNS or network policy. A volume that will not attach or mount often involves CSI, a StorageClass, a PersistentVolumeClaim or provider credentials. The control plane orchestrates the request, while specialised plugins perform provider-specific work.

CustomResourceDefinitions extend the Kubernetes API with new resource types. After installing a CRD, users can create and manage that resource with `kubectl`, RBAC and the normal API machinery. Cert-manager, for example, adds resource types such as `certificates`, `issuers`, `clusterissuers`, `orders` and `challenges`. Before the CRDs exist, the API server cannot recognise those resources. After installation, a query may return no objects rather than an unknown resource error.

A CRD defines and stores data. An operator adds behaviour. Operators are controllers that watch custom resources, compare the current state with the desired state and take action. They encode operational knowledge for tasks such as provisioning databases, rotating certificates, running backups, applying schema changes and handling failover. This moves administration from manual runbooks to reconciliation loops that keep complex services healthy.
## Build a highly available control plane
A production control plane should survive node failure. Kubernetes stores cluster state in etcd, a distributed key-value store that relies on quorum. Quorum is the majority required to make safe decisions, calculated as `floor(N/2) + 1`. A three-member etcd cluster needs two members for quorum and can lose one. A five-member cluster needs three and can lose two. A two-member etcd cluster still needs two members, so it cannot tolerate the loss of either member.

Control plane designs usually use one of two topologies:
- A stacked topology runs etcd on each control plane node. It is common, efficient and simpler for kubeadm to manage.
- An external etcd topology runs etcd on separate hosts. It costs more and adds operational complexity, but it isolates database performance from API server load.

A highly available API needs a stable endpoint in front of all API servers. A load balancer such as HAProxy can distribute traffic across healthy control plane nodes. Keepalived can provide a virtual IP so another load balancer takes over if the active one fails. The load balancer must perform health checks and send traffic only to live API servers. Without that stable endpoint, workers and administrators would need to know every control plane address.

Kubeadm must know the shared endpoint during initialisation. `kubeadm init --control-plane-endpoint` places the load balancer DNS name or virtual IP into the cluster configuration and certificates. This prevents TLS failures when clients connect through the shared endpoint rather than a single node IP. `--upload-certs` stores encrypted control plane certificates temporarily so additional control plane nodes can join. Later control plane joins need `kubeadm join --control-plane` and the matching certificate key. Worker joins omit the control plane flag.
## Recover etcd after disaster
High availability protects against infrastructure failure. It does not protect against logical failure. If an authorised user deletes a production namespace, a healthy HA control plane faithfully records and replicates that deletion. Etcd snapshots provide the recovery point.

Etcd holds the cluster state and requires TLS authentication for direct access. A snapshot command normally supplies the endpoint, trusted CA certificate, client certificate and client key:

```bash
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/etcd-backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```

A safe restore writes to a new data directory, not the active etcd directory. On kubeadm clusters, etcd commonly runs as a static Pod under `/etc/kubernetes/manifests`. Recovery stops the affected static Pods or moves manifests aside, restores the snapshot into a clean directory, updates the etcd static Pod manifest to mount the restored path, then lets the kubelet restart the control plane. Verification checks nodes, namespaces and all Pods after the API server returns.

Backups need regular testing. A snapshot that cannot be restored does not provide disaster recovery. Administrators should protect snapshot files because they contain cluster state, including objects that may hold sensitive data. Restore procedures should record the exact etcd member, data directory, certificate paths and static Pod manifest paths used by the cluster.