# Cluster Architecture, Installation, and Configuration for CKA: Cluster Management and Lifecycle
## Core purpose
Kubernetes administrators use kubeadm to build a standards-based cluster and manage its lifecycle. The process starts with sound installation choices, then prepares each host, bootstraps a control plane, deploys pod networking, joins worker nodes and validates that the cluster can run application workloads. Ongoing operations require disciplined node maintenance and controlled upgrades.

Administrators need practical knowledge of the Kubernetes API, API objects, pods, controllers, services, deployments, ReplicaSets, pod networks, control-plane nodes and worker nodes. They also need Linux administration skills, because kubeadm installs Kubernetes components onto operating system hosts and relies on system services, networking and package management.
## Installation choices
Organisations can deploy Kubernetes in three common ways:
- Managed cloud services, where the provider operates the control plane and much of the underlying platform.
- Infrastructure as a Service, where the organisation runs Kubernetes on cloud virtual machines.
- On-premises systems, where the organisation runs Kubernetes on bare metal or virtual machines.

The right option depends on organisational strategy, skills, operating model and compliance needs. Managed services reduce maintenance effort, while self-managed clusters provide greater control over configuration, upgrades and troubleshooting. kubeadm suits administrators who need to learn the cluster lifecycle, build lab environments or create a base cluster that other automation can extend.

Desktop Kubernetes tools, such as Docker Desktop or similar local environments, help with experimentation and application development. They do not replace understanding of a full cluster lifecycle, because production clusters require explicit decisions about networking, high availability, upgrades, security and capacity.
## Host requirements
A kubeadm cluster needs Linux hosts for the control plane and worker nodes. Windows can run worker workloads in supported configurations, but the installation path described here uses Linux nodes. Each node needs network connectivity to the other nodes, a unique hostname, a unique MAC address and a unique product UUID. A lab control-plane node needs at least 2 CPUs and 2 GiB of RAM. Production sizing must match the expected workload, system add-ons, failure tolerance and maintenance windows.

Each node must run a container runtime that implements the Container Runtime Interface. Common choices include containerd and CRI-O. Docker Engine no longer integrates directly through dockershim in current Kubernetes releases. Kubernetes can still use Docker Engine only through a CRI-compatible shim. On Linux systems that use systemd, the container runtime and kubelet should use matching cgroup drivers, with systemd commonly selected.

Kubernetes now supports controlled swap configurations, but the kubelet default on Linux still fails when it detects swap unless administrators configure kubelet to tolerate it. Lab installations normally disable swap with `swapoff` and make the change persistent through the host configuration.

Administrators install these components on each node:
- `containerd` or another CRI-compatible runtime
- `kubelet`, which runs node-level Kubernetes work
- `kubeadm`, which bootstraps, joins and upgrades nodes
- `kubectl`, which sends administrative requests to the API server

The current Kubernetes community package repositories use `pkgs.k8s.io`. Administrators should avoid relying on the deprecated legacy `apt.kubernetes.io` and `yum.kubernetes.io` repositories for current releases. Package holds help prevent routine operating system patching from upgrading Kubernetes components outside the supported upgrade process.
## Lab topology
A simple lab can use one control-plane node and three worker nodes. The control-plane node can also hold the administrator kubeconfig used by `kubectl`. Hostname and address records can live in `/etc/hosts` when no DNS service exists. The lab still needs reliable node-to-node connectivity, adequate CPU and memory, persistent disk space and disabled or explicitly configured swap.

A representative topology uses:
- `c1-cp1` as the control-plane node
- `c1-node1`, `c1-node2` and `c1-node3` as worker nodes

This structure gives enough worker capacity to test scheduling, services and maintenance while keeping the environment small enough for a local virtualisation platform.
## Preparing each node
Node preparation follows the same pattern for control-plane and worker nodes. Administrators disable or configure swap, enable IPv4 forwarding where the network plugin requires it, install the container runtime, generate its default configuration and set the systemd cgroup option when appropriate. After each configuration change, administrators restart the runtime and verify that the service runs correctly.

The kubelet may appear to crash or remain inactive before cluster initialisation. That behaviour is expected when kubeadm has not yet provided cluster configuration or static pod manifests. The runtime, such as containerd, should be active before kubeadm starts.

