# Kubernetes Cluster Management and Lifecycle with kubeadm
> [!NOTE]
> This guide explains the full lifecycle of a self-managed Kubernetes cluster with `kubeadm`, from deployment planning and node preparation through installation, networking, validation, maintenance, and upgrades.
## Choose the deployment model
An organisation can run Kubernetes as a managed cloud service, on cloud virtual machines, or on premises. A managed service transfers much of the control-plane operation, patching, and infrastructure maintenance to the provider. A self-managed cluster offers greater control but makes the organisation responsible for the operating system, container runtime, Kubernetes components, networking, upgrades, security, and recovery.

Kubeadm bootstraps and upgrades conformant Kubernetes clusters. It suits laboratories, repeatable infrastructure automation, and self-managed environments. It does not provision machines, install a pod network, or provide a complete production platform. A production design also needs high availability, load balancing, durable etcd protection, monitoring, backups, access control, and tested recovery procedures.

Desktop distributions provide a faster route for local development and basic experimentation, but they do not reproduce every operational characteristic of a multi-node cluster. Cloud virtual machines remove physical infrastructure work while leaving the guest operating systems and Kubernetes stack under the organisation's control. Selection should follow application requirements, regulatory obligations, staff capability, failure tolerance, cost, and the organisation's broader platform strategy.
## Prepare every node
A basic Linux cluster requires full network connectivity between nodes, open component and CNI ports, unique hostnames, unique MAC addresses, and unique `product_uuid` values. Each machine should have at least 2 GiB of RAM, while each control-plane machine should have at least two CPUs. Production capacity must cover system overhead, application requests, disruption, maintenance, and growth.

The administrator should also:
- Select a supported Linux distribution and kernel.
- Configure stable addressing, routing, name resolution, and time synchronisation.
- Ensure that pod, Service, host, and external network ranges do not overlap.
- Enable IPv4 forwarding when the selected network implementation requires it.
- Disable swap for the simplest configuration, or explicitly configure kubelet swap support with `failSwapOn` and `memorySwap` after assessing scheduling, storage, security, and performance effects.
- Install a CRI v1 compatible runtime, such as containerd or CRI-O.

The kubelet and container runtime must use compatible cgroup drivers. On a systemd host, both should use the `systemd` driver. Kubeadm selects `systemd` for the kubelet by default. Containerd may require an explicit runtime setting, followed by a service restart. Its CRI plugin must remain enabled.

Before bootstrapping, the administrator installs these components:

| Component | Purpose | Placement |
|---|---|---|
| `kubelet` | Runs pods and reports node state | Every cluster node |
| `kubeadm` | Initialises, joins, and upgrades nodes | Every node managed with kubeadm |
| `kubectl` | Sends administrative requests to the API server | Administrative hosts, including a control-plane host if desired |
| CRI runtime | Starts and manages containers | Every cluster node |

For Debian-based systems, each Kubernetes minor release has a dedicated `pkgs.k8s.io` repository. The administrator adds the repository signing key, selects the minor release, installs explicit package versions, and places `kubelet`, `kubeadm`, and `kubectl` on hold. Kubernetes upgrades require a controlled sequence, so routine operating-system updates must not advance these packages independently. The container runtime needs its own security and compatibility lifecycle and should not be held without a deliberate policy.

Before `kubeadm init` or `kubeadm join`, the kubelet may restart repeatedly while it waits for kubeadm to supply its configuration. That behaviour does not by itself show a failed installation.
### Repeatable node configuration
Every node should receive the same tested baseline through automation. The baseline normally configures the kernel and network, installs the CRI runtime, aligns cgroup settings, adds the Kubernetes repository, and installs pinned Kubernetes packages. The administrator verifies each stage before cloning or applying it across the fleet.

Typical checks include:

```bash
swapon --show
sysctl net.ipv4.ip_forward
systemctl status containerd
crictl info
kubeadm version
kubelet --version
```

An empty `swapon --show` result confirms that swap is inactive. Runtime inspection should show the intended CRI endpoint and cgroup driver. Package versions should follow Kubernetes version-skew rules. A host firewall must allow Kubernetes control-plane, kubelet, and CNI traffic without exposing sensitive ports to untrusted networks.

Static host-file entries can provide name resolution in a laboratory, but production systems benefit from managed DNS, stable addresses, and a durable load-balanced control-plane endpoint. Disk sizing must cover container images, logs, writable layers, and etcd data. Separate monitoring should alert before filesystem, inode, memory, or certificate exhaustion disrupts the cluster.
## Initialise the control plane
The first control-plane node starts with `kubeadm init`. A configuration file provides a clearer, repeatable record than a long command when the cluster needs custom settings. The configuration should define the Kubernetes version, stable control-plane endpoint, node address, pod CIDR, Service CIDR, and any required certificate subject alternative names. A stable control-plane endpoint is essential if the cluster may later gain more control-plane nodes.

