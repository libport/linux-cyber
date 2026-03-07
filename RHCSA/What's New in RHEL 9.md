# What's New in RHEL 9
Red Hat Enterprise Linux 9 is an incremental platform update rather than a full redesign. It keeps the core administration model from RHEL and updates the platform around newer software, tighter defaults, and cleaner tooling. The most important changes sit in four areas: the base operating system, the RHCSA study focus, storage through LVM-VDO, and container management with Podman and systemd.

RHEL 9 shipped with the 5.14 Linux kernel. It also standardised newer language runtimes, including Python 3.9 and Ruby 3.0. These updates improve support for current automation and configuration tools such as Ansible, Puppet, Chef, and Salt. The distribution also removed the old `network-scripts` package and pushed network configuration fully into NetworkManager. In SSH, the default root login policy became `PermitRootLogin prohibit-password`, which blocks password-based root logins over SSH and leaves key-based access as the safer default.
## Core platform changes
RHEL 9 reports version details through the same tools used in other current systemd-based distributions. `hostnamectl` gives a concise overview of the host, operating system, kernel, architecture, and virtualisation context. `uname -r` returns only the kernel release, which makes it easier to reuse in shell expansion and scripts. `/etc/os-release` remains the portable source of distribution metadata and can be sourced directly when scripts need values such as `NAME` or `VERSION`.

Typical inspection commands include:

```Bash
hostnamectl
uname -r
cat /etc/os-release
```

These commands answer slightly different questions. `hostnamectl` suits human inspection. `uname -r` suits shell substitution. `/etc/os-release` suits automation and cross-distribution checks.

Python 3.9 is the default Python implementation. Ruby 3.0 is available as the current Ruby stream. Automation, DevOps tooling, and configuration management often depend on Python and Ruby. Python supports common administration workflows and Ansible-based automation. Ruby still underpins tools and ecosystems such as Chef and Puppet.
## Networking and SSH
RHEL 9 removed the old `network-scripts` package from the default networking workflow. Earlier releases still carried `ifcfg` files under `/etc/sysconfig/network-scripts/`, but RHEL 9 expects network connection profiles under `/etc/NetworkManager/system-connections/`. That change does not alter the administrator's main tools as much as it alters the underlying configuration files. `nmcli`, the text user interface, and the graphical tools remain the standard front ends.

Network configuration now belongs to NetworkManager end to end. Administrators should stop treating `ifcfg` files as the source of truth and should instead work through NetworkManager-aware tools. That shift removes ambiguity and aligns the stored connection data with the tools that already manage it.

SSH hardening is another small but important change. RHEL 9 sets `PermitRootLogin` to `prohibit-password` by default. That default blocks brute-force attempts against the known `root` account and still permits key-based logins when policy requires them. Administrators can harden further by disabling remote root login entirely and using `sudo` after login.

## Command-line habits that matter
Several shell shortcuts matter because they reduce repetitive typing and limit avoidable mistakes:
- `sudo !!` reruns the previous command with privilege
- `Esc .` inserts the last argument from the previous command
- `Ctrl+R` searches command history
- `Ctrl+A` moves to the start of the line
- `Ctrl+E` moves to the end of the line

These shortcuts are standard shell habits that speed up routine administration. They also improve accuracy by reducing retyping of long paths, device names, and file names.

Newer `sudo` versions in RHEL 9 are less likely to lock an administrator out after a syntax error in `sudoers`. That change improves resilience in real systems, although it does not replace disciplined editing with `visudo`.
## LVM-VDO in RHEL 9
The biggest change in the storage section is VDO's move into LVM. In RHEL, administrators used separate VDO tooling. In RHEL 9, they create VDO logical volumes inside the LVM stack. This makes VDO part of a standard storage workflow rather than a parallel feature set.

VDO combines two main ideas:
- compression, which stores data in a smaller form
- deduplication, which stores identical data once and reuses it

These features help most when data repeats heavily, such as on virtual machine hosts or in estates that store many similar images. The logical design follows the same broad pattern as thin provisioning. A VDO logical volume sits on top of a VDO pool logical volume, and the kernel component handles the compression and deduplication work.

A clean build needs the right packages before any storage work begins. The material identifies `lvm2`, `vdo`, and `kmod-kvdo` as the practical prerequisites. In many lab systems they are already installed, but administrators still need to recognise them because a minimal build or exam machine may not include everything by default. If the kernel module update requires a newer kernel, a reboot may also become part of the setup path.

A basic workflow looks like this:
- install the required packages
- create or identify storage for a volume group
- create a volume group
- create a VDO logical volume with `lvcreate --type vdo`
- assign a virtual size larger than the physical allocation
- create a file system on the resulting logical volume
- mount and use the file system
- monitor real pool consumption rather than trusting file system size alone

