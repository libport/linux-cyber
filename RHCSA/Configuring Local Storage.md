# Configuring Local Storage
> [!NOTE]
> This file is a practical RHEL local storage administration guide that explains how to inspect block devices, use loop-backed disks and partitions, create persistent file systems with stable identifiers, manage LVM for flexible online growth, and configure swap safely.
## Linux block storage
A Linux block storage path starts with hardware or a disk image and ends with a device file in `/dev`. The kernel presents storage through device drivers, and user space works with the resulting block device like any other file. Common examples include SATA, SCSI, USB, SSD, iSCSI, and loop-backed devices.

`lsblk` lists block devices, their sizes, mount points, and their relationships. It can show a whole-tree view, a single device, or a reverse path back to the parent device. `lsmod` shows loaded kernel modules, and `modinfo` reports details about a specific driver. In a typical RHEL installation, the `sd_mod` driver handles many disk types that appear as `/dev/sdX` devices.

Major and minor numbers identify device classes and instances. The major number maps a device to its driver, while the minor number distinguishes individual devices and partitions within that driver class. This matters for inspection and troubleshooting, but day-to-day administration usually relies on stable device names, UUIDs, labels, and LVM paths rather than on the numbers themselves.

A baseline installation often shows a physical disk with a small boot partition and an LVM-backed root and swap layout beneath a larger partition. `lsblk` makes that hierarchy clear because it displays the parent disk, its partitions, and any child device-mapper volumes. That view helps an administrator answer three questions quickly. Which device holds the data. Which layer exposes the consumable block device. Where the storage is mounted.

Inspection usually starts before any change. A block layout that looks obvious from device names alone can become less obvious after new storage appears, after a loop device is attached, or after a VG begins to expose several LVs. Reading the tree before editing it prevents mistakes, especially when a lab mixes physical disks, loop devices, partitions, and logical volumes on the same host.
## Loop devices
A loop device maps a regular file to a block device. That mapping makes an ordinary file behave like a disk, which is useful for labs, testing, ISO access, and temporary storage workflows. `losetup` creates, lists, and detaches loop devices.

A common pattern uses `losetup -f` to find the next available loop device and attach it to an image file. After attachment, `lsblk` shows the loop device, and `mount` can mount it if the file contains a recognised file system. ISO images are a simple example because they can be attached to a loop device and mounted read-only at `/mnt`.

Loop devices also provide a safe way to rehearse destructive operations. A regular file can be partitioned, turned into an LVM PV, reformatted, extended, or discarded without touching the host system disk. That makes loop-backed storage a useful teaching tool because the workflow stays close to real administration while the risk stays low.

Loop devices persist only while the mapping exists. Unmounting a file system does not detach the loop device. Detachment requires `losetup -d <device>` for one loop device or `losetup -D` for all loop devices.
## Creating raw disk files
A lab can create disk images with either `dd` or `fallocate`.

- `dd` writes data block by block, often from `/dev/zero`, and creates a fully written file.
- `fallocate` allocates space quickly and usually creates large empty files much faster.
- `time` measures each command and makes the performance difference obvious.

For empty lab disks, `fallocate` is usually the quicker and simpler choice. After creation, `losetup` can attach the file to a named loop device such as `/dev/loop1`. Named attachment improves consistency in demonstrations and automation because the storage path stays predictable.
## Partitioning strategy
A disk does not need partitions. A file system can sit directly on a whole disk or loop device. Partitioning becomes useful when a system needs separate areas for boot files, root, variable data, home directories, or swap, or when an administrator wants clearer isolation and easier recovery.

RHEL commonly works with two partition table formats.

- MBR, also called MS-DOS, supports up to four primary partitions, or three primary partitions plus one extended partition that can contain logical partitions. Its practical size limit is about 2 TiB for standard 512-byte sectors.
- GPT supports very large disks and far more partitions, and it is the modern default for most systems.

