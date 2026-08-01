# Kubernetes Installation and Configuration Fundamentals
## Deployment model
Kubernetes can run on local development systems, cloud virtual machines, managed cloud services, on-premises virtual machines, or bare-metal servers. Desktop distributions suit learning and application development. Managed services shift much of the control plane and infrastructure operation to a provider, although the customer still manages workloads, access, policy, and the responsibilities defined by the service boundary.

Self-managed clusters provide greater control and require the organisation to operate the hosts, container runtime, networking, Kubernetes components, upgrades, security, and recovery. `kubeadm` bootstraps and joins nodes in this model. It does not provision hosts, install a pod network, create a production high-availability design, or manage the cluster after installation.

One control plane node and three workers form a useful lab. A single control plane node creates a single point of failure, so a production design normally uses multiple control plane nodes, a stable API endpoint, and a deliberate etcd recovery strategy.
## Host and software requirements
Every node needs a compatible Linux system, full network connectivity, the required open ports, and unique hostnames, MAC addresses, and product UUIDs. Each machine should have at least 2 GB of RAM. A control plane machine needs at least two CPUs. Production capacity must also cover system overhead, application requests, failure tolerance, and growth.

Node addresses and name resolution should remain stable. An `/etc/hosts` file can serve a small lab, while production environments generally need managed DNS and address allocation. Firewalls should allow the documented Kubernetes ports and any additional traffic required by the CNI implementation. Administrators should open the necessary paths between defined sources and destinations instead of disabling host or network firewalls. Every node also needs access to the selected package repositories and image registries, either directly or through controlled mirrors and proxies.

The hosts require the following software:

| Component | Purpose | Placement |
| --- | --- | --- |
| CRI runtime | Starts and manages containers | Every node |
| `kubelet` | Runs pods and reports node state | Every node |
| `kubeadm` | Initialises a control plane or joins a node | Every node built with kubeadm |
| `kubectl` | Sends administrative requests to the API server | Administrative systems only |

The runtime must support CRI v1. Containerd and CRI-O are common choices. Docker Engine requires a separate CRI adapter because Kubernetes no longer includes dockershim. On a systemd host, the kubelet and runtime should use the `systemd` cgroup driver. Kubeadm defaults the kubelet to this driver, while containerd must use the configuration path that matches its installed major version. CRI support must remain enabled in containerd.

IPv4 forwarding is commonly required. Kernel modules and bridge-related sysctls depend on the runtime, CNI implementation, kernel, and data plane. An administrator should follow the selected runtime and network provider's current instructions instead of treating `overlay`, `br_netfilter`, or bridge sysctls as universal kubeadm requirements.

The kubelet rejects an enabled swap configuration by default. The simplest lab configuration disables swap at runtime and in `/etc/fstab`. Current Kubernetes releases can use swap when the administrator explicitly configures `failSwapOn: false` and an appropriate `memorySwap.swapBehavior`. That choice requires an assessment of performance and the risk that sensitive tmpfs content could reach disk.
## Version and package management
An administrator should select a supported Kubernetes minor release and use its matching repository at `pkgs.k8s.io`. Each minor release has a separate repository. The administrator should install matching `kubeadm`, `kubelet`, and, where required, `kubectl` packages, then pin them to prevent an ordinary operating-system update from performing an unsupported Kubernetes upgrade.

Package holds do not replace maintenance. Kubernetes upgrades require a planned sequence, and security fixes arrive through supported patch releases. The kubelet must not run a newer minor version than the API server, although it may run up to three minor versions older. `kubectl` remains supported within one minor version of the API server. Administrators should normally install the newest patch release in the chosen supported minor line.

The same declared release should drive package installation and control plane initialisation. An administrator should record exact package versions, repository configuration, runtime version, CNI release, and a sanitised kubeadm configuration in automation or source control. Tokens and private keys belong in a secrets system. This record supports consistent worker builds, controlled upgrades, and recovery when a node must be replaced.

Before bootstrap, containerd should run successfully. The kubelet may restart repeatedly while it waits for kubeadm to supply its configuration. An inactive kubelet is not a universal sign of correct preparation.
## Control plane bootstrap
`kubeadm init` creates the first control plane node. A version-controlled kubeadm configuration file provides a repeatable way to define the Kubernetes version, API endpoint, pod CIDR, service CIDR, runtime socket, and component settings. The pod CIDR must match the chosen network add-on and must not overlap node, service, corporate, VPN, or cloud networks.

The API endpoint should use a stable DNS name or virtual address when the cluster may later gain more control plane nodes. Initialising against a single node's temporary address can complicate a later high-availability conversion because certificates and kubeconfig files embed API server endpoints. Pre-pulling images with `kubeadm config images pull` can expose registry or proxy failures before initialisation begins.

```bash
sudo kubeadm init --config kubeadm-config.yaml
```

The initialisation workflow performs these principal actions:
1. It runs preflight checks and obtains the required control plane images.
2. It creates or uses the cluster public key infrastructure in `/etc/kubernetes/pki`.
3. It writes component kubeconfig files under `/etc/kubernetes`.
4. It writes static Pod manifests under `/etc/kubernetes/manifests`.
5. It starts the local control plane and waits for it to become healthy.
6. It labels and taints the node, configures node bootstrap, and creates a join token.
7. It deploys CoreDNS and kube-proxy through the API server.