Kubeadm performs the following work:
1. Runs preflight checks for privileges, resources, networking, swap, the runtime, and existing state.
2. Pulls the required control-plane images.
3. Creates or consumes a certificate authority and issues component certificates.
4. Writes kubeconfig files for administrators and control-plane components.
5. Writes static pod manifests for the API server, controller manager, scheduler, and local etcd.
6. Starts the kubelet and waits for the control plane.
7. Labels and taints the node as a control-plane node.
8. Creates bootstrap-token and TLS-bootstrap configuration for joining nodes.
9. Deploys CoreDNS and kube-proxy.

The default control-plane taint blocks ordinary workloads that lack a matching toleration. It does not create an absolute ban, because a pod with the appropriate toleration can still run there.

The successful command output includes instructions for configuring `kubectl` and joining other nodes. The administrator records the non-secret cluster configuration but handles join tokens, certificate keys, and privileged kubeconfig files as credentials. A failed preflight check should prompt correction of the underlying condition. Bypassing checks with ignore flags can hide a real incompatibility and requires a documented justification.

Kubeadm creates a local etcd member when the configuration does not name an external etcd cluster. This arrangement makes a single control-plane node unsuitable for resilient production service. A highly available design uses multiple control-plane nodes, a stable API endpoint, and an etcd topology that can tolerate the planned failure set. Kubeadm cannot convert a cluster initialised without a stable `controlPlaneEndpoint` into high availability, so that endpoint belongs in the plan.
### PKI and kubeconfig files
Kubeadm stores its local public key infrastructure under `/etc/kubernetes/pki`. The API server uses TLS to protect connections, while client certificates authenticate components and privileged users. Kubeadm can use an externally managed certificate authority when organisational policy requires it.

Kubeconfig files under `/etc/kubernetes` identify the API endpoint, trusted certificate authority, and client credentials. The generated `admin.conf` grants cluster-admin privileges through RBAC. The generated `super-admin.conf` belongs to the `system:masters` break-glass group and bypasses normal authorisation. Both files require strict protection, and administrators should issue separate, least-privilege credentials for routine users.

Worker joins receive the public CA information needed to establish trust. They do not receive the cluster CA private key. Additional control-plane nodes can receive encrypted control-plane certificates only through the separate certificate-upload and certificate-key process.

The CA certificate hash in a worker join command pins discovery to the expected root of trust. During TLS bootstrap, the kubelet authenticates with the short-lived token, requests its own client certificate, and moves to certificate-based authentication. This process limits distribution of long-lived private credentials while still requiring strict control of token creation and approval policy.
### Static control-plane pods
Kubeadm writes these manifests to `/etc/kubernetes/manifests`:
- `etcd.yaml`
- `kube-apiserver.yaml`
- `kube-controller-manager.yaml`
- `kube-scheduler.yaml`

The kubelet watches this directory and starts the defined static pods, including after a reboot. These pods use host networking, which lets the control plane start before a CNI network exists. An administrator should avoid leaving backup manifest files in the watched directory because the kubelet reads every eligible manifest there.
## Install pod networking
Kubernetes requires a CNI-based pod network. The selected implementation must support the cluster's operating systems, address families, routing design, NetworkPolicy requirements, and scale. Some CNI implementations use an overlay, while others use native routing or combine both approaches.

The Kubernetes network model normally gives each pod an address and permits direct pod-to-pod communication across nodes without network address translation. The chosen pod CIDR must not overlap host or external networks, and it must match both the kubeadm configuration and CNI configuration. Only one pod network should serve a cluster.

After initialisation, an authorised administrator applies the vendor's reviewed manifest:

```bash
kubectl apply -f <cni-manifest.yaml>
kubectl get pods -A
kubectl get nodes
```

CoreDNS normally remains pending until the pod network operates. The control-plane node should report `Ready`, and the CNI, CoreDNS, control-plane, and kube-proxy pods should reach their expected healthy states before workers join.

Network readiness requires more than pods entering `Running`. The administrator should test pod-to-pod communication across nodes, DNS resolution, Service routing, and any required NetworkPolicy enforcement. A NetworkPolicy object has no effect unless the chosen CNI supports and enforces it. Provider documentation also governs maximum transmission unit settings, encapsulation, routing, firewall rules, upgrades, and dual-stack support.
## Join worker nodes
Each worker needs the same node preparation, compatible package versions, kubelet, kubeadm, and a CRI runtime. `kubectl` is optional on a worker. The initial `kubeadm init` output includes a secret join command with the API endpoint, bootstrap token, and CA certificate hash. The default token expires after 24 hours.

An administrator can create a replacement command when required:

```bash
kubeadm token create --print-join-command
```

Running the printed command as root on a prepared worker performs preflight checks, discovers and authenticates the control plane, submits the kubelet certificate-signing request, writes local kubelet configuration, and registers the node. Join tokens permit authenticated nodes to enter the cluster, so administrators must protect, rotate, and delete them when no longer required.

