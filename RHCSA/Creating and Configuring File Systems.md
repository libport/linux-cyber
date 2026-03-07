# Creating and Configuring File Systems
## Core commands and working habits
RHEL exposes block devices and file systems clearly through `lsblk`. The `lsblk -f` view shows device names, file system types, labels, UUIDs, and mount points in one place. `blkid` complements that view by reading UUIDs and labels directly from devices. These identifiers matter because a stable UUID or label survives device name changes better than `/dev/sdb` or `/dev/sdc`.

File systems are created with `mkfs` and its type-specific wrappers such as `mkfs.ext4`. Persistent mounts belong in `/etc/fstab`. Temporary mounts can be created from the command line with `mount`, but production systems usually rely on `fstab` so the layout returns after reboot. Most storage administration requires root privileges.

Metadata is part of routine administration, not just troubleshooting. Labels, UUIDs, block sizes, mount histories, and superblock details help identify devices, rebuild mount configurations, and recover from documentation loss after a failure. Good administration therefore treats metadata as operational information rather than background noise.
### Representative local file-system workflow
A practical local workflow follows a clear order:
- Inspect available storage with `lsblk` or `lsblk -f`.
- Create the required file system.
- Read metadata and record labels and UUIDs.
- Create and secure the mount point.
- Mount the file system manually for testing.
- Add the persistent `fstab` entry.
- Verify the layout after remounting or reboot.

This order reduces configuration drift. It also avoids the common mistake of writing an `fstab` entry before confirming the device, file-system type, and mount-point permissions.
## Local file systems
### XFS in RHEL
XFS is the default file system for many RHEL installations. It scales well, grows online, and integrates cleanly with logical volumes and higher-level tools such as Stratis. Administrators usually inspect existing XFS file systems before changing them.

`xfs_info` reports important metadata, including inode size and data block size. On common x86_64 systems, a 4 KiB data block size is typical. Larger blocks may suit platforms that store large sequential files, while smaller effective file sizes can increase internal slack when many tiny files occupy full blocks. The block size choice therefore affects efficiency as well as throughput.

`xfs_admin` manages identifying information. It can display or set a file system label and print the UUID. Labels help operators recognise a volume's purpose during recovery work. UUIDs provide a reliable mount target for `/etc/fstab`. Changing a label normally requires the file system to be unmounted first. In practice, short descriptive labels work best because they remain readable in command output and operational notes.

RHEL administration therefore begins with discovery. `lsblk -f`, `xfs_info`, and `xfs_admin` reveal what exists, how it is formatted, and how it should be mounted.

XFS metadata also supports day-to-day verification. Labels and UUIDs provide a second identity beyond the kernel device name. In disaster recovery, that second identity can matter more than the device node itself because cabling changes, controller order, or virtual-machine reconfiguration can shift `/dev/sdX` names. Administrators who rely only on positional device names make recovery harder than it needs to be.

The practical XFS toolset therefore serves two purposes. It helps with configuration when the system is healthy, and it provides evidence when the system must be reconstructed under pressure.
### EXT4 in RHEL
EXT4 remains common in RHEL and other Linux distributions. It offers mature tooling, predictable behaviour, and straightforward metadata management. Administrators can create an EXT4 file system on a whole device with either `mkfs -t ext4 /dev/sdb` or `mkfs.ext4 /dev/sdb`.

`dumpe2fs` displays EXT4 metadata in detail, including superblock information, mount counts, and check intervals. `tune2fs` adjusts settings after creation and can assign a label with `-L` or print metadata with `-l`. These tools make EXT4 easy to inspect and document.

The material emphasises one practical issue. Automated file system checks can occur at awkward times if mount-count or interval triggers are active. On the demonstrated system, the maximum mount count was disabled and the check interval was set to zero, which prevented routine forced checks. Scheduled maintenance remains the safer way to run file system checks on production systems.

