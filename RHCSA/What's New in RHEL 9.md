# What's New in RHEL 9
## Red Hat Enterprise Linux 9
Red Hat Enterprise Linux 9 (RHEL 9) refreshes the enterprise platform with newer core components and tighter defaults while keeping most administration workflows consistent with Red Hat Enterprise Linux 8. The release arrived on 17 May 2022 and concentrates on updated versions, clearer support boundaries, and container operations that align more closely with systemd service management.

RHEL 9 prioritises predictable operations over novelty. The distribution keeps systemd as the init system and retains NetworkManager as the network configuration framework, then removes older configuration paths that previously remained for compatibility.
## Platform changes that affect administrators
### Incremental change rather than a new operating model
Enterprise Linux releases sometimes introduce major operational shifts. Earlier generations moved from SysV init to Upstart, then to systemd. RHEL 8 introduced a noticeable increase in container and web based administration tooling. RHEL 9 continues that direction but does not introduce a new service manager or a new configuration paradigm. Administrators mostly apply existing habits, then update the places they look for configuration data and the commands they use to integrate newer features.
### Kernel baseline and verification
RHEL 9 standardises on the 5.14 kernel series. This baseline matters for hardware enablement, driver coverage, and the behaviour of kernel dependent features. Administrators confirm the running release early to avoid troubleshooting the wrong system state.

hostnamectl provides a systemd aware summary that includes the operating system release, kernel version, architecture, and virtualisation details.

```bash
hostnamectl
```

uname -r prints only the kernel release string. Scripts and shell users often prefer this command because it produces a single value that fits well into substitutions.

```bash
uname -r
```

RHEL 9 uses 5.14 as the kernel baseline and does not use a 5.4 kernel baseline. Any system reporting 5.4 indicates a different platform release or an incorrect assumption about the host.

The /etc/os-release file provides a portable mechanism for obtaining release metadata. The file uses key value pairs that shell scripts can parse or source directly.

```bash
cat /etc/os-release
. /etc/os-release
echo "$NAME"
echo "$VERSION"
```
### Practical use of release values in scripts
Administrators often reuse the kernel release string as an input to other commands. The module tree under /usr/lib/modules uses the kernel release as a directory name, so the release string can select the correct module directory without hard coding.

```bash
ls /usr/lib/modules/$(uname -r)
```

The /etc/os-release file also supports lightweight scripting. Sourcing the file loads variables into the current shell session, which allows scripts to report the distribution name and version consistently across environments.

```bash
. /etc/os-release
printf '%s %s\n' "$NAME" "$VERSION"
```
### Updated scripting runtimes
RHEL 9 updates common automation runtimes used in configuration management and operational tooling.

- Python 3.9 provides a modern default runtime and improves compatibility with current Ansible ecosystems and libraries that rely on Python 3 only support.
- Ruby 3.x supports tools that depend on Ruby, including common configuration management frameworks.

Administrators verify installed runtime versions during environment validation and when diagnosing tool behaviour.

```bash
python -V
ruby -v
```

The interactive Python interpreter also provides a fast validation surface for small expressions, module imports, and simple checks that confirm the runtime works as expected.
### SSH security defaults
RHEL 9 hardens remote access by disabling password based SSH authentication for the root account by default. This reduces exposure to password guessing attacks that target a predictable account name. Key based authentication can still apply where policy allows, and administrators can still gain elevated access through sudo or su after logging in as a regular user.

The primary configuration file remains /etc/ssh/sshd_config. Administrators review the root login control setting and apply additional hardening where required.

```bash
sudo grep -n '^PermitRootLogin' /etc/ssh/sshd_config
```

Administrators validate SSH server configuration changes before restarting the service. OpenSSH provides a syntax check mode, and systemd manages the daemon restart.

```bash
sudo sshd -t
sudo systemctl restart sshd
sudo systemctl status sshd
```


A secure operational baseline typically includes these controls.

