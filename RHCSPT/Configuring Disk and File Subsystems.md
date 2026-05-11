# Configuring Disk and File Subsystems
## Disk scheduler selection
The Linux disk scheduler orders read and write requests before they reach storage. Red Hat Enterprise Linux 8 supports multi-queue schedulers only, which helps the block layer scale on solid-state drives and multi-core systems.

| Scheduler | Main use |
|---|---|
| `none` | Uses first-in, first-out scheduling. It suits high-performance SSDs, CPU-bound systems with fast storage and NVMe. Leave NVMe devices on `none` unless testing proves a need to change. |
| `mq-deadline` | Groups requests into read and write batches, then schedules them by logical block address. It suits most workloads, especially asynchronous writes. |
| `bfq` | Favours low latency and responsiveness over maximum throughput. It suits desktops, interactive systems and large file copies. |
| `kyber` | Tunes for a latency target by measuring each I/O request. It suits fast, low-latency NVMe and SSD storage. |

The kernel selects a default scheduler by device type. Administrators should keep it unless workload analysis justifies a change. The active scheduler appears in square brackets in the device scheduler file.

```bash
cat /sys/block/device/queue/scheduler
```

`mq-deadline kyber bfq [none]` means `none` is active.
## Persistent scheduler changes
Administrators use TuneD or udev for persistent scheduler changes. Both methods should target specific devices by stable identifier, preferably a World Wide Name. If no WWN exists, `ID_SERIAL_SHORT` can identify the device.

TuneD applies scheduler settings through a custom profile. The profile needs a directory, a device identifier, a `[disk]` section in `tuned.conf`, activation and verification.

```bash
udevadm info --query=property --name=/dev/device | grep -E '(WWN|SERIAL)'
sudo mkdir /etc/tuned/sch_profile
sudo vi /etc/tuned/sch_profile/tuned.conf
sudo tuned-adm profile sch_profile
tuned-adm verify
```

A profile can include a baseline such as `virtual-guest`, then set `devices_udev_regex` and `elevator` for the selected device.

udev rules also persist across reboots. A rule in `/etc/udev/rules.d/99-scheduler.rules` matches the block device by identifier, sets `ATTR{queue/scheduler}`, reloads the rules and triggers a device change.

```bash
ACTION=="add|change", SUBSYSTEM=="block", ENV{ID_SERIAL_SHORT}=="value", ATTR{queue/scheduler}="mq-deadline"
sudo udevadm control --reload-rules
sudo udevadm trigger --type=devices --action=change
```
## File system choice
File system design should match workload, storage type, file sizes, growth and availability needs. Poor choices can force later rebuilds.

XFS is the default and generally recommended local file system in RHEL 8. It scales well for large files, large volumes, enterprise workloads and parallel I/O. It supports online growth, dynamic inodes and fast repair, but it cannot shrink.

ext4 remains useful where compatibility, familiarity or smaller systems matter. It supports growth and reduction, although reduction requires offline access. Its supported limits are lower than XFS.

NFS shares files over a network. SMB supports Microsoft Windows file sharing. GFS2 provides shared write access for clustered systems, but it adds complexity and suits only specific high-availability requirements. Prefer network file systems unless shared block access is required.
## XFS creation, growth and repair
Create an XFS file system with `mkfs.xfs`. The default options suit most common cases.

```bash
sudo mkfs.xfs /dev/nvme1n1p1
```

On hardware RAID, specify stripe geometry only when automatic detection is wrong. `su` sets the RAID chunk size. `sw` sets the number of data disks.

```bash
sudo mkfs.xfs -d su=64k,sw=4 /dev/md0
```

Grow a mounted XFS file system with `xfs_growfs`. The command takes the mount point, not the block device path. Without `-D`, it expands to the maximum size.

```bash
sudo xfs_growfs /mount-point
```

Use `xfs_repair -n` for a read-only check. Repair an unmounted file system with `xfs_repair`. If the log is dirty, mount and unmount the file system first so XFS can replay it. Use `-L` only as a last resort because it can discard in-progress metadata updates.

```bash
sudo xfs_repair -n /dev/nvme2n1p4
sudo xfs_repair /dev/nvme2n1p4
```