Partitioning is a design choice rather than a reflex. A separate boot partition can simplify bootloader requirements. A separate `/var` can protect the root file system from log growth. A separate home area can isolate user data from system files. At the same time, too many small fixed partitions can create future bottlenecks that LVM would avoid. Good partitioning reflects system purpose rather than habit.

`fdisk` offers an interactive workflow that suits manual work. `parted` supports both an interactive shell and direct non-interactive commands, which makes it better for scripting and automation. A normal workflow creates a disk file, attaches it to a loop device, writes a partition table, creates one or more partitions, and then informs the kernel of the changes.
## Updating the kernel after partition changes
The kernel does not always pick up new partition information immediately. `partprobe` tells the kernel to re-read the partition table. On loop-backed disks, this step is especially important after recreating a loop mapping or after changes to partitions. Without it, the parent device may appear, but partition devices such as `/dev/loop1p1` may not.
## File systems, labels, and UUIDs
A partition or whole device becomes usable only after a file system or swap header is written. For ordinary data storage, a common RHEL choice is XFS. A file system can also receive a label.

A label is readable and convenient, but it is not guaranteed to be unique. A UUID is designed to be unique and is the safer persistent identifier for mounts. `blkid` displays UUIDs, labels, and file system types.

This distinction matters because device names can change. A disk that appears as `/dev/sda` at one boot might appear differently after hardware changes, storage reordering, or use of the next available loop device. Mounting by UUID avoids that fragility. In `/etc/fstab`, a UUID-based entry keeps a file system mount stable even if the underlying device name changes.

A normal persistent mount entry specifies the source, mount point, file system type, mount options, dump field, and fsck field. For XFS, the final fsck pass field is normally `0` because `fsck.xfs` does not perform the kind of boot-time check used for ext file systems.

The same persistence logic applies during troubleshooting. A mount that fails because a directory is missing or an entry is malformed is easy to repair. A mount that fails because a transient device name no longer points to the expected file system is harder to diagnose. Stable identifiers reduce that uncertainty, especially on hosts that add and remove removable media, virtual disks, or loop-backed images.
## Persisting loop-backed storage
A loop-backed lab device disappears across a reboot unless the system recreates it. A persistent setup can use a small `systemd` service.

The service should run after udev has detected hardware and before local file systems mount. A practical design uses `Type=oneshot`, runs `losetup` to recreate the mapping, then runs `partprobe` so the kernel sees the partition table. Enabling the service under `local-fs.target` integrates the loop mapping into the boot sequence.

This approach solves only the device recreation problem. File systems still mount most safely by UUID. If the loop number changes unexpectedly, UUID-based mounts continue to work after the service recreates the backing device and the kernel re-reads the partition table.
## Logical Volume Manager
LVM adds a flexible abstraction layer over block storage.

- Physical volumes, or PVs, represent the underlying storage devices or partitions.
- Volume groups, or VGs, pool free space from one or more PVs.
- Logical volumes, or LVs, carve usable block devices from the pooled VG space.

RHEL commonly installs the root and swap areas on LVM by default. `lsblk` and `dmsetup` show these device-mapper relationships. LVs appear as block devices and can be formatted, mounted, extended, or used as swap.

A useful mental model separates storage roles clearly. PVs store real data and LVM metadata. VGs aggregate capacity. LVs present the consumable block devices. Device-mapper then exposes those volumes under `/dev/mapper` and through convenient symbolic links such as `/dev/<vg>/<lv>`.

This arrangement turns fixed storage into a pool that can be reassigned when requirements change. Instead of binding an application permanently to one partition on one disk, the administrator can allocate only the needed space, add capacity later, and reshape the layout without replacing every higher-level reference. That is the central reason LVM remains valuable on general-purpose Linux systems.
## Marking partitions for LVM
A partition intended for LVM can carry the LVM flag. In `parted`, that flag documents the partition's intended role and helps tools recognise the layout. The flag does not by itself create a PV or make the partition part of a VG. Actual LVM membership starts only when an LVM command writes the required metadata.
## Core LVM commands
The LVM command set follows a clear naming pattern.