After the container runtime works, administrators add the Kubernetes package repository for the target minor release, install `kubelet`, `kubeadm` and `kubectl`, then hold the packages. Version pinning matters because Kubernetes upgrades need deliberate sequencing rather than automatic package drift.

Package installation should use explicit target versions when the cluster must start below the latest available patch for upgrade practice or staged rollout. Administrators can query the package manager for available versions, install the chosen version, then apply package holds. That sequence keeps the node predictable. Automatic upgrades may place the kubelet, kubeadm and control-plane version outside the supported version skew and can interrupt cluster operations.

Runtime verification should precede kubeadm commands. `systemctl status containerd` or an equivalent runtime check should show an active service. The kubelet may restart until kubeadm supplies configuration, but that condition should disappear after initialisation or joining. Administrators should treat persistent kubelet failures after those steps as a signal to inspect the runtime socket, cgroup driver, swap configuration, node identity and kubelet logs.
## Bootstrapping the control plane
`kubeadm init` creates the first control-plane node. It runs preflight checks, verifies system resources and permissions, checks the container runtime, pulls control-plane images and stops when critical requirements fail. Administrators can customise the process through command-line options or a kubeadm configuration file.

During initialisation, kubeadm creates or uses a certificate authority, then writes certificates and kubeconfig files. Kubernetes uses certificates to secure HTTPS communication with the API server and authenticate components such as the scheduler, controller manager and kubelets. kubeadm stores certificate material under `/etc/kubernetes/pki`. Joining nodes trust the cluster through discovery information and CA public-key validation, not by receiving the CA private key.

kubeadm writes kubeconfig files under `/etc/kubernetes`. `admin.conf` gives cluster-administrator access. `super-admin.conf` can bypass the authorisation layer and should be treated as break-glass material. Component kubeconfig files allow the scheduler, controller manager and kubelet to locate and authenticate to the API server.

kubeadm also writes static pod manifests to `/etc/kubernetes/manifests`. The kubelet watches this directory and starts the control-plane pods described there. These static pods commonly include the API server, controller manager, scheduler and local etcd, unless the cluster uses external etcd. This design lets the control plane start after a reboot before the API server can schedule ordinary pods.

After the control-plane pods run, kubeadm labels and taints the control-plane node so normal application workloads do not schedule there by default. It then creates a bootstrap token for joining other nodes and deploys add-ons such as CoreDNS and kube-proxy. CoreDNS may remain pending until a pod network is installed.

Administrators copy `/etc/kubernetes/admin.conf` to the regular user's kubeconfig location when they need non-root `kubectl` access from the control-plane host. File ownership must match the user account that will run `kubectl`.

Administrators should also protect kubeconfig files and certificates. `admin.conf` grants broad control over the cluster, while `super-admin.conf` bypasses authorisation and carries higher risk. These files should remain on trusted hosts with restricted file permissions. Additional users should receive their own kubeconfig files and authorisation rules rather than shared administrator credentials.
## Pod networking
Kubernetes expects every pod to receive its own IP address and expects pods and nodes to communicate without application-level NAT assumptions. The cluster therefore needs a Container Network Interface plugin before normal workloads can run. Administrators choose a pod CIDR that does not overlap with node, service, data centre or cloud network ranges.

Direct routing can satisfy the Kubernetes networking model when the underlying network supports it. Overlay networking supplies another option by encapsulating pod traffic and presenting an apparent layer 3 network across nodes. Common CNI choices include Calico, Cilium and Flannel, with different policies, routing modes and operational features. The network plugin provides pod IP address management and node-level forwarding behaviour.

After `kubeadm init`, administrators apply the selected CNI manifest, such as a Calico manifest configured with the chosen pod CIDR. They then verify system pods across all namespaces. A healthy single control-plane lab shows the control-plane pods, CoreDNS, kube-proxy and the CNI pods in `Running` state, and the control-plane node in `Ready` state.
## Joining worker nodes
Each worker node receives the same base preparation as the control-plane node. After installing the runtime and Kubernetes packages, administrators generate or retrieve a join command from the control-plane node. `kubeadm init` prints an initial join command, and `kubeadm token create --print-join-command` can generate a fresh one. The default token from initialisation has a 24-hour time to live.

The join command points to the API server, presents a bootstrap token and includes the CA certificate hash for secure discovery. The worker runs `kubeadm join`, performs preflight checks, starts TLS bootstrap, submits a certificate signing request and receives configuration for its kubelet. The kubelet then registers the node with the API server.

