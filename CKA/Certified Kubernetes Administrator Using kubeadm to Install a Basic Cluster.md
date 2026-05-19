# Certified Kubernetes Administrator: Using kubeadm to Install a Basic Cluster
## Installing Kubernetes with kubeadm
`kubeadm` provides a practical path for creating a basic Kubernetes cluster on Linux hosts. It does not design the wider platform, provision infrastructure, configure production high availability, or operate workloads. It bootstraps the control plane, creates the join process for other nodes, and leaves administrators to choose the infrastructure, container runtime, network add-on, security model, and upgrade process.

A self-managed cluster suits teams that need control over the host operating system, runtime configuration, network layout, and Kubernetes version. Managed Kubernetes suits organisations that want a cloud provider to operate much of the control plane and platform plumbing. Infrastructure as a Service deployments sit between those models. The organisation runs Kubernetes on cloud virtual machines, while the provider manages the physical infrastructure and hypervisor.

A simple lab can run on virtual machines with one control plane node and three worker nodes. Production clusters need stronger design, including multiple control plane nodes, backups, monitoring, tested upgrades, security controls, capacity planning, and recovery procedures.
## Host requirements
Each node needs a compatible Linux host, network reachability to the other nodes, access to the required package and image repositories, and a Container Runtime Interface runtime such as `containerd` or CRI-O. Docker Engine can support Kubernetes only through the external `cri-dockerd` adapter.

For a small lab, each node should have at least 2 GB of memory. Control plane machines need at least two CPUs. Production sizing must follow the application workload, add-ons, logging, monitoring, storage, and failure tolerance requirements.

Each node must have a unique host name, MAC address, and product UUID. DNS can provide name resolution, or administrators can use consistent host file entries in small labs. The hosts must use non-overlapping address ranges for node addresses, Service addresses, and Pod addresses.

The simplest lab disables swap on Linux nodes. Current Kubernetes can support limited swap only when administrators deliberately configure kubelet swap behaviour. By default, the kubelet fails when it detects unmanaged swap.

When the host uses systemd, administrators should align the kubelet and runtime with the systemd cgroup driver, unless the Kubernetes and runtime versions support automatic cgroup driver detection.
## Packages and node preparation
Administrators install the same core Kubernetes packages on each node:
- `containerd` or another supported CRI runtime
- `kubelet`, the node agent that runs Pods and reports node status
- `kubeadm`, the bootstrap and join tool
- `kubectl`, the command-line client for the Kubernetes API

On Debian and Ubuntu hosts, administrators add the Kubernetes package repository, install the packages, then hold the Kubernetes packages with `apt-mark hold`. Holding these packages prevents normal operating system patching from changing Kubernetes versions outside the supported upgrade process.

The runtime also needs kernel and networking preparation. Typical `containerd` preparation loads the `overlay` and `br_netfilter` modules, sets the required `sysctl` values, creates a default `/etc/containerd/config.toml`, configures the systemd cgroup driver, and restarts `containerd`.

Before cluster creation, `containerd` should run under systemd. The `kubelet` service may appear loaded and enabled but inactive, because it has no cluster configuration or static Pod manifests yet.
## Version and repository control
Kubernetes version choices should remain deliberate. Administrators should install a specific minor release stream from the Kubernetes package repository, then install a selected patch version where repeatability matters. The kubelet, kubeadm, and kubectl packages should stay close to the control plane version and within the supported version skew policy.

Normal operating system patching should not silently upgrade Kubernetes packages. A controlled Kubernetes upgrade starts with planning, reading the release notes, upgrading kubeadm, applying the kubeadm upgrade workflow, then upgrading kubelet and kubectl on each node. A lab can use exact package versions to make later upgrade exercises predictable.
## How kubeadm creates the control plane
Administrators create the first control plane node with `kubeadm init`. The command runs a set of phases that prepare and start a minimum viable Kubernetes control plane.

`kubeadm init` first performs preflight checks. These checks verify permissions, required ports, host resources, runtime availability, and other conditions that would block cluster creation. If a critical check fails, `kubeadm` stops and reports the error.

It then creates cluster certificates, writes kubeconfig files, generates static Pod manifests for control plane components, and asks the kubelet to start those static Pods. The main control plane Pods include the API server, controller manager, scheduler, and local etcd instance. `kubeadm` waits until these components run.

`kubeadm` also taints the control plane node so normal application Pods do not run there by default. It creates a bootstrap token for node joins and installs core add-ons such as CoreDNS and kube-proxy. CoreDNS may remain pending until the Pod network add-on runs.

The default process can be changed through command-line flags or a kubeadm configuration file. A configuration file gives administrators a clearer and more repeatable way to declare version, endpoint, Pod CIDR, certificate, and networking choices.
## Certificates, kubeconfig files, and static Pods
By default, `kubeadm` creates a self-signed certificate authority for the cluster. Organisations can integrate an external public key infrastructure when policy requires it. The cluster certificate authority secures API server traffic and signs certificates used by administrators, kubelets, and control plane components.