The cluster should show every worker as `Ready`. CNI and kube-proxy DaemonSet pods should also run on each worker. A single-control-plane cluster remains a single point of failure and needs regular etcd backups. Production environments should use a documented high-availability design.

Node readiness can lag while the runtime pulls images and the CNI establishes networking. Persistent `NotReady`, certificate, sandbox, or registration errors require inspection of `journalctl -u kubelet`, runtime logs, node conditions, and CNI pods. Repeating `kubeadm join` without cleaning a partial state can obscure the original failure. The administrator should diagnose first, then use the documented reset and rejoin procedure when recovery requires it.
## Validate workloads and Services
A concise validation creates a Deployment, confirms ready replicas, exposes it through a Service, tests access from an appropriate network location, and then removes the test resources. Three replicas do not guarantee one pod on each of three workers. The scheduler may place multiple replicas on one node unless topology spread constraints, pod anti-affinity, or other scheduling rules require distribution.

A `ClusterIP` Service provides a stable virtual address inside the cluster and directs traffic to ready backend endpoints. External access requires another mechanism, such as a load balancer, NodePort, Ingress, or Gateway implementation.

Validation should confirm desired replicas rather than infer success from a single request. The administrator checks Deployment availability, pod readiness, EndpointSlices, events, and responses from several replicas. Topology rules can prove that scheduling spans the intended nodes or zones. Test resources should use a trusted image and explicit tag or digest, then be deleted after verification to restore the cluster's baseline.

Useful checks include:

```bash
kubectl get nodes
kubectl get pods -A -o wide
kubectl get deployments,services,endpointslices
kubectl describe pod <pod-name>
```
## Maintain worker nodes safely
Before planned maintenance, the administrator confirms that the remaining nodes have enough CPU, memory, storage, and replica capacity. PodDisruptionBudgets should protect workloads that require availability.

`kubectl cordon <node>` marks a node unschedulable but leaves existing pods in place. `kubectl drain <node> --ignore-daemonsets` also requests safe eviction of eligible pods and respects disruption budgets. DaemonSet pods remain, mirror pods cannot be deleted through the API, and unmanaged pods or pods using `emptyDir` may require carefully reviewed flags. A successful drain indicates that Kubernetes has safely evicted the pods covered by the command.

After operating-system, runtime, or hardware maintenance, the administrator verifies node and workload health, then runs `kubectl uncordon <node>`. Rebooting without a drain can leave workloads unavailable until node-health taints and pod tolerations trigger replacement. The exact delay depends on cluster and workload configuration, so an assumed fixed interval is unsafe.
## Upgrade a kubeadm cluster
An upgrade proceeds one supported step at a time. It cannot skip a minor release. The administrator reads the target release notes, version-skew policy, kubeadm upgrade guide, and CNI provider instructions, then confirms backups, capacity, and cluster health. The Kubernetes project recommends the latest patch release of a supported minor version.

Before any change, the administrator captures the current component versions, node state, configuration, certificate expiry, workload health, and backup status. Maintenance windows should allow rollback and observation after each node. Alerts, API responsiveness, DNS, Service routing, and application probes should remain under review throughout the rollout. A pause between batches helps expose faults before the same change reaches the remaining capacity in the cluster.

The safe order is:
1. Upgrade the first control-plane node.
2. Upgrade the CNI according to its provider's instructions.
3. Upgrade each additional control-plane node, one at a time.
4. Upgrade worker nodes one at a time, or in small batches that preserve required capacity.
5. Verify nodes, system pods, workloads, networking, and observability.

On the first control-plane node, the administrator upgrades `kubeadm`, verifies its version, runs `kubeadm upgrade plan`, and applies the approved target with `kubeadm upgrade apply <version>`. Kubeadm checks cluster health and version policy, updates control-plane static pod manifests, restarts changed components, updates managed configuration, and renews its managed certificates by default. Additional control-plane nodes use `kubeadm upgrade node` instead of `kubeadm upgrade apply`.

For each control-plane or worker node, the administrator then drains the node before the kubelet upgrade, upgrades `kubelet` and `kubectl` if installed, reloads systemd, restarts the kubelet, confirms health, and uncordons the node. A worker first receives the matching kubeadm package and runs `kubeadm upgrade node` before its kubelet upgrade.

Package repositories are tied to minor releases. A minor upgrade therefore requires changing the repository to the target minor before selecting package versions. The kubelet must never be newer than the API server. Supported temporary skew can assist a rolling upgrade, but matching component versions after completion reduces operational risk.

Kubeadm writes upgrade backups under `/etc/kubernetes/tmp`, including local etcd and manifest backups where applicable. These files support recovery but do not replace independent, tested etcd backups. If an upgrade stops unexpectedly, kubeadm's idempotent workflow can often resume, although recovery should follow the official procedure and the failure state.