- Direct root SSH logins remain disabled where possible.
- Administrative access relies on key based authentication rather than passwords.
- Privileged work uses sudo so actions remain attributable to named user accounts.
- Additional controls restrict SSH access by network, allowlisted users, or multi factor policy where required.
### Network configuration storage
RHEL 9 completes the move away from legacy network scripts. The /etc/sysconfig/network-scripts directory may still exist for compatibility messaging, but active profiles no longer live as ifcfg files in that location. NetworkManager now stores connection profiles under /etc/NetworkManager/system-connections using its keyfile format.

This change affects troubleshooting and audit tasks that previously relied on opening and editing ifcfg files. RHEL 9 still uses the same operational tooling for network management.

- nmcli provides command line control.
- nmtui provides a text interface.
- Graphical NetworkManager utilities remain available where installed.

Administrators validate configuration state through NetworkManager rather than through direct file edits.

```bash
nmcli connection show
nmcli device status
```

NetworkManager stores each connection profile as a separate file, often with a .nmconnection suffix. The files commonly restrict access to root because they can include Wi-Fi or VPN secrets. Administrators generally prefer nmcli and nmtui for updates because these tools maintain correct permissions and reload profiles safely.

```bash
sudo ls -l /etc/NetworkManager/system-connections
sudo nmcli connection reload
```

The system protects connection profile files with strict permissions. Administrators typically require sudo to read files under /etc/NetworkManager/system-connections, and they treat these files as sensitive because they can include credentials.
## RHCSA EX200 expectations in a RHEL 9 context
### Exam character and continuity
The RHCSA exam (EX200) tests practical administration across one or more systems. Candidates can use man pages and standard on system help. The exam focuses on correct outcomes and reliable reasoning, not memorised command sequences.

RHEL 9 keeps the majority of RHCSA relevant workflows aligned with RHEL 8.

- Essential tools and text processing remain consistent.
- Shell scripting fundamentals remain consistent.
- Running system and service management remain consistent under systemd.
- Local storage and filesystem administration remain consistent.
- User and group administration remain consistent.
- Networking administration still centres on NetworkManager tooling.
- Core security work still centres on permissions and SELinux.

RHEL 9 shifts emphasis rather than replacing foundations.

- Network configuration storage shifts from ifcfg files to NetworkManager keyfiles.
- VDO shifts from separate management tooling to LVM managed workflows.
- Some storage topics reduce in exam focus.
- Container operations expand and integrate with systemd service management.
### Command line efficiency practices
Command line fluency matters in a practical exam and in production work. Reliable shortcuts reduce repetition and prevent common errors.

- Ctrl+A moves the cursor to the beginning of the line.
- Ctrl+E moves the cursor to the end of the line.
- Ctrl+L clears the terminal display without resetting the session.
- Ctrl+R searches command history interactively and recalls earlier commands.
- Esc+. cycles through previous command arguments and reuses long paths.
- sudo !! repeats the previous command with sudo.

These habits speed up file edits, minimise retyping, and allow faster recovery when a command runs without required privileges.
### Sudo resilience
Recent sudo releases behave more defensively when visudo detects syntax problems. visudo still validates changes and warns about errors, but it is less likely to leave a system unusable due to a minor mistake in a new entry. Administrators still treat sudoers as a high impact file and apply careful review before saving changes.
### Objective changes to note
RHEL 9 does not demand a full restart of study for candidates prepared on RHEL 8 objectives. The exam remains a Linux administration exam first, with platform specific differences concentrated in a few areas.

- Stratis does not appear as a key focus.
- VDO appears through LVM and can appear as an LVM task rather than a separate tool chain task.
- POSIX ACLs do not appear as a key focus compared with core permissions and SELinux.
- Container tasks connect Podman and systemd, including running containers as services.
- Container tasks can include scenarios that require service management inside containers for testing purposes.
## LVM managed VDO volumes in RHEL 9
### What VDO provides
VDO provides two primary space saving features.

- Data deduplication stores repeated blocks once and references them multiple times.
- Data compression stores data in a compressed form to reduce physical consumption.