```Bash
fallocate -l 10G /root/disk1
losetup -f --show /root/disk1
vgcreate myvg /dev/loop0
lvcreate --type vdo --name myvdolv --extents 100%FREE --virtualsize 50G myvg
mkfs.xfs -K /dev/myvg/myvdolv
```

This example creates 10 GB of physical backing storage and presents 50 GB of virtual capacity. That oversubscription only works because VDO expects compression and deduplication to reduce real consumption.
## Monitoring and managing VDO
VDO needs different monitoring habits from ordinary file systems. A mounted file system only shows the logical space that it presents to applications. It does not show the real level of pool consumption clearly enough for capacity planning. Administrators need to track the physical pool as well.

`vdostats` remains the key monitoring tool. It shows how much real storage the VDO pool is using. That view matters more than `df` when the system stores repeated data. `df` may show rapid growth because it counts each copied file as a new logical allocation. VDO may show much slower physical growth because deduplication stores repeated blocks once.

The video demonstrates the point by copying the same 105 MB file multiple times into the VDO-backed file system. The file system usage rises sharply because each file appears distinct at the file level. The VDO pool usage barely rises because the underlying data is identical.

Administrators can also inspect VDO settings directly from LVM metadata. Compression and deduplication are enabled by default, and `lvs` can report them with additional fields. `lvchange` can enable or disable them at the pool level when workloads need different behaviour. Not every workload benefits equally from both features. Repeated images and similar datasets benefit strongly. Data that is already compressed may not.

Check the pool, not just the mount point, to avoid over-committing the physical store.
## Podman and systemd
RHEL 9 strengthens the relationship between Podman and systemd in two directions. It supports running containers as systemd-managed services on the host. It also supports running systemd inside a container when a use case needs multiple managed services in that container.

The first pattern is the more common one. A host runs a container, then systemd starts, stops, and restarts it like any other service.

The host-side workflow is:
- install Podman
- create a host directory for persistent data
- run the container in detached mode
- bind-mount the host directory into the container with the correct SELinux labelling
- set the root password variable required by the image
- test the container from inside with `podman exec`
- generate a systemd service file
- reload systemd
- enable the service so the container starts at boot

A simplified sequence looks like this:

```Bash
dnf install podman
mkdir -p /local/mysql
podman run -d --name mariadb -v /local/mysql:/var/lib/mysql:z IMAGE
podman exec -it mariadb /bin/bash
podman generate systemd mariadb > /etc/systemd/system/mariadb.service
systemctl daemon-reload
systemctl enable mariadb.service
```

Persistent data should live on the host. SELinux labels must allow the container to use the mount. The container should be tested before the unit file is generated. After that, systemd can manage the container like a native service. Once the unit exists, `systemctl stop`, `start`, and `enable` become the normal host-side control surface.
## Running systemd inside containers
Running systemd inside a container is a specialised pattern. A normal container should run one foreground process and keep its scope narrow. A container that runs `httpd`, `atd`, and other services under systemd is heavier and less idiomatic. Even so, it has legitimate uses in testing, automation validation, and environments that need a fuller service model.

This pattern is a DevOps testing technique. Playbooks or configuration code may expect systemd to exist. A container that boots with systemd makes those workflows easier to test without standing up full virtual machines.

The process has four main parts.

First, the host must permit the container to manage cgroups:

```Bash
setsebool -P container_manage_cgroup on
```

Second, the image needs a `Dockerfile` that installs systemd and the services required for the test. The example uses Fedora as the base image and installs `systemd`, `httpd`, and `at`. It then enables `httpd` and `atd`, exposes port 80, and sets `/usr/sbin/init` as the startup command.

Third, Podman builds the image and runs a container from it with the appropriate port mapping.

Fourth, the administrator verifies that systemd is process 1 and that the expected services are running. `curl` confirms the web service. `podman container top` confirms the process tree.

Systemd inside a container is justified only when the container must behave like a small service host. When a single process will do, a simpler container is better.
## Practical conclusions
RHEL 9 updates the kernel and language runtimes, removes outdated networking practices, hardens SSH defaults, moves VDO into LVM, and gives Podman a tighter relationship with systemd.

For administrators, the preferred path is:
- inspect the system with the standard current tools
- treat NetworkManager as the single source of truth for networking
- block password-based root SSH access
- build storage with LVM and use VDO through LVM where compression and deduplication add value
- monitor VDO pools with `vdostats`
- manage long-running containers with systemd on the host
- run systemd inside containers only for specific testing or service-hosting needs
- rely on fluent shell habits to work faster and more accurately