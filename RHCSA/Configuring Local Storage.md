# Configuring Local Storage
> [!NOTE]
> How to configure, manage, persist, and expand RHEL local storage using block devices, loop devices, partitions, file systems, stable identifiers, LVM, and swap.

Red Hat Enterprise Linux 8 presents local storage through block devices. An administrator can inspect those devices, create file-backed lab disks, partition storage, build filesystems, configure persistent mounts, manage Logical Volume Manager 2 storage, and provide swap space. Most operations require root privileges, and partitioning, filesystem creation, and LVM initialisation can destroy existing data. The administrator should identify every target with `lsblk`, confirm that required data has a backup, and test configuration changes before rebooting.
## Block devices, drivers, and device numbers
Linux represents storage devices as special files under `/dev`. Applications use normal input and output operations against these files, while the kernel and a device driver communicate with the underlying hardware or virtual device. Common examples include:
- `/dev/sda` for a disk handled through the SCSI disk layer
- `/dev/sda1` for the first partition on that disk
- `/dev/loop0` for a loop device backed by a regular file
- `/dev/mapper/vg-lv` for a device-mapper volume such as an LVM logical volume

The `sd_mod` module provides the SCSI disk upper layer. Linux also routes many SATA, USB mass-storage, and iSCSI devices through the SCSI subsystem, but it does not route every storage technology through `sd_mod`. NVMe devices, for example, use NVMe drivers and names such as `/dev/nvme0n1`.

Several commands reveal the relationship between devices and drivers:

```shell
lsmod
modinfo sd_mod
lsblk
lsblk /dev/sda
lsblk -s /dev/sda2
```

`lsmod` lists loaded kernel modules. `modinfo` reports a module's metadata, parameters, aliases, and file location. `lsblk` displays block-device topology, including names, sizes, types, mount points, and major and minor device numbers. The `-s` option reverses the dependency view and traces a child device towards its parent.

A major number identifies the kernel device class or driver interface associated with a device node. A minor number selects a device or subdivision managed through that interface. The pair identifies a device node, although Linux can assign major numbers dynamically and can allocate several major-number ranges to one subsystem.

Traditional `sd` device numbering reserves 16 minor numbers for each disk. The whole disk uses one number, leaving up to 15 partition numbers for that `/dev/sdX` device. The next disk starts at the next group of 16. This device-node allocation limit does not define the capacity of the partition-table format itself.
## Loop devices and file-backed storage
A loop device exposes a regular file as a block device. It supports useful lab work when a system has no spare disk, and it also provides access to filesystem images and ISO images. A loop device remains an abstraction over the backing file. It does not copy the file or create new storage.

The administrator can associate an ISO image with the next available loop device and display the selected name:

```shell
sudo losetup --find --show --read-only image.iso
sudo losetup --list
sudo mount -o ro /dev/loop0 /mnt
```

ISO 9660 images normally mount read-only. After use, the administrator should unmount the filesystem before detaching the loop device:

```shell
sudo umount /mnt
sudo losetup --detach /dev/loop0
```

The `--detach-all` option detaches all unused loop devices, but broad cleanup can affect unrelated work. A named detach operation provides safer control.

Raw backing files support partitioning and filesystem exercises. `dd` writes data through the normal input and output path, while `fallocate` asks the filesystem to preallocate space. `fallocate` often completes much faster because it need not write every zero-filled block:

```shell
time dd if=/dev/zero of=/var/disks/dd.disk bs=1M count=500 status=progress
time fallocate -l 500M /var/disks/fa.disk
```

These commands do not guarantee identical allocation behaviour on every filesystem. A sparse or copy-on-write backing file can also affect performance and reported disk usage.

When the backing file contains a partition table, `losetup --partscan` asks the kernel to scan it during attachment:

```shell
sudo losetup --find --show --partscan /var/disks/disk1
lsblk /dev/loop0
```

The kernel then exposes partitions with names such as `/dev/loop0p1`. If the loop device already exists, `partprobe /dev/loop0` can request another partition-table scan. The logical sector size influences that scan, so a non-default image may also require `losetup --sector-size`.