- `pvs`, `pvdisplay`, and `pvcreate` manage physical volumes.
- `vgs`, `vgdisplay`, `vgcreate`, and `vgextend` manage volume groups.
- `lvs`, `lvdisplay`, `lvcreate`, and `lvextend` manage logical volumes.

These commands provide both concise and detailed views. Scan commands such as `pvs`, `vgs`, and `lvs` summarise the current state quickly. Display commands expand the view and expose details such as metadata copies, extent size, free space, and logical volume attributes.
## Creating an LVM layout
A typical LVM workflow follows these steps.

1. Create or identify a partition or whole device for LVM.
2. Initialise it as a PV with `pvcreate`, or let `vgcreate` initialise it automatically when no special PV options are needed.
3. Create a VG with `vgcreate`.
4. Create one or more LVs with `lvcreate`.
5. Put a file system or swap header on each LV as required.
6. Mount file-system LVs or activate swap LVs.

An LV can be addressed through `/dev/<vg>/<lv>` or `/dev/mapper/<vg>-<lv>`. Both paths normally resolve to the same device-mapper node. Because those names are stable within the file system namespace, they are often practical mount sources.
## Physical extents and LV sizing
LVM allocates storage in physical extents. The extent size belongs to the volume group and is set when the VG is created. The default is commonly 4 MiB, though it can be changed with `vgcreate -s`.

This affects LV sizing.

- `lvcreate -L` specifies a human-readable size such as megabytes or gigabytes.
- `lvcreate -l` specifies a count or percentage of extents.

When an LV size does not divide evenly into the extent size, LVM rounds it to an extent boundary. That behaviour explains why a requested size such as 100 MiB may become slightly larger in the final LV.

Extent sizing also affects planning. Smaller extents allow finer-grained allocation. Larger extents reduce metadata overhead and may align better with specialised designs. On ordinary systems, the default usually works well, but understanding extents explains why LVM sometimes reports sizes that differ slightly from the requested number.
## Viewing and activating logical volumes
`lvs` reports LV state and attributes. In a new lab VG, an LV may exist before it carries a file system or mount point. After formatting and mounting, it becomes an online storage resource. `vgchange -ay <vg>` activates all logical volumes in a volume group if activation is needed.

Once active, an LV can receive an XFS or ext4 file system, mount to a chosen directory, and behave like any other block device. Stable LV paths often make explicit UUID use optional for those mounts, though UUIDs remain valid and portable.
## Online growth of logical volumes
The main practical advantage of LVM is online growth.

When a file system begins to run short of space, the administrator first checks whether the VG still has free extents. `vgs` or `vgdisplay` shows that information. If free space exists, `lvextend` can enlarge the LV.

With `lvextend -r`, RHEL can grow the file system at the same time for supported file systems such as XFS and ext4. This reduces disruption because the LV and its file system grow together while mounted. The workflow usually looks like this.

1. Check free space in the VG.
2. Extend the LV by size or by extents.
3. Allow `-r` to resize the file system.
4. Verify the result with `lvs`, `vgs`, and `df -h`.

The command can extend by a fixed amount, such as `+104M`, or by available extents, such as `+100%FREE` when using the extent form. This makes capacity management far more flexible than traditional fixed partitions.

The workflow stays safe because growth works upward through clear layers. First the VG must have free extents. Then the LV can grow. Then the file system can claim the new space. Verification at each layer matters because a larger LV without a larger file system still leaves the mounted capacity unchanged from the user's perspective.
## Extending a volume group
If the VG has no free extents, the LV cannot grow until the VG grows first. `vgextend` adds capacity by enrolling another PV into the VG. That PV can be a newly prepared partition, disk, or loop-backed lab device.