EXT4 metadata also records where a file system was last mounted. That history can help when an administrator inherits an unfamiliar server or recovers a damaged one. Combined with a meaningful label, the superblock tells a clear story about a device's intended role.
### Mount points and persistent mounts
A mount point is just a directory until a file system is mounted on it. That detail matters for security. The directory belongs to the underlying root file system when the target file system is absent. Users who can enter that directory while it is unmounted can create files in the wrong place and then lose sight of them when the real file system is mounted again.

- Create the mount point directory.
- Restrict the unmounted directory to root, typically with `0700`.
- Mount the file system.
- Apply the intended permissions to the root of the mounted file system.

This pattern prevents users from writing into the bare mount point during outages or maintenance. It also ensures that the visible permissions come from the mounted file system rather than from the underlying directory.

Persistent mounting belongs in `/etc/fstab`. Using `UUID=` entries avoids problems when kernel device names shift. A typical entry specifies the UUID, mount point, file system type, mount options, and the final dump and fsck fields. For the demonstrated EXT4 file system, `defaults 0 0` avoided automatic checks through the sixth field and left the obsolete dump mechanism disabled.

Manual testing still matters before an `fstab` entry is trusted. A safe workflow mounts the file system explicitly, confirms ownership and permissions, unmounts it, then runs `mount -a` after adding the persistent entry. If `mount -a` succeeds without errors and the expected mount appears in `mount` or `findmnt`, the configuration is usually ready for reboot testing.
### Extending logical volumes and file systems
RHEL commonly places the root file system on LVM. This layout makes expansion straightforward. `vgs` shows the available volume groups and free extents. `vgextend` adds another physical device, such as `/dev/sdc`, to the chosen volume group. `lvextend` then grows the logical volume.

The key operational shortcut is `lvextend -r`. The `-r` option resizes the file system together with the logical volume, provided the file system supports online growth. XFS and EXT4 both support the demonstrated workflow. Administrators can therefore add storage to a live system and expand the root file system without taking it offline.

This model separates concerns cleanly. Physical storage enters the volume group first. Logical space then expands. The file system grows last, either automatically with `lvextend -r` or explicitly with the correct file-system-specific tool. Capacity planning remains necessary, but the process is fast and low risk when the underlying storage is sound.

Growth still depends on correct layering. If extra space is added to the wrong volume group or the wrong logical volume, the file system cannot use it. Administrators therefore verify the target path, volume-group name, and current free extents before growing anything. The process is fast, but it still rewards discipline.
## Collaborative permissions
Shared storage needs more than ordinary `rwx` bits. RHEL uses special mode bits to control deletion behaviour and group inheritance in collaborative directories.
### Understanding the special bits
Linux mode values contain four octal digits. The rightmost three digits set user, group, and other permissions. The leftmost digit controls special bits.

- `1` sets the sticky bit.
- `2` sets SGID.
- `4` sets SUID.

For collaborative directories, the important values are the sticky bit and SGID. SUID mainly matters for executable files rather than shared directories.

`ls -l` shows these bits inside the execute positions. Lowercase `t` or `s` means the execute bit is also present. Uppercase `T` or `S` means the special bit is set but execute is not. Administrators therefore need to read both the symbol and its case.

`find` helps audit special permissions at scale. A search such as `find /path -type d -perm /1000` locates directories with the sticky bit, while similar searches can locate SGID directories or directories that combine both. That matters on larger estates where shared areas proliferate over time.
### Managing groups for collaboration
Collaborative storage usually begins with a dedicated group such as `sales`. `groupadd sales` creates the group. `gpasswd -a username sales` adds a user to it. `id` confirms the current account's user ID, primary group, and supplementary groups.

A practical detail often surprises administrators. New supplementary group membership does not appear in an existing login session until the user logs out and back in, or starts a new session that reads the updated group list. Access checks will continue to reflect the old session context until that happens.
### Auditing shared directories
Special permissions become harder to manage as the number of shared paths increases. `find` gives administrators a quick way to inspect the estate.

- `find /path -type d -perm /1000` locates sticky-bit directories.
- `find /path -type d -perm /2000` locates SGID directories.
- `find /path -type d -perm -3000` locates directories that have both bits set.

