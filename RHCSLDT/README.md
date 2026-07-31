# *Red Hat Certified Specialist in Linux Diagnostics and Troubleshooting* Course Notes
*v1.0 stable release*

These notes cover material from Pluralsight's 17 hour, self-paced [Red Hat Certified Specialist in Linux Diagnostics and Troubleshooting](https://www.pluralsight.com/paths/red-hat-certified-specialist-in-linux-diagnostics-and-troubleshooting-ex342) course. The notes cover how to discover, analyze, and fix common issues that can be expected as a Red Hat Specialist, systems administrator, or DevOps engineer working with Linux. In addition, these notes cover how to handle issues related to system boot, package management, system files, network connectivity, application degradation, and authentication issues, as well as how to gather forensic information and handle any server issues.
## A disciplined troubleshooting method
Troubleshooting works best as a controlled evidence cycle:
1. Establish the symptom, affected users, start time, scope, and required service level.
2. Record the system state before changing it. Preserve logs, command output, configuration copies, and relevant timestamps.
3. Reproduce the fault only when reproduction is safe. A storage, security, or kernel failure can become worse under repeated testing.
4. Separate observations from assumptions. Compare the failed system with its intended configuration, a healthy peer, package metadata, and earlier records.
5. Form one testable hypothesis that explains the available evidence.
6. Make one reversible change, unless incident containment requires a broader action.
7. Repeat the original test, inspect related logs, and check for unintended effects.
8. Verify persistent configuration and reboot when the task requires it.
9. Record the cause, correction, validation, and any remaining risk.

Administrators should first protect data and access. Before repairing storage, package databases, boot files, authentication, or remote networking, they should obtain a backup or snapshot when possible and retain a working console or recovery path. They should avoid destructive demonstrations such as overwriting superblocks, shrinking live logical volumes, forcing kernel crashes, deleting package databases, or removing boot files on systems that contain valuable data.

Time correlation often exposes a fault. `date -Is`, `timedatectl`, and `chronyc tracking` establish the host's clock state. A wrong clock can distort log analysis and break Kerberos, TLS, and distributed applications.
## Collecting system evidence
### Platform and workload baseline
The first pass should identify the operating system, kernel, boot, hardware, resource pressure, and failed services:

```shell
cat /etc/redhat-release
uname -a
hostnamectl
uptime
systemctl --failed
systemctl list-jobs
journalctl -p warning..alert -b
```

`uptime` reports the current time, session age, logged-in users, and load averages for the last 1, 5, and 15 minutes. Load average counts runnable tasks and tasks waiting in uninterruptible sleep. It is not a direct CPU percentage, so an administrator should compare it with CPU count, I/O wait, process state, and workload behaviour.

The following tools locate common resource constraints:

| Area | Useful commands | Diagnostic focus |
| --- | --- | --- |
| CPU and tasks | `top`, `ps -eo pid,ppid,user,stat,pcpu,pmem,etimes,cmd --sort=-pcpu`, `pidstat` | Saturation, blocked tasks, runaway processes, parent-child relationships, and recent starts |
| Memory | `free -h`, `vmstat 1`, `ps --sort=-pmem`, `slabtop` | Available memory, swap activity, reclaim pressure, and allocation growth |
| File systems | `df -hT`, `df -ih`, `findmnt`, `du -x`, `lsblk -f` | Capacity, inode exhaustion, mount state, file-system type, and unexpected devices |
| I/O | `iostat -xz 1`, `pidstat -d 1`, `lsblk -o NAME,TYPE,SIZE,FSTYPE,MOUNTPOINTS` | Latency, queueing, throughput, and the process generating I/O |
| Sockets | `ss -lntup`, `ss -s`, `lsof -i` | Listening ports, owning processes, connection state, and socket pressure |
| Open files | `lsof`, `ls /proc/PID/fd`, `cat /proc/PID/limits` | Per-process descriptors, deleted open files, and soft or hard limits |

`free -h` uses human-readable units. `free -g`, not `free -j`, requests gibibyte-scale output. `df -hT` reports space by mounted file system, while `df -ih` detects inode exhaustion. A full file system and an inode-starved file system can produce similar application errors.

Counting all `lsof` output lines does not accurately measure one process's descriptor use because the output includes a header and several object types. `/proc/PID/fd` gives a direct per-process view. An administrator should compare that count with `/proc/PID/limits`, service-level `LimitNOFILE`, and the system-wide file-handle state. `systemctl show SERVICE -p LimitNOFILE` reveals the effective systemd limit.

`ps aux` provides a BSD-style overview. More explicit `ps -eo` fields produce repeatable output and avoid ambiguity. Process state deserves attention. A process in `D` state waits in uninterruptible sleep, often on I/O, while a zombie has exited but still awaits collection by its parent.

Kernel and hardware evidence includes:

```shell
journalctl -k -b
dmesg -T
lscpu
lsblk
lspci -nnk
lsusb
dmidecode
udevadm info /sys/class/net/INTERFACE
```

Virtual machines expose virtualised hardware, so guest data may not reveal the physical fault. Administrators should correlate guest evidence with hypervisor, firmware, baseboard management controller, storage array, and cloud-provider telemetry. Repeated machine-check, EDAC, PCIe, NVMe, disk, or network-driver errors require hardware or platform investigation. An offline memory test should run from trusted, supported media during an approved outage.
### Hardware and platform faults
Hardware diagnosis should distinguish a component fault from a driver, firmware, power, thermal, cabling, or virtualisation fault. A single error can result from a transient event, while repeated errors across devices can indicate a shared controller, bus, power source, or host.

CPU and memory evidence can include machine-check records, Error Detection and Correction reports, corrected-error counts, uncorrected errors, thermal throttling, and out-of-memory events. RHEL 10 administrators should start with the kernel journal and platform telemetry rather than assume that a legacy `mcelog` workflow applies:

```shell
journalctl -k -b | grep -Ei 'machine check|hardware error|edac|thermal|memory'
lscpu
dmidecode --type memory
```

Corrected memory errors show that error correction operated, but a rising count can justify component replacement under vendor thresholds. An uncorrected error, kernel panic, or repeated fault at the same physical address requires prompt escalation. Guest memory addresses may not map directly to physical modules.

For PCIe and device faults, `lspci -nnk` identifies the device, vendor and product identifiers, active driver, and candidate modules. `lspci -vv` adds link and capability details. Repeated link resets, Advanced Error Reporting events, or a device disappearing from the bus can indicate hardware, firmware, slot, or power trouble. The administrator should compare firmware and driver support against the certified platform.

