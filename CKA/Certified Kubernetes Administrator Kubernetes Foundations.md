# Certified Kubernetes Administrator: Kubernetes Foundations
> [!NOTE]
> This guide explains Kubernetes administration fundamentals, from cluster architecture and declarative `kubectl` workflows to workloads, networking, security, and evidence-led troubleshooting.

Kubernetes, often abbreviated to K8s, automates the deployment, scaling, and management of containerised applications. It uses a declarative model. An administrator submits a desired state, and Kubernetes controllers continually compare that state with the cluster's actual state and close any gap. This reconciliation process drives deployment, scaling, replacement, and recovery.

A cluster contains a control plane and one or more worker nodes. The control plane stores cluster state and makes global decisions. Worker nodes run application workloads in Pods. Production clusters commonly distribute control plane components and workloads across several machines for resilience, while a compact three-node `kubeadm` cluster provides a useful learning environment.
## Cluster architecture
### Control plane components
| Component | Function |
| --- | --- |
| `kube-apiserver` | Exposes the Kubernetes HTTP API, authenticates and authorises requests, applies admission controls, validates objects, and persists API data in `etcd`. It forms the main entry point for `kubectl`, controllers, kubelets, and other clients. A `kubeadm` API server normally listens on TCP port 6443. |
| `etcd` | Stores Kubernetes API data in a consistent, highly available key-value database. Administrators must protect and back it up because it records the cluster's authoritative state. The API server normally provides the controlled path to this data. |
| `kube-scheduler` | Watches for Pods without a node assignment, filters and scores suitable nodes, and records a binding. It considers resource requests, taints and tolerations, affinity, topology, storage, and other constraints. It selects a node but does not start containers. |
| `kube-controller-manager` | Runs control loops for resources such as Deployments, ReplicaSets, Nodes, and Jobs. Each controller observes API state and acts to bring actual state closer to desired state. |

In a cluster created by `kubeadm`, the local kubelet starts the API server, scheduler, controller manager, and usually `etcd` as static Pods. `kubeadm` writes their manifests to `/etc/kubernetes/manifests`, and the kubelet watches that directory. Editing a manifest causes the kubelet to recreate the corresponding static Pod. This bootstrap design lets the control plane start before the API server can schedule ordinary Pods.

Nodes usually initiate API requests to the control plane. The API server also connects to kubelets for operations such as retrieving logs, attaching to containers, and port forwarding. Secure cluster design must therefore account for communication in both directions.
### Node components and extension interfaces
Each worker node runs these core services:
- The `kubelet` registers the node, watches assigned Pod specifications, asks the container runtime to create containers, runs health checks, and reports Pod and node status.
- A container runtime, commonly containerd or CRI-O, pulls images and manages containers. The kubelet communicates with it through the Container Runtime Interface, or CRI. Containerd implements CRI. It is not itself an interface. Kubernetes removed its built-in Docker Engine integration, `dockershim`, in version 1.24, although a CRI adapter can still connect Docker Engine.
- `kube-proxy`, or an equivalent implementation supplied by a network plugin, implements Service traffic forwarding on each node.

Kubernetes separates core orchestration from infrastructure through standard interfaces:
- CRI connects the kubelet to a compatible container runtime. `crictl` inspects and diagnoses CRI-compatible runtimes on a node.
- Container Network Interface, or CNI, plugins implement Pod networking and allocate network connectivity. A cluster must install a compatible plugin, such as Calico or Cilium, before ordinary Pods can communicate correctly.
- Container Storage Interface, or CSI, drivers connect Kubernetes to storage systems and support operations such as provisioning and mounting volumes.

CoreDNS commonly supplies in-cluster DNS. It watches Kubernetes resources and creates DNS records so workloads can reach Services by stable names instead of changing Pod IP addresses.
### Practical lab topology
A compact lab can place one control plane and two workers on Ubuntu virtual machines. `kubeadm` initialises the control plane and produces a join command for the workers. Containerd supplies the runtime, while a chosen CNI plugin supplies Pod networking. The Pod network range must not overlap the host, Service, or upstream network ranges. CoreDNS should become healthy after the CNI starts carrying Pod traffic.

Vagrant can reproduce this topology across Hyper-V, VirtualBox, VMware, Parallels, or another supported provider, but Kubernetes does not depend on Vagrant. Three cloud virtual machines or local machines work equally well when their operating systems, ports, addresses, and routes satisfy Kubernetes requirements. Version-controlled provisioning files make rebuilds consistent and keep experiments disposable. `vagrant halt` stops machines without deleting them, while `vagrant destroy` removes them.

Basic inspection should confirm the cluster before any workload exercise:

```bash
kubectl version
kubectl get nodes -o wide
kubectl cluster-info
kubectl get pods -n kube-system -o wide
systemctl status kubelet
```