After a worker joins, administrators check `kubectl get nodes`. The node should reach `Ready` after required DaemonSet pods, such as the CNI node agent and kube-proxy, start on it. Repeating this process for each worker completes the lab cluster.
## Validating workloads
A simple deployment proves that scheduling, container runtime integration, pod networking and services work together. Administrators can create a deployment with multiple replicas, then inspect pods with wide output to confirm that the scheduler spreads workloads across worker nodes. Exposing the deployment as a ClusterIP service proves in-cluster service discovery and load balancing.

A ClusterIP exists only inside the cluster network. Testing it from a node that can reach the service CIDR confirms internal access. Repeated requests should hit different backing pods when the service has multiple endpoints. After validation, administrators should remove the test deployment and service to return the cluster to a clean state.

A healthy validation run checks more than pod creation. Administrators should confirm that pod IPs come from the configured pod CIDR, service IPs come from the service CIDR, pods can reach each other across nodes and DNS resolution works through CoreDNS. These checks separate application faults from cluster plumbing faults and make later troubleshooting easier.
## Worker node maintenance
Maintenance includes operating system patching, Kubernetes upgrades and hardware or virtual machine changes. Administrators should drain a node before maintenance. `kubectl drain` marks the node unschedulable and evicts eligible pods. Controllers such as Deployments or ReplicaSets recreate replacement pods on other suitable nodes. DaemonSet-managed pods require `--ignore-daemonsets`, because DaemonSets intentionally run node-local pods.

`kubectl cordon` only marks a node unschedulable. It prevents new pods from landing on the node but leaves existing pods running. After maintenance, `kubectl uncordon` returns the node to scheduling.

Capacity planning matters during drains. The remaining nodes must have enough CPU, memory and placement flexibility to run the displaced workloads. Clusters should carry spare capacity for planned maintenance and unplanned failures. Rebooting a node without draining can delay pod recovery and may create avoidable application impact.
## Upgrading kubeadm clusters
A kubeadm upgrade follows a strict order. Administrators upgrade the first control-plane node, then any additional control-plane nodes, then worker nodes. Skipping minor versions is unsupported, so a cluster must move one minor version at a time. Patch upgrades within the same minor release are supported by the documented process. Release notes and changelogs must guide every upgrade, because API removals, feature changes and add-on requirements can affect workloads.

On the first control-plane node, administrators upgrade `kubeadm`, run `kubeadm upgrade plan`, then apply the selected target version with `kubeadm upgrade apply`. kubeadm runs checks, pulls images, renews certificates when required, updates static pod manifests and causes the kubelet to restart the control-plane pods with the new configuration. kubeadm stores backups under `/etc/kubernetes/tmp`, including manifest backups and local etcd backups where applicable.

After kubeadm upgrades the control plane, administrators drain the node for non-static workloads, upgrade `kubelet` and `kubectl`, reload systemd, restart kubelet, verify status and uncordon the node. Additional control-plane nodes use `kubeadm upgrade node` rather than `kubeadm upgrade apply`.

Worker node upgrades use a similar pattern. Administrators upgrade `kubeadm` on the worker, run `kubeadm upgrade node`, drain the node from the control-plane context, upgrade `kubelet` and `kubectl`, reload and restart kubelet, verify the node version and uncordon the node. The process repeats until every worker reports the target kubelet version and `Ready` status.

CNI providers may need their own upgrade steps. Administrators should check the network add-on documentation and upgrade it according to its supported path. They should also verify API changes, storage API versions and workload manifests after the cluster upgrade.
## Key corrections applied
- The command name is `kubeadm`, not `kubeadmin`.
- HTTPS is the API server transport, not HTPS.
- Kubernetes paths use `/etc/kubernetes`, `/etc/kubernetes/manifests`, `/etc/kubernetes/pki` and `/etc/kubernetes/tmp`.
- Joining nodes establish trust through discovery and TLS bootstrap. They do not receive the CA private key.
- Current Kubernetes package guidance uses `pkgs.k8s.io`, while legacy package repositories are deprecated.
- Linux swap can be configured, but kubelet still fails on swap by default unless explicitly configured to tolerate it.
- Worker node maintenance concerns draining and cordoning nodes, not worker domains.
- Upgrade commands use `kubeadm upgrade apply` for the first control-plane node and `kubeadm upgrade node` for additional control-plane and worker nodes.