Disk and NVMe analysis should correlate application errors with block-layer messages, path resets, device health, array state, and latency. A media error at one logical block differs from repeated controller timeouts across every disk. RAID can preserve service after a member fault, but degraded operation reduces redundancy and can increase rebuild load. Replacement should follow storage-vendor procedure and confirmed device identity.

Network errors also require counter context. Receive drops can result from host overload or queue limits, while cyclic redundancy check and carrier errors more strongly suggest a physical path. Virtual interfaces can inherit host congestion without reporting physical counters inside the guest.

Environmental and platform records often supply the decisive evidence. Baseboard management controllers can report fan, temperature, voltage, power-supply, memory, and processor events even when RHEL cannot boot. Cloud and hypervisor consoles can report host maintenance, volume degradation, or virtual-network faults. Administrators should preserve these records with RHEL timestamps and note any clock offset.

A component swap is also a diagnostic change. The administrator should record serial numbers and slots, change one component at a time, observe anti-static and outage controls, and run the original workload test afterwards. Repeated replacement without evidence can conceal a shared fault.
### Documentation and package ownership
The local system remains useful when internet access is unavailable:

```shell
COMMAND --help
man COMMAND
apropos KEYWORD
man -k KEYWORD
rpm -qf /path/to/file
rpm -ql PACKAGE
rpm -qd PACKAGE
dnf provides '/path/or/glob'
dnf repoquery -l PACKAGE
dnf repoquery --requires PACKAGE
```

Manual section 1 covers user commands, section 5 covers file formats, and section 8 covers administration commands. `man 5 rsyslog.conf`, for example, requests the configuration-file page rather than the daemon page. Package documentation usually resides under `/usr/share/doc/PACKAGE`.

`rpm -qf` identifies the installed package that owns a file. `dnf provides` searches enabled repository metadata, including files from packages that are not installed. `dnf repoquery` replaces older assumptions that every query belongs to a separate `yum-utils` workflow.

RHEL 10 also offers an optional command-line assistant through supported repositories. It can explain commands, logs, and error output, but administrators must review every generated suggestion and protect personal, confidential, business-sensitive, and system-sensitive data. An offline certification environment may not provide the service.
## Monitoring and automation
### Correlating resource pressure
Administrators should compare live utilisation with a historical baseline and a user-visible event. One high CPU sample can represent useful work, while sustained saturation with a growing run queue can explain latency. High load with low CPU use often indicates tasks blocked on I/O.

`vmstat 1` separates runnable tasks, blocked tasks, swap, I/O, interrupts, context switches, CPU use, and I/O wait. The first line reflects averages since boot, so administrators should not mistake it for the first one-second interval. `iostat -xz 1` shows device throughput, queueing, utilisation, and latency. No single utilisation percentage identifies saturation across every storage technology, so latency and workload expectations remain essential.

`pidstat 1` attributes CPU use by process. `pidstat -d 1` adds task I/O, and `pidstat -r 1` adds memory faults and usage. A process can generate I/O through mapped files or delegated workers, so administrators should correlate process and device evidence by time.

Swap use alone does not prove current memory pressure. Sustained swap-in and swap-out activity, low available memory, direct reclaim, allocation failures, or an out-of-memory kill provide stronger evidence. Page cache is reclaimable and normally improves performance.

Historical evidence from PCP or sysstat can show whether a fault began with a deployment, traffic change, device slowdown, memory-growth trend, or scheduled job. Administrators should align archive time zones, collection intervals, and gaps before comparing metrics with logs.
### Performance Co-Pilot
Performance Co-Pilot, or PCP, records and analyses live and historical system metrics. A basic installation commonly uses the `pcp` and `pcp-system-tools` packages:

```shell
dnf install pcp pcp-system-tools
systemctl enable --now pmcd
systemctl enable --now pmlogger
systemctl status pmcd pmlogger
```

`pmcd` receives metrics from Performance Metrics Domain Agents. `pmlogger` writes archives, normally beneath `/var/log/pcp/pmlogger`. Administrators should inspect the active service configuration instead of assuming an archive filename.

Common tools serve different purposes:
- `pmstat` provides a concise live overview of CPU, memory, disk, network, load, and process activity.
- `pminfo` lists metrics. `pminfo -t METRIC` adds a short description.
- `pmval METRIC` displays values for one metric.
- `pmrep` reports selected metrics in a configurable table and can read live or archived data.
- `pcp atop` presents an interactive, resource-oriented view of processes and system activity.

PCP time options include an interval, start time, finish time, sample count, and archive. Administrators should confirm the requested interval against the archive's actual coverage and host time zone. High-frequency collection increases overhead and archive size.

The RHEL Web Console can display PCP metrics, logs, services, storage, networking, software updates, and other host state. Where policy permits it, administrators can enable the socket and required firewall service:

```shell
systemctl enable --now cockpit.socket
firewall-cmd --permanent --add-service=cockpit
firewall-cmd --reload
```

The Web Console normally listens on TCP port 9090. Its graphical views support diagnosis, but command-line evidence remains important for repeatability and low-bandwidth recovery.
### Ansible and RHEL system roles
The published EX342 objectives now include configuration with Ansible. An administrator should validate inventory, transport, privilege escalation, syntax, and idempotence before blaming a managed node:

```shell
ansible-inventory --graph
ansible all -m ping
ansible-playbook --syntax-check playbook.yml
ansible-playbook --check --diff playbook.yml
ansible-playbook playbook.yml
```

`ansible.builtin.ping` tests Python execution and authentication, not ICMP. Administrators should separate a failed run into DNS, SSH, host-key, credential, Python, privilege, variable, module, and target-service faults. `-vvv` can expose connection and task details, but its output can contain sensitive data.

Tasks should declare the intended state and notify handlers only when a change requires a restart or reload. A second successful run should report no unexpected changes. RHEL system roles provide supported automation for storage, networking, firewall, logging, journald, kernel settings, PCP, the Web Console, SSH, and other functions. Administrators should inspect each installed role's documentation under `/usr/share/ansible/roles` or the applicable collection before setting variables.
## Logs and central collection
### The systemd journal
`systemd-journald` collects kernel, boot, service, standard-output, and syslog messages. RHEL normally uses `rsyslog` to route selected messages into persistent text files under `/var/log`. The journal uses volatile storage under `/run/log/journal` unless configuration and storage enable persistence.

Useful queries include:

```shell
journalctl -b
journalctl -b -1
journalctl -k -b
journalctl -u SERVICE --since today
journalctl -xeu SERVICE
journalctl -p warning..alert
journalctl --since "2026-07-31 09:00:00" --until "2026-07-31 10:00:00"
journalctl -f
```