These searches are useful during hardening, migration, and post-incident review because they expose directories whose collaboration model may not match current policy.
### Using the sticky bit correctly
Users need write permission on a directory to create files in it. That same directory write permission also allows deletion of entries. In a shared workspace, that means one user can remove another user's files unless an extra control is applied.

The sticky bit solves that problem. When it is set on a directory, users can delete only files they own, unless they are root. A shared directory can therefore remain writable without becoming vulnerable to accidental or malicious deletion by peers.

A typical collaborative directory starts by changing its group ownership to the shared group and then setting group write permissions. If administrators stop there, any group member can remove any file. Adding the sticky bit preserves collaboration while restricting deletion to the file owner.
### Using SGID on directories
SGID fixes a different problem. Without SGID, new files usually inherit the creator's primary group. In a collaborative directory that can break access immediately, especially when users employ a restrictive `umask` such as `007`.

When SGID is set on the directory, every new file inherits the directory's group owner instead. In a `sales` directory, new files therefore become group-owned by `sales` regardless of each creator's primary login group. This makes shared access predictable.

The most useful collaborative pattern is to combine SGID with group-writable permissions and, where required, the sticky bit. For example, `3770` creates a directory that is accessible to the owner and group, preserves the shared group on new files, and prevents users from deleting files they do not own. This pattern works well for team workspaces that exclude everyone outside the group.

A common sequence is therefore to set the directory group to the collaboration group, apply SGID so new files stay in that group, and add the sticky bit if many users will create content in the same path. This creates a stable shared area instead of a directory that slowly fills with files owned by inconsistent groups.
## Network file sharing with NFS
NFS extends local file-system management across the network. In RHEL, the demonstrated model uses an RHEL server and a CentOS Stream client on a private lab network.
### NFSv4 server configuration
The `nfs-utils` package provides both NFS server and client components. RHEL includes `nfsconf`, which writes settings to the newer NFS configuration layout. The practical recommendation is to run NFSv4 only where possible.

Restricting the service to NFSv4 simplifies firewalling. NFSv3 relies on additional helper services and ports, while NFSv4 traffic can be handled through TCP port 2049. The demonstrated configuration therefore enables version 4 and disables version 3 and UDP-based behaviour that older deployments often used.

Once configured, the server is started with `systemctl`. Firewall access is then opened with `firewall-cmd --add-service=nfs`, and the runtime rules are written to the permanent configuration. This keeps the service reachable after reboot.
### Practical NFS workflow
A clean NFS rollout usually follows this sequence:
- Confirm server and client connectivity.
- Install `nfs-utils` on both systems.
- Limit the server to NFSv4 where older clients are not required.
- Open the firewall for the NFS service.
- Define the export.
- Reload exports.
- Test with a manual client mount.
- Replace the manual mount with `autofs` if on-demand access is preferable.

This sequence keeps troubleshooting narrow. Each step either works or reveals the next problem clearly.
### Exporting a shared directory
NFS shares are exports. They can be defined directly in `/etc/exports`, but RHEL also supports drop-in files under `/etc/exports.d/` with the `.exports` suffix. A file such as `sales.exports` keeps the configuration tidy.

The demonstrated export shares `/data/sales` to a private subnet with read-write access and synchronous writes. The safer default is `root_squash`, which maps remote root access to an unprivileged identity. `no_root_squash` exists for special cases, but it expands risk because it grants remote root users root-level access on the server-side file system. Production systems should avoid it unless there is a clear operational need and compensating controls.

`exportfs -r` reloads export definitions after changes. Client testing should follow immediately so that permission problems, path mistakes, and firewall gaps appear while the configuration is still fresh.

The export options are not decorative. They define trust and write behaviour. `rw` allows change, `sync` makes write acknowledgement more conservative, and root-squashing controls how much authority a remote root account carries. A permissive export may appear convenient during testing, but it usually expands the blast radius of any client-side compromise.
### Client access and autofs
A client can mount an NFSv4 export manually with `mount -t nfs4 server:/data/sales /mnt`. Manual mounting is useful for testing, but regular user environments benefit more from automounting.