After `vgextend`, the new free extents become available to all LVs in the group. The administrator can then run `lvextend` on the target LV and, if needed, grow the file system in the same operation.

This model avoids the old pattern of replacing one full disk with a larger disk and copying data across. Instead, LVM allows incremental growth by adding storage to the VG and assigning only the required extra space to the LV that needs it.
## Swap space
Swap extends virtual memory by providing disk-backed pages when RAM pressure rises. In RHEL, swap can live on a partition, a regular file, or an LV. Systems often use an LV for swap by default.

Swap does not use a normal file system. `mkswap` writes the swap area header and creates the UUID and label metadata that tools use to identify the swap device. `swapon` enables swap, `swapoff` disables it, and `free -h` or `swapon --show` reports current usage.

A practical workflow for an additional swap LV is straightforward.

1. Create the LV.
2. Run `mkswap` on the LV.
3. Add an entry to `/etc/fstab` if the swap should persist across reboots.
4. Enable it with `swapon -a` or a direct `swapon` command.

Swap priority influences which swap areas the kernel prefers. Higher priority values are used first. That can matter when a system combines multiple swap areas with different performance characteristics.

Swap should be sized and used deliberately. A system that never touches existing swap usually does not need more just because free VG space exists. The administrator should inspect real memory pressure before adding or extending swap. In practice, swap configuration is part of performance policy rather than a substitute for adequate RAM.
## Choosing identifiers for mounts and swap
Persistent storage configuration should prefer identifiers that remain stable across reboots and hardware changes.

- Use UUIDs for ordinary partitions, loop-backed file systems, and swap entries when device names may change.
- Use LV paths when the VG and LV naming scheme is stable and unambiguous.
- Avoid hard-coding transient names such as `/dev/loop0` unless a boot service recreates them predictably.

This principle runs through the entire storage workflow. The more dynamic the lower layers become, the more valuable stable identifiers become at the mount and activation layer.
## Practical command set
The storage workflow relies on a compact set of commands.

- Inspection uses `lsblk`, `lsmod`, `modinfo`, `blkid`, `pvs`, `vgs`, `lvs`, `vgdisplay`, `lvdisplay`, `free -h`, and `swapon --show`.
- Loop work uses `losetup`, `mount`, `umount`, and `partprobe`.
- Disk-image creation uses `fallocate`, `dd`, and `time`.
- Partitioning uses `fdisk` and `parted`.
- LVM work uses `pvcreate`, `vgcreate`, `vgextend`, `lvcreate`, `lvextend`, and `vgchange`.
- Swap work uses `mkswap`, `swapon`, and `swapoff`.

These commands cover most of the administrative path from a blank disk image to an online file system or swap device.
## Administrative pattern
A disciplined storage workflow follows a repeatable order.

1. Inspect the current block layout.
2. Create or attach storage.
3. Partition only where partitioning adds value.
4. Write the file system or swap header.
5. Mount or enable the storage.
6. Persist the configuration with stable identifiers.
7. Extend capacity later at the VG and LV layers rather than redesigning the whole layout.

This pattern reduces ambiguity, keeps boot behaviour predictable, and makes later expansion far simpler.
## Summary
RHEL local storage administration begins with block devices and device drivers, then moves through loop devices, partitioning, file systems, persistent identifiers, LVM, and swap. `lsblk` reveals the storage tree. `losetup` turns files into block devices for labs and test workflows. `fdisk` and `parted` write partition tables. UUIDs stabilise mounts when device names move. `systemd` can recreate loop-backed storage during boot. LVM pools capacity into VGs, presents LVs as flexible block devices, and allows online growth with minimal disruption. Swap extends virtual memory and can live on partitions, files, or logical volumes.

The overall design favours abstraction without losing control. Files can stand in for disks. Partitions can document intent. UUIDs can outlast device renumbering. LVM can absorb new storage and grow live file systems. Taken together, those features make RHEL local storage both methodical and adaptable.