`-b` selects a boot, `-k` selects kernel messages, `-u` selects a unit, `-p` selects a priority range, and `-f` follows new records. `journalctl --list-boots` shows retained boot identifiers. `journalctl --disk-usage` and `journalctl --verify` assist with storage and integrity checks.

Persistent journal configuration belongs in a drop-in such as `/etc/systemd/journald.conf.d/persistent.conf`:

```ini
[Journal]
Storage=persistent
```

After creating the drop-in, the administrator can restart `systemd-journald` and run `journalctl --flush`. The service manages journal ownership and layout. Hand-created paths with guessed permissions can create access or labelling faults.
### Rsyslog
Rsyslog selectors use a facility and priority. Priorities run from `emerg` through `debug`. A selector for a priority normally includes that priority and all more severe priorities. Administrators should consult `rsyslog.conf(5)` when they require an exact comparison or exclusion.

Remote collection requires a receiver, forwarding clients, network reachability, firewall rules, durable queues, and an agreed message format. Configuration belongs under `/etc/rsyslog.d/` where possible. Before a restart, `rsyslogd -N1` validates syntax. `logger` can then send a controlled test message.

Plain TCP improves delivery over UDP but does not provide confidentiality or peer authentication. Production deployments should use TLS or RELP with appropriate certificates, queue settings, storage controls, and rotation. The RHEL logging system role can configure remote collection and TLS consistently across multiple hosts.

Central logging does not replace local evidence. A network outage can interrupt forwarding, and an attacker may target the collector. Administrators should retain suitable local buffers, protect logs from unauthorised alteration, synchronise time, and verify both local and remote receipt.
## Boot, kernel, and service recovery
### Firmware, GRUB, and the boot sequence
A RHEL 10 host progresses through firmware, the boot loader, the kernel and initial RAM file system, and systemd. Locating the failed stage narrows the investigation:
- A host that never reaches GRUB requires firmware, boot-order, storage, or platform checks.
- A GRUB prompt or menu failure points to the boot loader, its configuration, or the boot file system.
- A kernel panic before the real root mounts points to the kernel command line, drivers, storage discovery, encryption, LVM, or the initial RAM file system.
- A host that reaches systemd but not its normal target requires unit, mount, dependency, authentication, or service analysis.

BIOS firmware normally loads GRUB boot code from a disk boot area. UEFI firmware loads an EFI application from the EFI System Partition. UEFI removes several legacy BIOS constraints, but it still depends on valid firmware entries, an accessible EFI System Partition, and a bootable loader.

RHEL 10 uses Boot Loader Specification entries and `grubby` for routine kernel command-line management. `/boot/loader/entries/` contains the entries. Administrators should inspect the effective configuration before editing it:

```shell
grubby --default-kernel
grubby --info=ALL
cat /proc/cmdline
ls -l /boot/loader/entries/
```

The primary generated GRUB configuration is `/boot/grub2/grub.cfg` on both BIOS and UEFI RHEL 10 installations. The file on the EFI System Partition acts as a stub. Administrators must not replace that stub by directing `grub2-mkconfig` to its path. When administrators justify regeneration, the command targets `/boot/grub2/grub.cfg`:

```shell
grub2-mkconfig -o /boot/grub2/grub.cfg
```

`grubby --update-kernel=ALL --args="ARGUMENT"` adds a persistent argument. `--remove-args` removes it. A one-boot test can edit the GRUB menu entry interactively, which reduces recovery risk. The administrator should confirm the active value in `/proc/cmdline` after reboot.

Reinstalling GRUB differs by firmware. A BIOS system can require `grub2-install` against the correct whole-disk device. An x86_64 UEFI system normally requires reinstallation of the `grub2-efi-x64` and `shim-x64` packages from a rescue environment. Secure Boot depends on signed components and firmware trust, so an unsigned kernel module or altered loader can fail even when its file exists.
### Kernel and initial RAM file system
The kernel mediates access to processors, memory, devices, file systems, networking, and process isolation. It is not the processor itself. Administrators can compare a running kernel with installed packages and boot files:

```shell
uname -r
rpm -q kernel-core
ls -lh /boot/vmlinuz-* /boot/initramfs-*
journalctl -k -b
```

`lsmod` lists loaded modules, `modinfo MODULE` describes a module, and `modprobe MODULE` loads it with dependencies. `modprobe -r MODULE` removes it only when no active user or dependency prevents removal. Built-in drivers appear in `/lib/modules/$(uname -r)/modules.builtin`, and module files may use compression.

Module parameters supplied through `modprobe` apply when the module loads. Changing a parameter normally requires an unload and reload, a supported run-time interface, or a reboot. Persistent options belong in a `.conf` file under `/etc/modprobe.d/`. A blacklist can prevent automatic loading, but the administrator should test whether the storage, network, or console path depends on that driver before making the change persistent.

The initial RAM file system contains the early userspace required to discover and mount the real root. A missing storage driver, stale LVM metadata, wrong UUID, damaged image, or incorrect kernel command line can stop the boot before systemd starts. `lsinitrd` inspects an image. `dracut -f` rebuilds the image for the running kernel, while `dracut -f /boot/initramfs-VERSION.img VERSION` targets a specified installed kernel. The administrator should confirm sufficient free space in `/boot` and retain another bootable kernel.
### Systemd units and dependencies
Systemd starts and supervises units. Service, socket, target, mount, timer, path, and device units describe resources and relationships. The following commands distinguish installation state, active state, failure, dependency, and ordering:

```shell
systemctl status UNIT
systemctl is-enabled UNIT
systemctl is-active UNIT
systemctl is-failed UNIT
systemctl show UNIT
systemctl list-dependencies UNIT
systemd-analyze critical-chain
systemd-analyze blame
```

`Requires=` adds a strong requirement, and `Wants=` adds a weaker one. These directives do not establish start order by themselves. `After=` and `Before=` define ordering. `BindsTo=` can stop a unit when a bound resource disappears. `PartOf=` propagates selected stop and restart operations. A correct diagnosis therefore checks both requirement and ordering relationships.

Vendor unit files reside under `/usr/lib/systemd/system`. Administrators should not edit them because package updates can replace those changes. Full local replacements belong under `/etc/systemd/system`, but a focused drop-in is usually safer:

```shell
systemctl edit SERVICE
systemctl cat SERVICE
systemctl daemon-reload
systemctl restart SERVICE
```