Control plane certificates live under `/etc/kubernetes/pki`. Kubeconfig files live under `/etc/kubernetes`. These files define the API server endpoint and the credentials that a component or user uses to authenticate.

Important kubeconfig files include:
- `admin.conf`, the cluster administrator kubeconfig
- `super-admin.conf`, a break-glass kubeconfig that bypasses the authorisation layer
- `controller-manager.conf`, used by the controller manager
- `scheduler.conf`, used by the scheduler
- `kubelet.conf`, used by the kubelet on the node

Administrators usually copy `/etc/kubernetes/admin.conf` to `$HOME/.kube/config` for the first administrative account on the control plane node. `super-admin.conf` should stay tightly protected and should not become a routine user credential.

Static Pod manifests live in `/etc/kubernetes/manifests`. The kubelet watches that directory and starts any Pods defined there. This design lets the API server, controller manager, scheduler, and local etcd start as Pods even before the Kubernetes API can schedule normal workloads. After a reboot, systemd starts the kubelet, and the kubelet recreates the static control plane Pods from those manifests.
## Pod networking
Kubernetes requires each Pod to receive its own cluster-wide IP address. Pods should communicate with other Pods without Network Address Translation, and nodes must reach Pods across the cluster. The platform does not install a Pod network add-on automatically.

Administrators must select and install a Container Network Interface add-on. Common choices include Calico, Cilium, and Flannel. Some environments use direct routing. Others use overlay networking, which encapsulates traffic to provide Pod-to-Pod connectivity across an underlying network that does not directly route all Pod addresses.

The Pod CIDR must not overlap with node networks, Service CIDRs, cloud networks, VPN routes, or on-premises routes. Network teams should reserve the Pod range before installation. When using Calico, administrators must ensure the Calico address pool matches the Pod CIDR chosen for the cluster.
## Lab topology
A clear lab topology reduces troubleshooting noise. A useful naming scheme gives the control plane and workers predictable names, such as `c1-cp1`, `c1-node1`, `c1-node2`, and `c1-node3`. Static host file entries can replace DNS in a small private lab, provided every node uses the same names and addresses.

The control plane node can also act as the administrative workstation for the lab. Installing and configuring `kubectl` there lets the administrator run cluster checks immediately after `kubeadm init`. Larger environments should manage access from dedicated administration hosts or secure workstations rather than relying on shell access to a control plane node.

Generous disk allocation helps avoid avoidable lab rebuilds. Disk size is usually harder to change than CPU or memory after virtual machine creation. Production disk design should cover container image storage, logs, etcd data, monitoring agents, and operating system growth.
## Creating the first node
A repeatable control plane build follows this sequence:
- Prepare the Linux host, runtime, kernel modules, package repositories, and Kubernetes packages.
- Choose the Kubernetes version, API server endpoint, and Pod CIDR.
- Run `kubeadm init` with the selected version and configuration.
- Copy `admin.conf` to `$HOME/.kube/config` for the administrative user.
- Install the CNI add-on with `kubectl apply` or the vendor-supported installation method.
- Confirm that the control plane Pods and network Pods are running.
- Confirm that the control plane node reports `Ready`.

Useful checks include:

```bash
kubectl get pods --all-namespaces
kubectl get nodes
systemctl status kubelet
systemctl status containerd
```

A healthy control plane shows running control plane Pods, running network add-on Pods, and a `Ready` node. After `kubeadm init`, the kubelet service changes from inactive to active because the static Pod manifests now give it work to run.
## Joining worker nodes
Worker nodes run application workloads. Each worker needs the same runtime and Kubernetes packages as the control plane node. After preparation, administrators run `kubeadm join` on the worker.

The join command printed at the end of `kubeadm init` is convenient but temporary. Administrators should treat it as sensitive because it contains a bootstrap token. Tokens expire by default, and expired or exposed tokens should be replaced rather than reused.

A join command includes the API server endpoint, a bootstrap token, and the CA certificate hash. The token gives the joining node temporary authentication to the API server. The CA certificate hash pins trust to the expected cluster certificate authority and protects the discovery step.

During the join process, the node downloads cluster metadata, submits a certificate signing request for the kubelet, receives an approved client certificate, and stores kubelet credentials under `/var/lib/kubelet/pki`. `kubeadm` also creates `/etc/kubernetes/kubelet.conf`, which tells the kubelet how to reach and authenticate to the API server.

A newly joined node can show `NotReady` while the CNI add-on deploys its node-level components and pulls container images. Once the network components start, the node should change to `Ready`.

Administrators can print a fresh join command from the control plane when the original token expires:

```bash
kubeadm token create --print-join-command
```
## Operating the lab cluster
A basic kubeadm lab cluster provides a sound foundation for learning Kubernetes administration. It demonstrates how the runtime, kubelet, static Pods, certificates, kubeconfig files, control plane add-ons, CNI networking, and node bootstrap process fit together.

The same cluster should not be treated as a production pattern without further design. Production environments need high availability, tested certificate and backup procedures, controlled upgrades, image and package supply controls, network policy, observability, and documented recovery steps.