The node list exposes roles, readiness, versions, addresses, and container runtimes. The `kube-system` Pod list shows control plane mirror Pods and installed add-ons. On a newly joined node, `NotReady` commonly points to an absent or unhealthy CNI, a failed kubelet, a runtime problem, or network reachability. The administrator should diagnose the affected node and avoid assuming that the API server caused every readiness failure.
### Releases and version skew
Kubernetes versions use `major.minor.patch` notation. The project maintains the three most recent minor release branches, and current releases receive about one year of patch support. Administrators should use supported releases and pin component packages to intended patch versions.

The API server leads a supported upgrade sequence. A kubelet must not run a newer minor version than any API server it contacts, and it may run up to three minor versions older. `kube-proxy` follows a similar limit. The scheduler and controller managers should match the API server's minor version, although they may be one minor version older during an upgrade. `kubectl` may run within one minor version of the API server. High availability can narrow these ranges when API servers run different versions. Administrators should consult the current skew policy before every upgrade.
## Efficient kubectl workflows
`kubectl` sends authenticated requests to the Kubernetes API. It reads connection details from a kubeconfig file, normally `~/.kube/config`, unless an option or environment variable selects another file. A kubeconfig defines clusters, users, and contexts. Each context combines a cluster, a user, and an optional default namespace.

Context checks prevent changes to the wrong environment:

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
```

An administrator should verify the context and namespace before changing resources. Shell completion and a short alias can reduce typing, but the alias should retain completion support.
### Imperative-to-declarative pipeline
Imperative commands provide fast resource creation and inspection:

```bash
kubectl run probe --image=busybox:1.36
kubectl create deployment web --image=nginx:stable --replicas=3
kubectl expose deployment web --port=80 --target-port=80
kubectl scale deployment web --replicas=5
```

`kubectl run` creates a standalone Pod. No workload controller replaces that Pod if an administrator deletes it or its node fails. `kubectl create deployment` creates a Deployment that manages a ReplicaSet, which manages Pods. `kubectl expose` can copy a workload's labels into a Service selector, which reduces selector errors. `kubectl scale` changes the live object, but it does not update a manifest stored in version control.

When an imperative command cannot set every required field, client-side dry-run can generate a valid starting manifest without sending it to the API server:

```bash
kubectl create deployment web --image=nginx:stable \
  --dry-run=client -o yaml > web.yaml