A drop-in can add environment settings, resource limits, dependencies, or execution controls. Some list-valued directives require an empty assignment before replacement. `systemd-delta` displays local overrides. `systemd-analyze verify FILE` can detect selected syntax and dependency errors.

When a service fails, the administrator should inspect the unit, journal, executable, user, environment, working directory, permissions, labels, ports, resources, and dependencies. Running the daemon manually can reveal an error, but the test must use the same identity and relevant environment, and it must not create a second instance or bypass required confinement.

Restart limits can hide the original cause. `systemctl reset-failed SERVICE` clears the failed state and counters after the fault has been understood. Repeatedly resetting and restarting a damaged service can overwrite useful logs or intensify resource pressure.

The debug shell can provide a root shell on virtual terminal 9 early in boot:

```shell
systemctl enable debug-shell.service
```

It bypasses normal authentication. Administrators should enable it only for a controlled recovery, protect console access, and disable it immediately afterwards.
### Rescue and emergency access
Systemd rescue mode starts a minimal environment with local file systems and a rescue shell. Emergency mode starts an even smaller environment and may leave the root file system read-only. Kernel arguments such as `systemd.unit=rescue.target` or `systemd.unit=emergency.target` can select them for one boot.

Installation media can boot a rescue environment when the installed system cannot. The environment normally mounts the installed root at `/mnt/sysroot`. The administrator should verify mounts before entering it:

```shell
lsblk -f
findmnt
mount -o remount,rw /mnt/sysroot
chroot /mnt/sysroot
```

Encrypted volumes, software RAID, and LVM may require discovery first. `vgchange -ay` activates available volume groups. The command is `vgchange`, not `vdchange`.

`rd.break` interrupts the initial RAM file system before switching to the real root. A common root-password recovery sequence verifies the target, remounts `/sysroot` read-write, enters it, changes the password, and requests SELinux relabelling:

```shell
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
exit
exit
```

The file is `/.autorelabel`. A targeted `restorecon` can be preferable when the administrator knows exactly which label changed. A full relabel can extend the next boot substantially. Physical or virtual-console access to an editable kernel command line can permit privileged recovery, so firmware, boot-loader, console, and data-centre controls remain security boundaries.

Recovery ends with validation. The administrator should remove temporary arguments, disable debug access, confirm the default target, boot the intended kernel, verify services and mounts, and retain the recovery record.
## Storage diagnosis and recovery
### Mapping the storage stack
A storage fault can occur at the application, file-system, logical-volume, encryption, multipath, transport, device, or platform layer. Administrators should map each layer before repairing any one of them:

```shell
findmnt
lsblk -o NAME,TYPE,SIZE,FSTYPE,FSVER,LABEL,UUID,MOUNTPOINTS
blkid
df -hT
df -ih
pvs
vgs
lvs -a -o +devices
```

`df` reports a mounted file system's view. `du` totals reachable directory entries. A large difference can indicate deleted files that a process still holds open, hidden data beneath a mount point, reserved space, snapshots, or different traversal boundaries. `lsof +L1` finds open files with a zero link count. Restarting the owning service, after impact assessment, releases that space.

Kernel messages can distinguish file-system corruption from transport or media errors:

```shell
journalctl -k -b | grep -Ei 'I/O|xfs|ext4|nvme|scsi|reset|timeout'
smartctl -a /dev/DEVICE
```

SMART data can support a diagnosis, but a clean SMART summary does not prove that a device, path, controller, cable, or backing store is healthy. Virtual disks require evidence from the host or storage service.
### XFS and ext4 repair
Repair starts with protection. The administrator should stop writers, capture logs, confirm the exact device and file-system type, and obtain a current backup, snapshot, or block-level image when feasible. Repair tools should not run against a mounted file system.

For XFS, `xfs_repair -n DEVICE` performs a no-modify scan. It can still read the complete device and take substantial time. Normal repair uses `xfs_repair DEVICE` while the file system remains unmounted. A dirty log normally requires mounting and unmounting the file system on the same system so that XFS can replay the log. If replay cannot occur, `xfs_repair -L` zeroes the log. That last-resort operation can discard metadata changes and cause data loss.

For ext2, ext3, and ext4, `e2fsck -n DEVICE` answers repair prompts with no and provides a read-only assessment. `e2fsck -p DEVICE` applies fixes that the tool considers safe for automatic repair. Interactive or forced checks require a maintenance window and a verified unmounted target. An administrator should never infer the target from a changing device name when a stable UUID or mapped path is available.

After repair, the administrator should mount the file system, inspect the repair report and `lost+found`, verify ownership and SELinux labels, test application data, and monitor kernel logs. A clean metadata check does not prove that every file contains correct application data.
### LVM metadata
LVM separates physical volumes, volume groups, and logical volumes. Metadata archives under `/etc/lvm/archive` and backups under `/etc/lvm/backup` describe that layout. They do not contain the files stored inside logical volumes.

Useful checks include:

```shell
pvscan
vgscan
lvscan
pvs -o +pv_uuid,pe_start,pv_size,pv_free
vgs -o +vg_uuid,vg_size,vg_free
lvs -a -o +devices,segtype,lv_size
pvck /dev/DEVICE
vgcfgbackup VOLUME_GROUP
```

A missing logical volume can result from an inactive volume group, a missing physical volume, a filter or devices-file issue, duplicate identifiers, stale multipath state, or damaged metadata. The administrator should resolve discovery and path problems before restoring metadata.

`vgcfgrestore -l VOLUME_GROUP` lists available metadata versions. Restoration requires the correct volume group, matching physical volumes, and an inactive affected layout. The administrator should save the current metadata, inspect the selected archive, and understand every changed extent before running `vgcfgrestore`. Restoring old metadata can remove later logical volumes or point extents at the wrong places.

Metadata restoration cannot recreate overwritten user data. Enlarging an underlying block device after accidental truncation can restore an address range, but it does not restore discarded bytes. XFS cannot shrink, so a proposed recovery that depends on shrinking an XFS file system is invalid.

LVM snapshots allocate copy-on-write storage for changed blocks. A snapshot that fills becomes unusable. Thin pools also require data and metadata monitoring. `lvs -a -o +data_percent,metadata_percent` reveals utilisation. Administrators should not treat a snapshot as an independent backup, especially when it shares the original device and failure domain.
### LUKS2 encryption
RHEL 10 uses LUKS2 for new encrypted block devices by default. A LUKS header contains encryption metadata and keyslots. Loss or corruption of the header can make intact encrypted data inaccessible.

Administrators should inspect the device and record its UUID without exposing keys:

```shell
cryptsetup luksDump /dev/DEVICE
cryptsetup luksUUID /dev/DEVICE
cryptsetup open /dev/DEVICE MAPPING_NAME
cryptsetup close MAPPING_NAME
```

`/etc/crypttab` should normally identify the source by UUID. A key file should contain strong random data, use root-only permissions, and reside on protected storage. A plaintext copy of a human passphrase is not an appropriate key file.

Key management needs verification. `cryptsetup luksAddKey` adds a new keyslot after authenticating with an existing key. `cryptsetup luksRemoveKey` removes the slot associated with a supplied key. Before removal, the administrator should prove that another independent key opens the volume and confirm that the proposed key does not unlock an unintended slot.

Header backup and restoration use explicit device and backup-file arguments:

```shell
cryptsetup luksHeaderBackup /dev/DEVICE --header-backup-file HEADER_FILE
cryptsetup luksHeaderRestore /dev/DEVICE --header-backup-file HEADER_FILE
```

The backup must remain confidential, integral, and available offline. Anyone who obtains it and a valid key from the backup's time may be able to decrypt the data. Restoring it overwrites current header metadata and rolls back later keyslot changes, so the administrator should verify the target device and exhaust safer options first.
### iSCSI
An iSCSI path includes a target, portal, network route, TCP port 3260, initiator, discovery record, session, and exported logical unit. Authentication and access control add further failure points.

On a RHEL target, `targetcli` can create a backstore, an iSCSI Qualified Name, a logical unit, a portal, and an access-control entry. Administrators must save the configuration, and the `iscsi-target` firewall service must permit the chosen network. The target should expose only the intended backing device and initiator.

On an initiator, diagnosis proceeds from network reachability to discovery and login:

```shell
ip route get TARGET_ADDRESS
nc -vz TARGET_ADDRESS 3260
iscsiadm -m discovery -t sendtargets -p TARGET_ADDRESS
iscsiadm -m node
iscsiadm -m session -P 3
lsblk
```

`nc` checks a TCP connection more directly than a legacy Telnet client. A successful TCP connection proves only that something accepted the port. Discovery, CHAP, access control, logical-unit mapping, multipath, and file-system checks still follow.

Node-specific `iscsiadm` updates are safer than broad edits to global authentication files. Credentials should not appear in command history, logs, or unrestricted configuration copies. The `iscsid` service can start on demand, so an inactive status without a requested session does not by itself indicate failure.

A remote block device can disappear during network or target failure. Administrators should use suitable timeouts, multipath design, `_netdev` mount semantics, and service ordering. They should not run a local file-system repair until the remote path is stable and the storage owner has ruled out concurrent access.
## Software and package integrity
RHEL 10 uses DNF for repository and package operations. A package failure can arise from repository configuration, entitlement, DNS, proxy settings, TLS, metadata, dependency resolution, a locked transaction, insufficient space, damaged RPM state, or altered installed files.

The first checks should preserve the original error and separate network access from dependency resolution:

```shell
dnf repolist --all
dnf repoinfo REPOSITORY
dnf makecache
dnf check
dnf repoquery --duplicates
dnf repoquery --unsatisfied
rpm -qa | sort
```

`dnf clean all` removes cached metadata and packages. It can help when a cache is stale or corrupt, but it also removes evidence and forces new downloads. Administrators should not use it as an unexplained first step.

GPG verification authenticates signed package content against an installed trusted key. Disabling signature checks converts a trust failure into a supply-chain risk. An administrator should verify repository URLs, system time, CA trust, signing-key provenance, and package origin instead.
### RPM database recovery
Current RPM uses an SQLite-based database on RHEL 10. Old procedures that delete `__db*` files or rebuild a Berkeley DB `Packages` file do not apply and can destroy useful state. Recovery should start by stopping package operations, finding processes that hold the database, checking storage, and copying `/var/lib/rpm` to protected storage.

The supported RPM database utilities include:

```shell
rpmdb --verifydb
rpmdb --rebuilddb
rpm -qa
dnf check
```

`rpmdb --rebuilddb` rebuilds database indices from available records. It cannot reconstruct package state that no longer exists. After a rebuild, the administrator should compare the installed package list with a known baseline, run DNF consistency checks, inspect the transaction history, and verify critical packages.

`dnf history` displays recorded transactions. `dnf history info ID` shows one transaction. History undo or rollback can be useful for selected packages, but it depends on available package versions and suitable script behaviour. Red Hat does not support using history rollback to downgrade core RHEL packages as a general system rollback mechanism. Backups, snapshots, and tested deployment procedures provide a safer recovery boundary.

Version locking can prevent an expected update. Administrators should inspect active locks and repository excludes before treating a missing candidate as corruption.
### Installed-file verification
`rpm -V PACKAGE` compares installed files with package metadata. Output positions represent size, mode, digest, device, link, owner, group, and time differences. A leading file-type marker identifies configuration, documentation, ghost, licence, or other special content. A changed configuration-file digest does appear in verification output.

Verification reports a difference, not its legitimacy. Administrators should compare configuration with policy, package defaults, configuration management, and a healthy host. `rpm -qf FILE` confirms ownership, and `rpm -qc PACKAGE` lists packaged configuration.

`rpm --restore PACKAGE` restores supported metadata such as permissions, ownership, capabilities, and timestamps. It does not replace changed file content. `dnf reinstall PACKAGE` restores packaged payloads, but RPM can preserve a modified `%config(noreplace)` file and install a `.rpmnew` copy. Administrators must inspect `.rpmnew` and `.rpmsave` files rather than assume that reinstalling reset the configuration.

SELinux context repair is separate:

```shell
restorecon -RFv /path
rpm -V PACKAGE
```

The administrator should verify the service, logs, sockets, and original user-facing test after package repair.
## Network diagnosis
### Layered isolation
A useful network diagnosis starts locally and expands only after each layer works:
1. Confirm the intended NetworkManager connection, device state, link, address, and prefix.
2. Confirm the local route decision and neighbour resolution.
3. Test the local gateway and a remote address.
4. Test DNS selection and name resolution.
5. Test the required transport port and application protocol.
6. Inspect firewalls, policy routing, tunnels, namespaces, containers, and remote controls.
7. Capture packets at the narrowest useful point when ordinary evidence cannot distinguish the fault.

The OSI model provides vocabulary, but real faults often cross layers. A DNS timeout can come from routing, a firewall, server health, MTU, TLS, or an application resolver. The administrator should base each next step on observed packets and state.

Core RHEL 10 commands include:

```shell
nmcli general status
nmcli device status
nmcli connection show --active
ip -br link
ip -br address
ip route
ip rule
ip neighbour
ss -lntup
```

