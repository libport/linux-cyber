# Certified Kubernetes Administrator: Performing Cluster Version Upgrades
## Scope and prerequisites
Kubeadm upgrades apply to kubeadm-built clusters that use static control plane pods or external etcd. Administrators confirm shell access, working `kubectl` configuration, the correct package repository, current backups, release notes, spare CPU and memory, and disabled swap before starting. They upgrade the first control plane node, upgrade any extra control plane nodes one at a time, then upgrade worker nodes.

Kubernetes supports upgrades to the next minor version and to later patch versions within the same minor version. It does not support skipping minor versions. A cluster can move from 1.28 to 1.29, or from 1.29.1 to 1.29.5, but it should not jump from 1.27 to 1.29.
## Maintenance workflow
Administrators drain a node before planned maintenance:

```bash
kubectl drain <node> --ignore-daemonsets
```

Drain marks the node unschedulable and evicts regular workloads gracefully. DaemonSet pods remain. Pods that use local `emptyDir` data can require `--delete-emptydir-data`. Kubernetes replaces controller-managed pods on other schedulable nodes when capacity and disruption policies allow. Cordon only prevents new pods from landing on a node. It does not evict existing pods.

Administrators return a maintained node to service with:

```bash
kubectl uncordon <node>
```

A reboot without draining can leave pods unavailable while Kubernetes detects the node state and waits through the default not-ready or unreachable toleration period, often about five minutes. Draining provides planned failover and reduces outage risk.
## Upgrade the first control plane node
Administrators select the target version with the operating system package manager, then upgrade `kubeadm`:

```bash
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm='<version-pattern>'
sudo apt-mark hold kubeadm
kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v<target-version>
```

`kubeadm upgrade plan` checks cluster health, supported version transitions and component configuration. `kubeadm upgrade apply` repeats the checks, pulls images, renews kubeadm-managed certificates by default and rewrites static pod manifests in `/etc/kubernetes/manifests`. The kubelet watches that directory and restarts control plane static pods from the new manifests. Manifest backups belong outside the static pod directory, because the kubelet reads all non-dot files in that path.

Administrators then drain the control plane node for non-control plane workloads:

```bash
kubectl drain <control-plane-node> --ignore-daemonsets
```

They upgrade `kubelet` and `kubectl`, reload systemd, restart the kubelet and restore scheduling:

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet='<version-pattern>' kubectl='<version-pattern>'
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <control-plane-node>
```

They verify the result with:

```bash
kubectl version
kubectl get nodes
```
## Upgrade additional control plane nodes
Each extra control plane node follows the same package and drain process, but runs:

```bash
sudo kubeadm upgrade node
```

This command upgrades local configuration on that node. Administrators process one control plane node at a time to preserve availability.
## Upgrade worker nodes
Workers follow the upgraded control plane version. On each worker, administrators upgrade `kubeadm` and the node configuration:

```bash
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm='<version-pattern>'
sudo apt-mark hold kubeadm
sudo kubeadm upgrade node
```

They drain the worker from a control plane node:

```bash
kubectl drain <worker-node> --ignore-daemonsets
```

They return to the worker and upgrade `kubelet` and `kubectl`:

```bash
sudo apt-mark unhold kubelet kubectl
sudo apt-get update
sudo apt-get install -y kubelet='<version-pattern>' kubectl='<version-pattern>'
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

They bring the worker back into scheduling and confirm version alignment:

```bash
kubectl uncordon <worker-node>
kubectl get nodes
```

Administrators repeat the worker process node by node. A successful upgrade leaves every node in `Ready` state on the target Kubernetes version.
## Technical accuracy
- Kubernetes uses the terms `unschedulable` and `schedulable`.
- The static manifest directory is `/etc/kubernetes/manifests`.
- The first control plane node uses `kubeadm upgrade apply`, while additional control plane nodes and workers use `kubeadm upgrade node`.
- Current Kubernetes package guidance favours `pkgs.k8s.io` over the frozen legacy repositories.
- Recreated pods receive new UIDs. Kubernetes replaces pods through their controllers rather than moving the same pod object to another node.