`autofs` provides on-demand mounting. The client installs the package, defines a master map, and points that map to a file such as `auto.data`. The map entry then links a local path, such as `sales`, to the remote export and its mount options. When a user enters the mapped path, `autofs` mounts the share automatically. After a period of inactivity it unmounts the path again.

This model reduces boot-time delays and avoids keeping unused network file systems mounted permanently. It also makes remote storage appear at familiar directory paths only when needed.

Permissions still matter across the network. If the server expects group-based access, the client users need compatible group membership and, in some cases, matching numeric group IDs. Shared access policies do not become simpler merely because the storage is remote.


`autofs` also reduces the operational cost of stale mounts. Workstations and application hosts do not need to wait for remote storage that users may never touch during a session. The automounter also clears idle mounts after its timeout, which reduces clutter in `mount` output and lowers the chance of hanging processes tied to forgotten network paths.
### SELinux considerations
SELinux remains part of the operational picture. It is a mandatory access control system, not just a file permission extension. NFS exports and server daemons can therefore fail even when Unix permissions and firewall rules look correct.

The demonstrated workflow uses `sepolicy` documentation to inspect policy material for the NFS daemon domain. That approach encourages administrators to troubleshoot denials with policy awareness instead of disabling SELinux. In production, SELinux should remain enabled and be configured correctly.
## Data optimisation with VDO
VDO adds a block-layer optimisation stage beneath the file system. It performs compression and deduplication before data reaches physical storage. It is therefore not a file system by itself. Administrators still create a file system on top of the VDO device.
### Installing and creating a VDO volume
VDO requires the userspace tools and the kernel support module. After installation, `vdo.service` must run before the system can use VDO devices.

A VDO device is created on a block device, such as `/dev/sdc`, with a chosen logical size that can exceed the physical capacity. The demonstrated volume uses an 8 GiB physical device but presents 20 GiB of logical space through `vdo create`. That design assumes the data will compress well or contain repeated blocks.

`vdo status` reports key properties, including whether compression and deduplication are active. After the VDO device exists, administrators format `/dev/mapper/vdo1` with a file system. XFS is used in the demonstration, created with `mkfs.xfs -K`.
### Demonstrated VDO workflow
The VDO sequence has four layers.

- Install the VDO packages and start `vdo.service`.
- Create the VDO device on the chosen block device with the required logical size.
- Format `/dev/mapper/<name>` with a file system.
- Mount and monitor the resulting file system.

This matters because VDO sits below the file system. Administrators who skip the formatting step do not have usable storage yet. Administrators who skip monitoring may fill the underlying physical device while the logical file system still appears to have room.
### Mounting and monitoring VDO
Because VDO depends on a background service, the mount configuration needs an explicit service dependency. In `/etc/fstab`, the VDO mount uses the option `x-systemd.requires=vdo.service`. That ensures the system starts the VDO service before attempting to mount the file system.

The mount-point security pattern remains the same as for ordinary file systems. The underlying directory should stay private to root when the file system is absent. After mounting, administrators can apply the required ownership and permissions to the VDO-backed file system itself.

Monitoring matters more with VDO than with ordinary partitions because logical capacity can exceed physical reality. `vdostats --human-readable` reports usage and reveals how much physical space the device still has. The demonstrated system also notes that VDO keeps working space and metadata on the physical device, so apparent free space on the file system does not remove the need to watch the underlying pool closely.
### Growing a VDO-backed file system
VDO logical space can grow later. The device is enlarged first with `vdo growLogical`. That changes the virtual block device size but does not automatically enlarge the file system on top of it.

XFS must then be told to consume the extra space with `xfs_growfs`. Only after that step does `df` show the increased capacity. This sequence mirrors other layered storage workflows. Lower layers grow first. The topmost file system grows last.

Compression and deduplication ratios depend on workload. Virtual machine images and containers often contain many repeated blocks and therefore benefit strongly. Workloads full of already-compressed or highly unique objects benefit less. Sensible overprovisioning depends on that data profile, so administrators should size VDO conservatively until real usage patterns are known.