Workloads with repeated data patterns benefit most. Virtual machine hosts often store many similar guest images. Template based deployments also repeat operating system files and package data. VDO can reduce physical storage use in these cases, provided the workload tolerates the additional CPU cost of compression and deduplication.
### Integration with LVM
RHEL 9 integrates VDO with LVM so administrators manage VDO volumes through standard LVM tooling.

- A VDO logical volume sits on top of a VDO pool logical volume.
- The kvdo kernel module supports VDO operations in the data path.
- Deduplication and compression remain enabled by default unless changed.

This design mirrors thin provisioning concepts. Administrators can present more space than the physical backing store provides, then rely on deduplication and compression to keep real usage within the backing limit.

The VDO pool provides the physical allocation. The VDO logical volume provides the presented capacity. LVM creates both objects and reports them as logical volumes with different roles.
### Prerequisites and readiness checks
A VDO capable host typically requires these packages.

```bash
sudo dnf install -y lvm2 vdo kmod-kvdo
```

If installing kmod-kvdo pulls a newer kernel, the host needs a reboot before VDO volumes operate correctly. Administrators confirm readiness before building volumes by checking installed packages and by verifying the running kernel.
### Core LVM inspection commands
LVM state verification often uses these commands.

```bash
lsblk
pvs
vgs
lvs
```

lsblk shows attached block devices, including loop devices used in lab environments. pvs, vgs, and lvs show physical volumes, volume groups, and logical volumes. Administrators confirm free space in the target volume group before allocating a VDO pool.
### Creating a VDO volume with lab storage
Lab environments can simulate an additional disk using a loop device backed by a file. Administrators commonly create a backing file, attach it to a loop device, build a volume group, then create a VDO logical volume.

A typical flow uses these steps.

```bash
sudo fallocate -l 10G /root/disk1
sudo losetup -f --show /root/disk1
sudo vgcreate myvg /dev/loop0
```

The VDO logical volume creation uses two key ideas.

- The command allocates physical space from the volume group to a VDO pool.
- The command presents a larger virtual size to the filesystem layer.

The example allocates all free extents to the pool. Administrators can also allocate a fixed size with -L on systems that need leftover free space for other volumes.

```bash
sudo lvcreate --type vdo --name myvdolv --extents 100%FREE --virtualsize 50G myvg
```

LVM creates an additional pool logical volume to back the VDO logical volume. The pool appears in lvs output alongside the presented VDO volume, often with a vpool naming pattern.

```bash
sudo lvs
```

Administrators then create a filesystem and mount it. XFS and ext4 both apply, and the filesystem reports the virtual size.

```bash
sudo mkfs.xfs -K /dev/myvg/myvdolv
sudo mkdir -p /mnt/vdo
sudo mount /dev/myvg/myvdolv /mnt/vdo
df -h /mnt/vdo
```
### Monitoring physical usage
df reports filesystem capacity and used space based on the presented virtual size. df does not report physical backing store usage for VDO. Administrators avoid relying on df alone for capacity planning and instead monitor the VDO pool.

vdostats reports physical usage and efficiency at the pool level.

```bash
sudo vdostats --human-readable myvg/vpool0
```

VDO reserves metadata and working space. A baseline allocation remains in use even on an empty filesystem, and that baseline becomes the minimum practical pool size for meaningful results. Overcommit works best when the workload contains real duplication and compressible data. Administrators set alerting on pool usage so that physical space does not silently exhaust while df still reports ample virtual free space.
### Demonstrating deduplication impact
Deduplication effectiveness appears when the filesystem stores repeated content. Copying identical files increases the filesystem used value, but physical consumption increases only slightly because VDO stores repeated blocks once.

A duplicate file test can validate deduplication without relying on a single line loop syntax. The following example copies the same file multiple times with distinct names.

```bash
for i in 1 2 3 4 5 6 7 8 9
do
  sudo cp /usr/lib/locale/locale-archive /mnt/vdo/rescue-copy-$i.img
done
```

df then reports a substantial increase in used space, while vdostats reports a modest increase in physical consumption. This difference provides a direct indicator that VDO deduplication operates correctly.
### Viewing and changing VDO feature flags
LVM reporting can include VDO feature flags for compression and deduplication.