Loop assignments normally disappear at reboot. An administrator should not place a loop partition in `/etc/fstab` unless the boot process creates the loop device before the dependent mount. A fixed name such as `/dev/loop1` can also collide with another consumer, so fixed loop numbers suit controlled labs better than general-purpose systems.
## Partition tables and partitions
A filesystem can occupy a whole block device, but partitions divide a device into independently managed regions. Separate filesystems can isolate capacity for `/boot`, `/home`, `/var`, and application data. Isolation can prevent one workload from exhausting the root filesystem, although it also divides available capacity.

RHEL 8 commonly uses two partition-table formats:

| Format | Main characteristics |
| --- | --- |
| MBR, also called DOS | Supports four primary entries. One entry can act as an extended partition that contains logical partitions. With 512-byte logical sectors, its 32-bit sector fields address about 2 TiB. |
| GPT | Stores primary and backup headers, identifies partitions with GUIDs, and uses 64-bit logical block addresses. With 512-byte logical sectors, its theoretical address space approaches 8 ZiB. The table records its own number of entries, so GPT has no fixed limit of 255 partitions. Common implementations initially allocate 128 entries. |

The operating system, driver, hardware, and management tools can impose lower limits than the on-disk format. A `/dev/sdX` device still exposes no more than 15 partition minors under the traditional `sd` allocation described above.

`fdisk` provides an interactive interface and supports both MBR and GPT on RHEL 8. `parted` can run interactively or accept subcommands, and `sfdisk` provides a script-oriented interface. Automation should use explicit, reviewable input rather than feeding keystrokes to an interactive session.

The following sequence creates a GPT and one partition on a disposable loop device:

```shell
sudo parted --script /dev/loop0 mklabel gpt
sudo parted --script /dev/loop0 mkpart primary 1MiB 256MiB
sudo partprobe /dev/loop0
lsblk /dev/loop0
```

Starting at 1 MiB normally gives suitable alignment. `parted` accepts percentages, but explicit binary units give clearer boundaries. `parted print` or `fdisk -l` verifies the result.

An LVM partition type or flag records the intended use of a partition. LVM does not require the flag before `pvcreate`, but the metadata helps installers, discovery tools, and administrators interpret the layout:

```shell
sudo parted /dev/loop0 set 1 lvm on
```

Changing a partition table can erase access to existing data. The administrator should unmount affected filesystems, disable affected swap, deactivate dependent LVM volumes, and verify the device name before writing the table.
## Filesystems, identifiers, and persistent mounts
RHEL 8 supports XFS and ext4 as its principal local filesystems. `mkfs.xfs` creates XFS, and `mkfs.ext4` creates ext4. A filesystem can carry a human-readable label and a generated universally unique identifier:

```shell
sudo mkfs.xfs -L DATA /dev/loop0p1
sudo blkid /dev/loop0p1
```

A device name such as `/dev/sda1` can change when hardware detection order changes. A filesystem label remains easier to read, but Linux does not enforce label uniqueness. A filesystem UUID gives a stronger persistent identifier. GPT also gives each partition a PARTUUID, which differs from the UUID stored inside a filesystem.

An administrator can mount a filesystem by device name, label, or UUID:

```shell
sudo mkdir -p /data
sudo mount UUID=12345678-1234-1234-1234-123456789abc /data
findmnt /data
```

`/mnt` suits temporary mounts, while a dedicated directory such as `/data` suits an ongoing service. A persistent entry in `/etc/fstab` contains the device specification, mount point, filesystem type, options, dump field, and filesystem-check field:

```text
UUID=12345678-1234-1234-1234-123456789abc /data xfs defaults 0 0
```

The administrator should create the mount point, run `systemctl daemon-reload` after editing `fstab`, and validate the entry with `mount -a` before rebooting. `findmnt --verify` provides another useful check. XFS entries normally use `0` in the final field because the generic boot-time `fsck` sequence does not repair XFS.
## Boot-time loop setup
A systemd oneshot service can attach a loop-backed lab disk before a dependent filesystem mounts. The service should run `losetup --partscan`, stay active after its setup commands finish, and detach the device when stopped. `RemainAfterExit=yes` retains the active state. A value of `no` would let the oneshot unit return to an inactive state after successful setup.