vi web.yaml
kubectl apply -f web.yaml
```

This sequence generates most boilerplate, leaves only the necessary fields for manual editing, and creates a durable declaration. `kubectl apply` then creates or updates resources from the file. Reapplying the same declaration supports repeatable administration, but competing field managers or later imperative changes can still alter the result.
### Resource manifests
Kubernetes API objects commonly use these top-level fields, although submitted manifests normally omit `status`:

| Field | Purpose |
| --- | --- |
| `apiVersion` | Selects the API group and version, such as `v1` for a Pod or `apps/v1` for a Deployment. |
| `kind` | Identifies the resource type. |
| `metadata` | Supplies identity and organisation, including the name, namespace, labels, and annotations. |
| `spec` | Declares the desired state for resources that define a specification. |
| `status` | Reports observed state. Controllers usually write this field, and administrators normally omit it from declarations. |

Not every API object has a `spec` or `status`, so the server's schema remains authoritative. YAML uses indentation to express structure. Spaces must remain consistent, and tabs should not indent YAML. Two spaces form a common convention, not a language requirement.

`kubectl explain` retrieves field descriptions from the server's OpenAPI schema. `kubectl api-resources -o wide` lists the kinds, short names, API groups, namespace scope, and supported verbs available in the current cluster. These commands reflect installed custom resources as well as built-in APIs.
### Queries and configuration objects
Precise queries reduce noise:
- Label selectors filter metadata chosen by users or controllers. Multiple comma-separated requirements form a logical AND. Set-based selectors support values such as `environment in (dev,test)`, but selectors do not provide a general logical OR between independent requirements.
- Field selectors filter supported object fields, such as `status.phase=Running` or `spec.nodeName=worker-1`.
- `-o wide`, `-o custom-columns`, JSONPath, and YAML output shape returned data for people or scripts.
- `--sort-by=.metadata.creationTimestamp` orders objects by creation time. Event series may record later repetitions separately from the original creation time, so this ordering does not always show the latest occurrence.

`kubectl get all` returns a useful collection of common workload resources, but it does not list every resource type. It omits resources such as Secrets, ConfigMaps, Roles, RoleBindings, and many storage objects.

Useful discovery commands combine server-side filtering with concise client output:

```bash
kubectl get pods -l app=web
kubectl get pods --field-selector=status.phase=Running
kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
kubectl get events --sort-by=.metadata.creationTimestamp
```

Server-side label and field selectors reduce the objects returned over the API. Output formats then shape the result locally. Custom columns suit rapid human inspection, while JSONPath produces compact values for scripts. Wide YAML dumps remain valuable for complete inspection, but targeted output makes missing node assignments, labels, addresses, and ownership easier to detect.

ConfigMaps hold non-confidential configuration outside container images. Secrets hold sensitive values, but the `data` field only base64-encodes them. Base64 does not encrypt data. Cluster operators should enable encryption at rest, limit RBAC access, and consider an external secrets system where risk warrants it.

RBAC separates permissions from subjects. A Role grants namespaced permissions, while a RoleBinding assigns a Role or ClusterRole to users, groups, or service accounts within a namespace.

Resource requests guide scheduling. The scheduler places a Pod only where requested resources fit. The kubelet and container runtime enforce limits through operating-system controls. Linux normally throttles CPU use above a CPU limit and may terminate a container through an out-of-memory kill when it exceeds a memory limit under pressure.

A dry-run manifest gives resource controls, probes, volumes, security settings, and node placement rules a valid scaffold. Administrators should edit only required fields, validate indentation, review the resulting diff, and apply the declaration. This practice reduces transcription errors without replacing an understanding of the object schema.
## Workloads, Services, and organisation
A Pod wraps one or more tightly coupled containers that share a network namespace and can share volumes. Higher-level controllers should manage most application Pods.

```text
Deployment -> ReplicaSet -> Pods
```

A ReplicaSet maintains the requested number of matching Pods. A Deployment manages ReplicaSets and performs declarative rollouts. During an image update, it creates a new ReplicaSet, increases the new replica count, and reduces the old count. Retained ReplicaSet history can support rollback. The kubelet may restart a failed container inside a standalone Pod according to its restart policy, but only a controller can replace the Pod after deletion or node loss.

Pod IP addresses change as controllers replace Pods. A Service provides a stable access point and selects backend Pods through labels. Common Service types include:
- `ClusterIP` exposes a stable virtual IP inside the cluster and serves as the default type.
- `NodePort` opens a port on every node and forwards traffic to Service endpoints. The default range is 30000 to 32767.
- `LoadBalancer` asks an installed cloud or load-balancer controller to provision external access. A bare cluster without such a controller may leave the external address pending.

The EndpointSlice controller normally records the network endpoints selected for each Service. Kubernetes deprecated the older Endpoints API in version 1.33. Missing EndpointSlices or a lack of usable endpoints can indicate mismatched selectors, unready Pods, missing Pods, or another endpoint selection problem. This evidence does not by itself prove a network fault.

Namespaces scope names for namespaced resources and provide boundaries for controls such as quotas, namespaced RBAC, and NetworkPolicies. They do not contain cluster-scoped resources, and a namespace alone does not enforce isolation.

Labels identify selectable attributes and connect Services and controllers to Pods. Administrators should design stable labels and keep a Deployment selector, Pod template labels, and Service selector consistent. Annotations hold non-identifying information for people and tools, such as build identifiers, checksums, and ownership details. Label selectors cannot query annotations.
## Diagnostic ladder
Troubleshooting should move from broad state to specific evidence:
1. `kubectl get` reveals phase, readiness, restart count, age, node placement, and obvious status strings.
2. `kubectl describe` shows configuration, conditions, ownership, scheduling information, and related events. Recent events often state the scheduler's or kubelet's reason for failure.
3. `kubectl logs` reads application output. `kubectl logs <pod> --previous` retrieves the preceding container instance after a restart. Multi-container Pods require a container name.
4. `kubectl get events` supplies wider cluster context. Events expire according to cluster configuration, and repeated events can retain an earlier creation time. Their absence does not prove that no failure occurred.

Status strings guide the next check:

| Symptom | Meaning and first checks |
| --- | --- |
| `ImagePullBackOff` | The kubelet cannot pull an image and retries with increasing delay. `describe` and events usually reveal an invalid image name, unavailable tag, registry failure, or missing pull credentials. |
| `CrashLoopBackOff` | A container repeatedly starts and exits, and Kubernetes delays later restarts. Current and `--previous` logs, termination details, commands, configuration, and probes usually expose the cause. |
| `Pending` | Kubernetes accepted the Pod, but one or more containers have not started. The phase can cover scheduling, image retrieval, and setup. The `PodScheduled` condition, `describe`, and events distinguish insufficient resources, taints, affinity rules, unmatched node selectors, unbound volume claims, image delays, and other causes. |
| Node `NotReady` | The control plane has lost reliable node health information. An administrator should inspect the kubelet service, container runtime, CNI, disk, memory, certificates, and node networking on that node. |
| Service has no backend traffic | EndpointSlices reveal whether the Service selected ready endpoints. The investigation should compare Service selectors with Pod labels, readiness, ports, namespaces, and network policy before blaming the network implementation. |

Ownership explains recovery. Deleting a standalone Pod removes it, while deleting a Deployment-owned Pod prompts its ReplicaSet to create a replacement. Selectors explain connectivity. A Service can exist, resolve in DNS, and still send no traffic when its selector finds no ready Pods. Context discipline, generated manifests, consistent labels, and an evidence-led diagnostic sequence provide the foundation for reliable cluster administration.