```bash
sudo lvs -o +vdo_compression,vdo_deduplication
```

Feature changes apply at the pool level and use lvchange. Administrators enable or disable features to match workload characteristics and performance targets.

```bash
sudo lvchange --compression y myvg/vpool0
sudo lvchange --deduplication n myvg/vpool0
```

Compression and deduplication trade CPU cycles for storage savings. Administrators validate performance on representative workloads before disabling features or relying on aggressive overcommit.
## Podman and systemd in RHEL 9
### Installing and validating Podman
Podman is the container engine used for RHEL 9 container management. Administrators install Podman through the package manager and validate the installation through basic inventory commands.

```bash
sudo dnf install -y podman
podman version
podman info
podman image ls
podman container ls
podman container ls --all
```

The container and image inventory distinction matters. Images define templates, while containers represent runtime instances. A stopped container remains a container and can be restarted, while --all exposes non running containers for cleanup and audit.
### Running a container as a systemd service
RHEL 9 expects administrators to manage long running containers with the same operational discipline as other services. A common pattern combines a persistent host directory for application data with a systemd unit generated from the container definition.

Administrators often use a database container as an example because databases require persistent storage.

A typical flow looks like this.

```bash
sudo mkdir -p /var/lib/containers/mariadb
sudo chmod 700 /var/lib/containers/mariadb

sudo podman run -d --name mariadb \
  -v /var/lib/containers/mariadb:/var/lib/mysql \
  -e MARIADB_ROOT_PASSWORD='change-me' \
  docker.io/library/mariadb:latest
```

The environment variable passes sensitive material in clear text, so production practice often uses secrets rather than plain environment variables. The example focuses on mechanics rather than hardening.

Administrators validate container behaviour by running an interactive shell inside the container and confirming that the service accepts logins.

```bash
sudo podman exec -it mariadb bash
```

podman generate systemd converts the container into a unit file definition and prints the unit content to standard output. Redirection captures the unit content into a service file under /etc/systemd/system so the service manager can load it.

```bash
sudo podman generate systemd --name mariadb > /etc/systemd/system/mariadb.service
sudo systemctl daemon-reload
sudo systemctl enable --now mariadb.service
```

systemctl status then reports container service state through systemd, which simplifies routine checks.

```bash
systemctl status mariadb.service
```

Administrators often confirm that systemd controls the container by checking both the unit state and the container list. systemd reports service health through systemctl, while Podman reports container state through podman ps.

```bash
systemctl is-enabled mariadb.service
systemctl is-active mariadb.service
podman ps
```


This approach allows clean start and stop behaviour through systemctl, automatic startup at boot, and consistent operational audit points through unit files and system logs.
### Running systemd managed services inside a container
Containers typically run a single main process, but systemd inside a container supports test harnesses and DevOps workflows. Automation systems sometimes expect to enable and manage services with systemctl, and a systemd inside container pattern supports that expectation.

RHEL 9 requires the container to manage its own control groups when systemd runs as PID 1. Administrators enable the SELinux boolean that allows this behaviour.

```bash
sudo setsebool -P container_manage_cgroup true
```

A Dockerfile can build an image that runs systemd and starts enabled services. Administrators commonly use Fedora as a base image for demonstration and testing.

```bash
FROM registry.fedoraproject.org/fedora:38
RUN dnf -y install systemd httpd at && dnf clean all
RUN systemctl enable httpd atd
EXPOSE 80
CMD ["/usr/sbin/init"]
```

Administrators build the image and run a container with a host port mapping. The container then runs systemd as PID 1 and starts the enabled services.

```bash
sudo podman build -t web .
sudo podman run -d --name web1 -p 8080:80 web
```

Verification focuses on confirming that multiple processes run inside the container and that services start under systemd control.

```bash
sudo podman top web1
sudo podman exec -it web1 systemctl status httpd
sudo podman exec -it web1 systemctl status atd
```

This pattern supports repeatable testing of service enablement, service dependencies, and service restart behaviour without changing the host service set.