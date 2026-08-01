# Certified Kubernetes Administrator: Performing Cluster Version Upgrades
> [!NOTE]
> This guide outlines a safe, sequential `kubeadm` upgrade process covering preparation, backups, node draining, control-plane and worker upgrades, version compatibility, and post-upgrade verification.
## Prepare the cluster
Kubeadm upgrades a cluster in this order: the first control-plane node, each additional control-plane node, and then the worker nodes. An administrator must not skip a minor release. Before changing the cluster, the administrator should:
- Read the target release notes, deprecations, version-skew policy, and CNI upgrade instructions.
- Move to the latest patch release of the current minor version before a minor upgrade.
- Back up etcd, application data, configuration, and other critical state, then test recovery.
- Confirm that the control plane, nodes, workloads, networking, and monitoring are healthy.
- Preserve enough CPU, memory, storage, and replica capacity during each maintenance window.
- Change the `pkgs.k8s.io` repository when moving to a new minor release.

Package versions use two forms. The operating-system package manager selects a full package version, while `kubeadm upgrade apply` receives the Kubernetes semantic version. The administrator should inspect available packages rather than assume a release exists, pin every selected package, and record the before-and-after state. Patch upgrades remain within one minor repository, whereas minor upgrades require the next repository. The cluster upgrade changes Kubernetes internals, not application workloads, but control-plane and node restarts can still directly affect availability, admission webhooks, operators, and add-ons throughout the planned maintenance window.

The kubelet must not run a newer version than any API server it contacts. Supported temporary skew enables a rolling upgrade, but matching versions after completion simplifies operation. Administrators should upgrade one control-plane node at a time and workers individually or in small capacity-safe batches.
## Drain nodes safely
`kubectl cordon <node>` blocks new scheduling but leaves existing pods on the node. `kubectl drain <node> --ignore-daemonsets` also requests safe eviction of eligible pods and respects PodDisruptionBudgets. DaemonSet pods remain. Mirror pods remain, while unmanaged pods and pods using `emptyDir` need assessment and may require flags.

A successful drain confirms that the command has safely evicted the pods within its scope. Rebooting without a drain can interrupt workloads until node-health taints and pod tolerations trigger replacement. The common five-minute delay comes from automatically added tolerations, but workloads and cluster configuration can change it. After maintenance and health checks, `kubectl uncordon <node>` restores scheduling.
## Upgrade the control plane
The first control-plane node uses the following sequence:
1. Unhold and upgrade `kubeadm` to the selected package version, then hold it again.
2. Verify the binary with `kubeadm version`.
3. Run `sudo kubeadm upgrade plan` and review every preflight, health, configuration, and version check.
4. Apply the approved version with `sudo kubeadm upgrade apply <version>`.
5. Upgrade the CNI at the point required by its provider.
6. Drain the node before a minor kubelet upgrade.
7. Unhold, upgrade, and hold `kubelet` and `kubectl`.
8. Reload systemd, restart the kubelet, verify health, and uncordon the node.

`kubeadm upgrade apply` pulls required images, enforces version policy, updates component configuration, and replaces the static pod manifests in `/etc/kubernetes/manifests`. The kubelet detects those changes and restarts the API server, controller manager, scheduler, and local etcd as required. Kubeadm renews its managed certificates by default.

Kubeadm stores upgrade backups under `/etc/kubernetes/tmp`, including dated etcd and manifest backup directories where applicable. These recovery files do not replace independent etcd backups. Additional control-plane nodes follow the same package, drain, kubelet, and verification process, but each runs `sudo kubeadm upgrade node` instead of `kubeadm upgrade apply`.
## Upgrade worker nodes
Each Linux worker follows this order:
1. Upgrade and re-hold `kubeadm` to match the upgraded control plane.
2. Run `sudo kubeadm upgrade node` to fetch cluster configuration and update the local kubelet configuration.
3. Drain the worker from an authorised administrative host.
4. Upgrade and re-hold `kubelet` and `kubectl` if installed.
5. Run `systemctl daemon-reload`, restart the kubelet, and confirm that it remains active.
6. Uncordon the worker and verify its readiness before continuing.

For a minor kubelet upgrade, draining must precede the kubelet package change. PodDisruptionBudgets may pause a drain, while insufficient capacity can leave replacements Pending. Both require remediation rather than bypass.
## Verify completion
After every node reaches the target version, the administrator should confirm node readiness, system pod health, application availability, DNS, Service routing, CNI operation, storage, alerts, and logs. The cluster should remain under observation before the maintenance window closes. If an upgrade stops unexpectedly, kubeadm can often resume its idempotent workflow, but recovery should follow the official procedure and the captured failure state.