The demonstration deliberately copies repeated large files into the VDO-backed file system to show why deduplication changes the capacity story. Many logical files do not always mean many unique physical blocks. The opposite is also true. A workload full of unique media or already-compressed archives can consume physical space much faster than its logical size suggests.
## Layered storage with Stratis
Stratis aims to simplify storage management by exposing pools, file systems, and snapshots through one command set. It hides much of the complexity that would otherwise require separate LVM and file-system workflows.
### How Stratis works
Stratis uses `stratisd` as its backend service and `stratis` as its command-line interface. Under the surface, it creates XFS file systems on thin-provisioned device-mapper storage. Administrators do not need to manage those layers directly for routine work.

A Stratis pool aggregates one or more block devices. From that pool, administrators create file systems without manually creating logical volumes, formatting block devices, or tuning a separate thin pool by hand. This is the main value of Stratis in RHEL. It reduces operational steps while preserving modern storage features.
### Stratis operating pattern
Stratis is most useful when administrators want modern storage features with less manual assembly. Instead of juggling physical volumes, volume groups, logical volumes, file systems, and snapshot commands across separate layers, they can work from one pool-centric interface.

That does not remove the need to understand the stack. It changes where the day-to-day work happens. Administrators still benefit from knowing that Stratis ultimately relies on XFS and thin-provisioned device-mapper storage, especially when capacity planning or troubleshooting becomes more complex.
### Creating pools and file systems
The software is installed with `stratisd` and `stratis-cli`, then the `stratisd.service` unit is enabled and started. A pool can then be created with `stratis pool create pool1 /dev/sdd`. Additional devices can be added later if the pool needs more space.

A file system is created with `stratis filesystem create pool1 fs1`. The file system can then be mounted from its Stratis path and persisted in `/etc/fstab`. As with VDO, the mount should depend on the backend service, using `x-systemd.requires=stratisd.service`.

`stratis pool list` and `stratis filesystem list` display the available pools and file systems, while normal tools such as `df` still show the file-system view of capacity and usage. Administrators therefore combine Stratis-native status with ordinary Linux file-system monitoring.
### Snapshots and recovery workflows
Stratis supports snapshots as point-in-time copies of existing file systems. A snapshot such as `snap1` can be created from `pool1/fs1`, mounted separately, and used for inspection, recovery, or backup-style operations.

This makes short-term rollback and file restoration straightforward. Administrators can mount the snapshot at a directory such as `/backup`, copy required data back into the live file system, and then unmount and destroy the snapshot when it is no longer needed.

Because the underlying storage is thin-provisioned, snapshots use space efficiently compared with a full duplicate copy. They still consume capacity as changed blocks accumulate, so they should be monitored and managed deliberately rather than left to grow without oversight.

Snapshots are especially useful before risky maintenance or bulk content changes. A snapshot does not replace a full backup strategy, but it can provide a fast local recovery point. When teams need a short rollback window or a safe point-in-time copy for extraction, Stratis snapshots reduce the time and complexity involved.
## Operational model
RHEL storage administration becomes predictable when it follows a few principles.

- Discover devices and metadata before changing anything.
- Prefer UUIDs and clear labels over unstable device names.
- Secure bare mount points so users cannot write into them when storage is absent.
- Use groups, SGID, and the sticky bit to shape collaboration deliberately.
- Export NFS shares with conservative options and automate client mounts where appropriate.
- Treat VDO as an optimising block layer that still needs careful capacity monitoring.
- Use Stratis when simplified pool, file-system, and snapshot management is more valuable than manual control of each underlying layer.

With those habits in place, RHEL can manage local storage, shared team directories, network exports, deduplicated block devices, and snapshot-based layered storage without turning routine administration into ad hoc recovery work.

These layers are complementary rather than interchangeable. Ordinary XFS and EXT4 still suit many direct mounts. NFS extends those file systems to other hosts. VDO optimises block usage beneath a file system. Stratis simplifies the administration of pools, file systems, and snapshots above the block layer.