`ip route get DESTINATION` shows the route, source address, interface, and next hop that the kernel would select. It is often more informative than reading the main routing table alone.

Interface counters and driver data can expose a physical or virtual link fault:

```shell
ip -s link show dev INTERFACE
ethtool INTERFACE
ethtool -i INTERFACE
ethtool -S INTERFACE
journalctl -k -b
```

Administrators should compare errors, drops, carrier changes, queue counters, negotiated speed, duplex, and driver messages over a measured interval. Offload features can make a host capture appear to contain oversized or incomplete packets even when packets on the wire are valid.
### NetworkManager configuration
NetworkManager is the supported network configuration service. RHEL 10 stores persistent keyfiles under `/etc/NetworkManager/system-connections/`. Legacy persistent-interface rules under `/etc/udev/rules.d/70-persistent-net.rules` are not the standard configuration mechanism.

`nmcli connection show NAME` displays a connection profile. `nmcli device show INTERFACE` displays current device state. A profile can exist without being active, and an active device can use a different profile than expected.

After a manual keyfile correction, the administrator should protect the file with root-only permissions, reload profiles, activate the intended connection, and verify the resulting state:

```shell
nmcli connection reload
nmcli connection up NAME
nmcli connection show --active
ip address show dev INTERFACE
ip route
```

Remote changes can sever the management path. An administrator should use console access, a scheduled rollback, `nmstatectl` safe application where appropriate, or an atomic automation workflow. The RHEL network system role can apply repeatable connection profiles across hosts.

Duplicate addresses can produce intermittent reachability. `ip neighbour`, arping from an authorised segment, switch tables, DHCP records, and packet capture can identify conflicting link-layer addresses. IPv6 diagnosis should account for router advertisements, neighbour discovery, duplicate-address detection, prefix length, and link-local scope.
### Routing, transport, and MTU
`ping` tests ICMP echo only when both endpoints and controls permit it. Failure does not prove that the target is down, and success does not prove that an application port works. High-rate flood tests can disrupt systems and should not form part of routine diagnosis.

`tracepath DESTINATION` estimates the path and path MTU without requiring the legacy assumptions of a simple traceroute. `ss` replaces most diagnostic uses of `netstat`. It can display listening sockets, established sessions, queues, timers, owning processes, and protocol statistics.

For a service listening only on loopback, remote packets can reach the host but never reach the application. The administrator should compare:

```shell
ss -lntup
systemctl status SERVICE
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

Connection refusal normally indicates an active host with no listener or an explicit rejection. A timeout can indicate packet loss, filtering, routing failure, an unresponsive service, or asymmetric return traffic. Packet capture and server-side logs distinguish these cases.

An MTU fault often permits small exchanges but stalls larger TLS, storage, or tunnel traffic. `tracepath`, interface MTU values, tunnel overhead, and a controlled capture can identify fragmentation or Packet Too Big failures. Blocking all ICMP and ICMPv6 can break path MTU discovery.
### DNS
`/etc/resolv.conf` is the resolver file. NetworkManager normally manages it from active connection and DNS policy. A manual edit may disappear when a connection changes.

Diagnosis should compare the configured servers, search domains, routing domains, and the response from a selected server:

```shell
cat /etc/resolv.conf
nmcli device show | grep -E 'IP4.DNS|IP6.DNS|DOMAIN'
getent hosts NAME
dig NAME
dig @SERVER NAME
dig +trace NAME
```

`getent` follows the system's Name Service Switch path and therefore reflects application-style resolution more closely than `dig`. `dig` interrogates DNS directly. Their results can differ because of `/etc/hosts`, NSS order, caching, split DNS, search domains, or an application-specific resolver.

Forward and reverse records serve different purposes. A missing reverse record does not prevent ordinary forward resolution, though applications and security controls can depend on it. DNSSEC validation, stale caches, truncated UDP answers, TCP fallback, wrong time, and blocked port 53 can also produce failures.
### Firewalld and packet capture
Firewalld associates interfaces and sources with zones. Runtime configuration takes effect immediately but disappears on restart. Permanent configuration survives reload but does not take effect until a reload or matching run-time change:

```shell
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --permanent --list-all
firewall-cmd --check-config
```

Opening a port does not start the service. Starting a service does not open the firewall. The administrator should confirm the listener, zone assignment, rule, route, and remote test independently. Rich rules and direct nftables state can require additional inspection.

`tcpdump` uses Berkeley Packet Filter syntax for capture and display filters:

```shell
tcpdump -ni any host 192.0.2.10 and port 443
tcpdump -ni INTERFACE -s 0 -w capture.pcap
tcpdump -nnr capture.pcap
```

`tshark` uses `-f` for a capture filter and `-Y` for a display filter. Those syntaxes are not interchangeable. PCAP is a capture-file format, not a programming interface.

A capture can contain credentials, tokens, personal data, internal addresses, and application content. Administrators should minimise scope and duration, restrict access, transfer captures securely, and delete them according to policy. Encrypted traffic still reveals endpoints, timing, sizes, handshakes, resets, and retransmissions.

Capture location influences interpretation. A host capture can miss packets dropped in hardware or show offloaded segments. A switch, hypervisor, container namespace, firewall, load balancer, or remote peer may see different traffic. Simultaneous captures with synchronised clocks can reveal the loss point.
## Application and security diagnosis
### From service symptom to process cause
Application diagnosis should reproduce the actual client operation and trace it through name resolution, transport, service supervision, process state, configuration, dependencies, access controls, and data. A healthy process identifier does not prove that the application accepts requests correctly.

The first service checks include:

```shell
systemctl status SERVICE
systemctl cat SERVICE
systemctl show SERVICE
journalctl -u SERVICE -b
ss -lntup
ps -ef --forest
```

The administrator should identify the process's user and group, arguments, environment, limits, namespaces, current directory, open files, sockets, and cgroup. `/proc/PID/` exposes much of this state. `systemctl show` reveals values after systemd has combined vendor units, drop-ins, defaults, and run-time properties.

Many daemons provide a syntax or configuration test. That test should run before a restart. It can detect parsing errors, but it may not test credentials, remote dependencies, file access under the service identity, or the complete production request.

An application that starts manually but fails under systemd may depend on an interactive shell's environment, current directory, resource limits, terminal, supplementary groups, umask, or credentials. The repair belongs in supported service configuration, not in a global shell start-up file.

Resource failures can look like application defects. The administrator should check:
- File descriptor use and `LimitNOFILE`.
- Process and thread limits.
- Cgroup CPU, memory, and I/O controls.
- File-system space, inodes, quotas, and read-only state.
- Memory pressure and out-of-memory kills.
- Port conflicts and ephemeral-port exhaustion.
- Dependency timeouts, connection pools, and queue depth.

`journalctl -k` and the service journal can reveal an out-of-memory kill. `systemd-cgtop`, `systemctl show`, and files under `/sys/fs/cgroup/` expose cgroup state. Increasing a limit can postpone a leak, so the administrator should identify the growth pattern and expected workload before changing it.
### Executables, libraries, and system calls
`file PROGRAM` identifies the object type and architecture. `rpm -qf PROGRAM` identifies package ownership. `rpm -V PACKAGE` detects packaged-file changes.

`ldd` displays dynamic-library resolution for a trusted executable, but it can be unsafe on an untrusted binary because some implementations or object formats may execute code. `readelf -d PROGRAM` or `objdump -p PROGRAM` provides safer static inspection of dynamic dependencies. `ldconfig -p` displays the current library cache.

A missing shared object can result from a missing package, a wrong architecture, a stale cache, an invalid run path, or an unsupported third-party build. RHEL 10 no longer supplies 32-bit packages, so applications that require them need a supported 64-bit build, container, or other approved execution environment.

`strace` records system calls and signals. It does not produce a language-level stack trace. A focused trace reduces noise:

```shell
strace -f -tt -o trace.log -e trace=file,network,process COMMAND
strace -f -p PID
```

Attaching changes timing and can expose sensitive arguments or data. The administrator should capture only what the diagnosis requires. Common results include `ENOENT` for a missing path, `EACCES` for denied access, `ECONNREFUSED` for a rejected connection, and `ETIMEDOUT` for an expired operation.

GDB can inspect a stopped process, executable, or core dump. Systemd-coredump integrates crash records with the journal:

```shell
coredumpctl list
coredumpctl info PID
coredumpctl debug PID
```

Useful symbols may require matching debuginfo packages. A core can contain passwords, keys, customer data, and application content, so storage and transfer controls apply.

Valgrind can identify invalid memory access and allocation leaks in compatible user-space programs, but it changes execution speed and memory use. A `still reachable` allocation at exit is not automatically a leak. Developers should interpret loss categories, call paths, program lifetime, suppressions, and repeated growth together.
### SELinux
SELinux enforces mandatory access controls after ordinary Unix permissions and access controls. Diagnosis should preserve enforcing mode and test discretionary access first:

```shell
getenforce
ls -lZ /path
ps -eZ
ausearch -m AVC,USER_AVC -ts recent
journalctl -t setroubleshoot
```

An access failure with no AVC record often originates in ownership, mode, an ACL, a read-only mount, application logic, or another control. If policy denies the action, the AVC record identifies the source context, target context, object class, and requested permission.

The administrator should ask whether the file has the expected location and label, whether a supported Boolean enables the intended behaviour, and whether the application follows a standard RHEL deployment pattern. `semanage boolean -l` lists Boolean descriptions, while `getsebool -a` shows current values. `setsebool -P NAME on` changes a Boolean persistently after its security effect has been reviewed.

`chcon` makes a direct label change that a later relabel can discard. Persistent local file-context rules use `semanage fcontext`, followed by `restorecon`:

```shell
semanage fcontext -a -t httpd_sys_content_t '/srv/site(/.*)?'
restorecon -RFv /srv/site
```

Administrators should check package defaults and existing rules before adding a local rule. Broad recursive relabelling can alter unrelated content and take a long time.

`audit2why` can explain a denial, and `audit2allow` can generate candidate policy. Administrators must not install generated permission without review. A denial may reveal a mislabelled file, unsafe application action, or unsupported layout rather than a missing policy rule. Disabling SELinux hides that distinction and weakens the host.
### Containerised applications
The current EX342 objectives include containerised application troubleshooting. RHEL 10 uses Podman and associated tools for daemonless containers. Diagnosis begins with container, image, process, network, mount, and log state:

```shell
podman ps -a
podman images
podman logs CONTAINER
podman inspect CONTAINER
podman top CONTAINER
podman stats --no-stream
podman events --since 1h
```

Administrators should inspect an exited container before removal. Its exit code, state, command, mounts, health check, and logs can separate an application crash from an image, resource, or orchestration fault.

Rootless and rootful Podman use different storage, network state, sockets, and privileges. The administrator must run inspection under the owning account. User namespaces can map a container UID to a different host UID, so numeric ownership inside and outside the container can differ.

Bind mounts commonly fail because of host permissions or SELinux labels. The `:Z` mount option assigns a private label, while `:z` permits sharing between containers. Either option changes host labels, so the administrator should understand the directory's other consumers before using it.

Published ports require a listening process inside the container, a correct port mapping, host reachability, and firewall permission. The administrator should compare `podman port`, container network details, host listeners, and a request from both host and remote client.

Immutable image practice favours rebuilding an image from a corrected Containerfile rather than editing a running container. `podman exec` can gather evidence, but an unrecorded live edit disappears when the container is replaced. Quadlet or systemd configuration should declare persistent run-time settings. Journal and unit checks apply when systemd manages the container.
## Authentication and identity
Authentication proves an identity. Authorisation decides what that identity may do. Name Service Switch supplies account and group lookups, PAM applies authentication and session policy, and SSSD can connect both functions to identity providers such as Identity Management and Active Directory.

The administrator should keep a tested local recovery account and console path before changing remote identity configuration. A failed remote login can result from networking, DNS, time, TLS trust, Kerberos, SSSD, NSS, PAM, access policy, the shell, the home directory, SSH policy, or the identity provider.
### Authselect and PAM
RHEL 10 requires `authselect` for supported PAM and NSS profile management. Red Hat has removed the former `authconfig` workflow. Authselect owns generated authentication files, so later updates can overwrite direct edits, and direct edits can cause integrity checks to fail:

```shell
authselect current
authselect check
authselect list
authselect list-features PROFILE
```

An administrator should select a suitable supported profile and feature set, or create a custom profile when a genuine policy requirement cannot use the standard templates. The administrator should test the resulting change in a separate session before the existing privileged session closes.

PAM stacks use module types such as `auth`, `account`, `password`, and `session`. Control values determine how a module result influences the stack:
- `required` records failure but normally continues through the stack.
- `requisite` returns immediately on failure.
- `sufficient` can return success when no earlier required module has failed.
- `optional` contributes only when no other module determines the result.
- Bracketed controls express explicit actions for individual return values.

Order changes behaviour. Copying a line into the wrong stack can bypass a control, deny every user, or run a session action twice. Administrators should inspect included stacks and use authselect templates instead of making isolated edits.

`pam_faillock` records failed attempts and can lock an account according to policy. Diagnosis should distinguish a lock from a bad password, expired account, access-provider denial, or unavailable identity service:

```shell
faillock --user USER
pamtester SERVICE USER authenticate
journalctl -b | grep -Ei 'pam|faillock'
```

`pamtester` may require an additional package and should run only under an approved test account. Clearing a lock removes a security control, so the administrator should confirm identity and cause before using `faillock --user USER --reset`.
### SSSD, DNS, and Kerberos
Basic identity checks should compare the system lookup with SSSD and provider state:

```shell
getent passwd USER
id USER
sssctl config-check
sssctl domain-list
sssctl domain-status DOMAIN
sssctl user-checks USER
systemctl status sssd
journalctl -u sssd -b
```

`getent` follows NSS configuration. A provider may resolve a user while PAM still denies access, so `id` alone does not prove that login will succeed. `sssctl user-checks` helps compare identity and access decisions.

SSSD configuration requires root ownership and restrictive permissions. Domain logs under `/var/log/sssd/`, the SSSD journal, and provider logs can reveal discovery, TLS, Kerberos, access, and cache faults. Raising debug level increases sensitive output and disk use, so it should remain temporary.

Kerberos depends on correct host and realm names, service records, forward and reverse DNS where the deployment requires them, synchronised time, reachable key distribution centres, valid keytabs, and matching encryption policy. Useful tests include:

```shell
timedatectl
chronyc tracking
dig _kerberos._udp.REALM SRV
kinit USER
klist
kvno SERVICE/FQDN
```

`kinit` tests ticket acquisition. `kvno` tests access to a service principal. Neither proves that the final application accepts the session. Administrators should protect ticket caches, keytabs, bind passwords, and diagnostic bundles.

RHEL 10 no longer supports enumerating all users and groups from Active Directory or Identity Management through SSSD. Direct lookup of a known identity remains the appropriate test. A missing full directory listing is therefore not proof of failure.

Cached credentials can permit offline authentication under configured policy. Purging caches can remove that access and useful evidence. The administrator should first establish provider reachability, cache status, and business impact.
## Advanced evidence and escalation
### Sos reports
The `sos` utility gathers configuration, logs, package data, command output, and subsystem state into a support archive. A standard collection uses:

```shell
sos report
```

The administrator should reproduce the fault where safe, record the time, collect the report before logs rotate, and retain the generated archive checksum. The administrator can adjust plug-in selection and command timeouts when the ordinary report omits a required subsystem or stalls.

A sos report can contain hostnames, addresses, user names, configuration, logs, certificates, and application data. Administrators should review handling requirements, store the archive securely, and transfer it only to authorised recipients. Obfuscation can reduce selected identifiers but cannot guarantee complete anonymisation.

An archive preserves evidence for support, but it does not replace a concise incident description. Escalation should include the symptom, impact, start time, reproduction, environment, recent changes, tests, results, suspected layer, and archive identifier.
### Kdump
Kdump loads a capture kernel into reserved memory. After a panic, that kernel writes a `vmcore` for analysis. A configured service cannot work without sufficient `crashkernel` reservation and an accessible dump target.

Administrators should inspect reservation, configuration, target capacity, and service state:

```shell
cat /proc/cmdline
grep -i crash /proc/iomem
kdumpctl status
kdumpctl estimate
cat /etc/kdump.conf
systemctl status kdump
```

The target can be local or remote and must remain available in the restricted capture-kernel environment. Network dumps require the necessary drivers, routes, authentication, and storage. Filters and compression can reduce dump size, but filtering can remove evidence needed for a specific analysis.

After changing the reservation or early-boot requirements, the administrator should apply the supported RHEL 10 configuration method and reboot. A status check after reboot must confirm that kdump loaded. Administrators should never declare a production host protected on configuration text alone.

Testing kdump by forcing a kernel crash destroys the running workload and can expose latent storage faults. Only an approved, disposable, or carefully isolated system should undergo that test, with current backups, console access, a known dump destination, and an agreed recovery plan. A normal status check cannot prove the complete panic path, but an uncontrolled crash is not an acceptable shortcut.

`crash` can analyse a matching `vmcore` with the corresponding uncompressed `vmlinux` debug image. Kernel version, debuginfo, and dump must align. Common analysis identifies the panic reason, current task, stack traces, loaded modules, memory state, and blocked work. Specialist kernel analysis may require Red Hat Support.
### SystemTap
SystemTap instruments a running kernel and user-space processes through scripts. Tapsets provide reusable probe definitions and helper functions. They are libraries for scripts, not stand-alone kernel modules.

A matching kernel development and debuginfo environment is essential. `stap-prep` helps install dependencies for the running kernel when the required repositories are available. A small test should precede a broad probe:

```shell
stap-prep
stap -v -e 'probe begin { log("SystemTap ready") exit() }'
stap -l 'kernel.function("vfs_*")'
```

`stap -l` lists matching probes. A script should specify a narrow event, collect only required fields, and define an exit condition. High-frequency probes and expensive handlers can change the workload or overwhelm output.

SystemTap can compile a script through pass 4 into a module for the selected kernel and then run it with `staprun`. The selected kernel build determines module compatibility, and the module should remain a diagnostic artefact. Administrators should not copy it into `/lib/modules` or configure it as a permanent boot driver.

Secure Boot and kernel lockdown can prevent loading an unsigned instrumentation module. Administrators must resolve signing, privilege, compiler, debuginfo, and production policy requirements through supported controls. They should not weaken Secure Boot or system integrity controls for convenience.

Tracing can expose file names, arguments, user data, keys, and timing details. It can also destabilise a busy host. The administrator should test scripts on a comparable non-production system, set resource bounds, keep console access, and stop the probe as soon as sufficient evidence exists.
## Final verification
A repair is complete only when the original requirement works under the intended identity and access path. The administrator should:
- Repeat the original failing operation.
- Check the relevant service and kernel logs for new warnings.
- Confirm configuration syntax, ownership, permissions, and SELinux labels.
- Confirm active sockets, routes, mounts, dependencies, and resource limits.
- Verify that package and automation state match the intended baseline.
- Reboot when persistence forms part of the task.
- Recheck the service after reboot without relying on an interactive shell.
- Remove temporary debug settings, captures, credentials, and recovery access.
- Record the cause, change, result, and residual risk.

A successful command is one observation. End-to-end service, persistence, security controls, and absence of new faults provide the stronger result.