With local etcd, the static manifests define the API server, controller manager, scheduler, and etcd Pods. The kubelet watches the manifest directory and starts those Pods without relying on the API server. After a reboot, systemd starts the kubelet, and the kubelet restores the static Pods from disk.

Kubeadm applies a `NoSchedule` taint to the control plane node. This blocks Pods that lack a matching toleration, rather than creating an absolute ban on application workloads. An administrator can remove the taint for a constrained single-node lab, but separating applications from control plane components improves resilience.
## Certificates and kubeconfig files
Kubeadm creates a cluster certificate authority by default, or it can use an externally managed CA. Server and client certificates provide authenticated TLS connections between components. The CA private key remains protected on the control plane and is not distributed to worker nodes. A joining worker retrieves and verifies the CA certificate, creates its own key, and obtains a signed kubelet client certificate through TLS bootstrap.

The principal kubeconfig files include `admin.conf`, `controller-manager.conf`, `scheduler.conf`, and `kubelet.conf`. Current kubeadm releases also create `super-admin.conf`. `admin.conf` grants broad cluster administration through RBAC. `super-admin.conf` uses a break-glass identity that bypasses normal authorisation. Both files require strict access controls, secure transfer, and careful rotation.

An administrator can configure `kubectl` for a non-root account after `kubeadm init` succeeds:

```bash
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
```

This copy contains powerful credentials. A production environment should issue purpose-specific identities and RBAC permissions instead of distributing `admin.conf` to routine users.
## Pod networking
Kubernetes expects every Pod to receive a cluster-wide IP address and, unless policy intentionally restricts traffic, allows Pods to communicate directly without proxies or network address translation. Kubernetes does not implement this network by itself. A compatible CNI-based network add-on supplies address management, routing or encapsulation, and, when supported, NetworkPolicy enforcement.

Not every network add-on uses an overlay. Calico, for example, supports several routed and encapsulated designs. The administrator should select a design that fits the underlying network, security controls, performance goals, supported IP families, and operational skills.

The administrator should obtain a release-pinned manifest from the provider, review its settings, confirm its integrity, and apply it from a system with suitable kubeconfig credentials. The file commonly contains many Kubernetes resources, not one Pod definition.

```bash
kubectl apply -f cni.yaml
kubectl get pods -A --watch
kubectl get nodes
```

Kubeadm deploys CoreDNS during initialisation, but CoreDNS normally remains unscheduled until the pod network is available. A cluster should use one pod network implementation. The control plane node should reach `Ready`, and the CNI, CoreDNS, kube-proxy, and control plane Pods should run successfully before workers join.
## Joining worker nodes
Each worker needs the same host preparation, a compatible CRI runtime, `kubelet`, and `kubeadm`. It does not need `kubectl` unless an administrator will manage the cluster from that host. The installed versions must comply with the cluster's version-skew policy.

Kubeadm prints a worker join command after successful initialisation. If that command or token expires, a control plane administrator can generate a new command:

```bash
sudo kubeadm token create --print-join-command
```

The default bootstrap token expires after 24 hours and should be handled as a secret. The generated command includes the API endpoint, token, and SHA-256 hash of the cluster CA public key. The worker runs the complete command with elevated privileges.

During `kubeadm join`, the worker verifies the cluster information against the pinned CA public key, starts TLS bootstrap, generates a local key pair, submits a certificate signing request, and receives an automatically approved kubelet client certificate under the default kubeadm policy. Kubeadm writes `/etc/kubernetes/kubelet.conf`, and the kubelet then authenticates to the API server as the new node.

The node can remain `NotReady` briefly while the runtime downloads images and the network DaemonSet configures the host. Persistent `NotReady` status does not prove a CNI fault. An administrator should inspect node conditions, events, kubelet logs, runtime health, CNI Pods, routes, firewall rules, DNS, and time synchronisation before assigning a cause.
## Validation and ongoing operation
A successful installation shows every expected node as `Ready` and all required system Pods as healthy. Validation should also test Pod-to-Pod connectivity across nodes, service routing, cluster DNS, image pulls, workload scheduling, and recovery after a node restart. Static Pod files and kubeconfig files confirm the control plane layout, but their presence alone does not establish cluster health.

Useful checks include `kubectl get nodes -o wide`, `kubectl get pods -A`, `kubectl describe node`, and `journalctl -u kubelet`. Cluster events can reveal image, scheduling, certificate, and network failures. Administrators should preserve command output and relevant logs when diagnosing a failed bootstrap, then correct the cause before rerunning kubeadm. If a partial attempt requires cleanup, `kubeadm reset` performs a best-effort local reset and may leave CNI configuration or network rules that need separate removal.

Kubeadm completes bootstrap, not lifecycle management. Administrators must still protect PKI assets, back up etcd, monitor capacity and component health, patch hosts, upgrade Kubernetes in the supported order, renew certificates, test recovery, and design high availability for production use.