The service must order itself before the dependent mount. The mount must also require and follow the service, rather than relying only on a broad target order. If the image resides below a separately mounted `/var`, the service must start after that backing filesystem becomes available. A controlled lab can use the following unit:

```ini
[Unit]
Description=Attach loop-backed lab disk
RequiresMountsFor=/var/disks
Before=data.mount

[Service]
Type=oneshot
ExecStart=/usr/sbin/losetup --partscan /dev/loop1 /var/disks/disk1
ExecStop=/usr/sbin/losetup --detach /dev/loop1
RemainAfterExit=yes

[Install]
WantedBy=local-fs.target
```

This example belongs in `/etc/systemd/system/loop-storage.service`. The `Before=data.mount` line identifies the mount unit for `/data`. Another mount point requires the correctly escaped mount-unit name, which `systemd-escape --path --suffix=mount` can generate. `RequiresMountsFor=/var/disks` ensures that systemd can reach the backing file before it runs `losetup`. The `fstab` mount options should include `x-systemd.requires=loop-storage.service`, which adds `Requires=` and `After=` dependencies to the generated mount unit. The administrator should choose an unused loop number and update both the unit and every dependent configuration entry.

After creating or changing the unit, the administrator runs:

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now loop-storage.service
systemctl status loop-storage.service
```

This arrangement supports a controlled training environment. Dedicated block storage, managed virtual disks, or purpose-built image units usually provide a stronger production design.
## LVM2 structure and creation
LVM2 adds a flexible allocation layer above block devices:

| Layer | Function | Common commands |
| --- | --- | --- |
| Physical volume | Initialises a disk, partition, or loop device for LVM and stores LVM metadata | `pvcreate`, `pvs`, `pvdisplay`, `pvremove` |
| Volume group | Pools capacity from one or more physical volumes | `vgcreate`, `vgs`, `vgdisplay`, `vgextend`, `vgremove` |
| Logical volume | Allocates a block device from a volume group | `lvcreate`, `lvs`, `lvdisplay`, `lvextend`, `lvremove` |

Device Mapper presents logical volumes to userspace. `dmsetup ls --tree` shows device-mapper relationships, while `lsblk`, `pvs`, `vgs`, and `lvs` provide progressively more LVM-specific views. `pvmove` relocates allocated extents between physical volumes within the same volume group. It does not move a physical volume between volume groups.

An explicit creation sequence keeps each layer visible:

```shell
sudo pvcreate /dev/loop0p1
sudo vgcreate vg1 /dev/loop0p1
sudo lvcreate -n data -L 200M vg1
sudo lvs
```

`vgcreate` establishes the physical extent size for the group. The default is 4 MiB, and every physical volume in that group uses the same extent size. An administrator can choose another size with `vgcreate -s`, but ordinary linear volumes rarely need a custom value.

`lvcreate -L 200M` requests a size in ordinary units and rounds it to whole extents. Lower-case `-l` requests a number or percentage of extents. For example, `-l 100%FREE` allocates all unassigned extents in the volume group.

LVM normally activates volume groups during boot. A loop device attached after boot may require explicit activation:

```shell
sudo vgchange -ay vg1
sudo mkfs.xfs /dev/vg1/data
sudo mkdir -p /srv/data
sudo mount /dev/vg1/data /srv/data
```

Both `/dev/vg1/data` and `/dev/mapper/vg1-data` normally resolve to the same device-mapper volume. LVM names remain stable across ordinary disk detection changes, provided the volume-group and logical-volume names remain unique in the active system.

LVM stores configuration metadata on its physical volumes. Additional metadata copies can improve recovery options, but they consume space and do not replace backups. An administrator should use `vgcfgbackup`, maintain independent data backups, and test recovery procedures.
## Extending storage online
A volume group needs free extents before it can enlarge a logical volume. `vgs` shows group capacity and free space. If the group lacks room, the administrator can initialise another block device and add it:

```shell
sudo pvcreate /dev/loop0p2
sudo vgextend vg1 /dev/loop0p2
vgs vg1
```

The plus sign in an extension request means "add this amount". Without it, LVM treats the value as the requested final size:

```shell
sudo lvextend -r -L +1G vg1/data
```

`-r`, or `--resizefs`, enlarges the logical volume and then grows a supported filesystem through the appropriate helper. RHEL 8 can grow mounted XFS and ext4 filesystems online. The administrator should still confirm free space, review the target path, keep a current backup, and check the result with `lvs`, `findmnt`, and `df -h`.

XFS can grow but cannot shrink. Ext4 can shrink only while unmounted and through a separate, carefully ordered procedure. Storage growth therefore offers less risk than reduction, but a wrong device or interrupted infrastructure operation can still cause data loss.
## Swap space
Disk-backed swap provides space for memory pages that the kernel can evict from RAM. It supports the virtual-memory system, but it does not turn storage into RAM and usually operates much more slowly. Capacity planning should follow workload needs, memory pressure, hibernation requirements, and storage performance.

RHEL 8 can use a partition, an LVM logical volume, or a regular file as swap. `mkswap` writes a swap signature and UUID, while `swapon` activates the area:

```shell
sudo lvcreate -n swap_extra -L 2G vg1
sudo mkswap /dev/vg1/swap_extra
sudo swapon /dev/vg1/swap_extra
swapon --show
free -h
```

A persistent `fstab` entry uses `none` as the mount point and `swap` as the type:

```text
/dev/vg1/swap_extra none swap defaults,pri=10 0 0
```

The kernel prefers an active swap area with a higher numeric priority. It can distribute pages across areas with equal priority. Fast storage can improve swap performance, but extra swap does not help a system that never experiences memory pressure.

`swapoff` removes an area from active use:

```shell
sudo swapoff /dev/vg1/swap_extra
```

The administrator must ensure that RAM can absorb the resident pages before disabling swap. `swapoff -a` disables every active swap area and can trigger severe memory pressure, so targeted operations provide safer control. Resizing an LVM swap volume requires `swapoff`, an LVM resize, a new `mkswap` signature, and `swapon`.
## Verification and safe sequencing
Storage administration works best as a repeated cycle of inspection, one controlled change, and verification. Each command views a different layer, so no single report proves that the whole configuration works.

| Question | Useful command |
| --- | --- |
| Which block devices and dependencies exist? | `lsblk -o NAME,MAJ:MIN,SIZE,TYPE,FSTYPE,MOUNTPOINT` |
| Which signatures and identifiers exist? | `blkid` or `lsblk -f` |
| Which filesystems are mounted? | `findmnt` |
| How much filesystem capacity remains? | `df -h` |
| How much LVM capacity remains? | `pvs`, `vgs`, and `lvs` |
| Which swap areas are active? | `swapon --show` and `free -h` |

Before creating a filesystem, partition table, physical volume, or swap signature, the administrator should confirm that the target has no required mount, holder, or existing signature. `lsblk`, `findmnt`, and `wipefs --no-act` support that check without writing to the device. After the change, the same reports should show the intended result.

Configuration files require functional tests. `findmnt --verify` checks `fstab` syntax and references, while `mount -a` attempts eligible mounts. `systemctl status` and `journalctl -u` reveal service failures. A successful command exit does not prove that boot ordering works, so a disposable lab should include a reboot test after the administrator confirms that emergency access remains available.
## End-to-end disposable lab
A disposable lab can combine loop devices, GPT partitions, XFS, LVM, online growth, and swap without adding a physical disk. The administrator should run the exercise only on newly created image files and should never substitute a production device without a verified plan.
### Create and attach the first image
The first image supplies one ordinary filesystem partition and one LVM partition:

```shell
sudo mkdir -p /var/disks
sudo fallocate -l 1GiB /var/disks/disk1
sudo losetup --partscan /dev/loop1 /var/disks/disk1
lsblk /dev/loop1
```

`fallocate` reserves the requested length quickly. The fixed `/dev/loop1` name simplifies later commands in this isolated lab. `losetup --find --show` provides safer allocation when no later configuration depends on a known name.
### Partition the image
The administrator creates a GPT, reserves the first 256 MiB for XFS, assigns the remaining capacity to LVM, and requests a fresh kernel scan:

```shell
sudo parted --script /dev/loop1 mklabel gpt
sudo parted --script /dev/loop1 mkpart primary xfs 1MiB 256MiB
sudo parted --script /dev/loop1 mkpart primary 256MiB 100%
sudo parted --script /dev/loop1 set 2 lvm on
sudo partprobe /dev/loop1
sudo parted /dev/loop1 print
lsblk /dev/loop1
```

The 1 MiB starting offset leaves room for GPT metadata and aligns the first partition. The second partition starts at the first partition's stated end. `parted print` and `lsblk` should show `/dev/loop1p1` and `/dev/loop1p2` before the exercise continues.
### Create and mount XFS
The first partition receives an XFS filesystem and a label:

```shell
sudo mkfs.xfs -L LABDATA /dev/loop1p1
sudo mkdir -p /data
sudo mount /dev/loop1p1 /data
findmnt /data
sudo blkid /dev/loop1p1
```

The `blkid` output supplies the filesystem UUID for a persistent `fstab` entry. The administrator should not add that entry until the systemd loop service can create `/dev/loop1p1` before `data.mount`. After both configurations exist, `systemctl daemon-reload`, `mount -a`, and `findmnt /data` test the dependency without a reboot.
### Build LVM storage
The second partition becomes a physical volume. A new volume group pools its extents, and a logical volume receives 300 MiB:

```shell
sudo pvcreate /dev/loop1p2
sudo vgcreate labvg /dev/loop1p2
sudo lvcreate -n files -L 300M labvg
sudo mkfs.xfs /dev/labvg/files
sudo mkdir -p /srv/files
sudo mount /dev/labvg/files /srv/files
sudo pvs
sudo vgs
sudo lvs
```

The three reports show the same storage at different layers. `pvs` associates `/dev/loop1p2` with `labvg`. `vgs` reports total and unallocated capacity. `lvs` reports the `files` logical volume. `lsblk` displays the full dependency tree from the loop device to the mounted device-mapper volume.

An `fstab` entry can identify the logical volume by `/dev/mapper/labvg-files` or by its filesystem UUID. The loop service must still run before the generated mount unit because LVM cannot activate a physical volume that does not yet exist.
### Add capacity and grow XFS
A second image demonstrates volume-group growth. LVM can use the whole loop device as a physical volume, so this image does not require a partition table:

```shell
sudo fallocate -l 512MiB /var/disks/disk2
sudo losetup /dev/loop2 /var/disks/disk2
sudo pvcreate /dev/loop2
sudo vgextend labvg /dev/loop2
sudo lvextend --resizefs --size +256M labvg/files
sudo vgs labvg
sudo lvs labvg
df -h /srv/files
```

`vgextend` adds the new physical volume to the pool. `lvextend` allocates another 256 MiB and grows XFS while it remains mounted. Existing files remain accessible throughout the operation. The reports should show a larger logical volume, a larger filesystem, and some free extents left in `labvg`.
### Add and inspect swap
The remaining volume-group capacity can supply a small swap logical volume:

```shell
sudo lvcreate -n swap_extra -L 128M labvg
sudo mkswap /dev/labvg/swap_extra
sudo swapon --priority 5 /dev/labvg/swap_extra
swapon --show
free -h
```

`swapon --show` confirms the device, size, usage, and priority. `free -h` reports aggregate swap capacity. A persistent entry can use the stable LVM path, but the boot dependency must ensure that both loop-backed physical volumes exist before LVM activation and swap activation.
### Remove the lab in reverse order
Safe removal unwinds consumers before providers. The administrator disables the added swap, unmounts both filesystems, removes any related `fstab` entries, reloads systemd, disables the loop service, and removes the LVM objects before detaching loop devices. Commands such as `lvremove`, `vgremove`, and `pvremove` erase LVM metadata and require exact targets.

```shell
sudo swapoff /dev/labvg/swap_extra
sudo umount /srv/files
sudo umount /data
sudo lvremove labvg
sudo vgremove labvg
sudo pvremove /dev/loop1p2 /dev/loop2
sudo losetup --detach /dev/loop2
sudo losetup --detach /dev/loop1
```

`lvremove labvg` asks LVM to remove the logical volumes in that group. The administrator should review the prompt and stop if the listed objects differ from the lab. Backing image files remain ordinary files after loop detachment and can be removed only after the administrator confirms that no required data remains.