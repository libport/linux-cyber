# Creating and Configuring File Systems
> [!NOTE]
> A practical framework for creating, securing, sharing, optimising, and expanding RHEL file systems using XFS, EXT4, LVM, NFS, VDO, Stratis, and stable mount configurations.

Red Hat Enterprise Linux 8 supports XFS and ext4 as its principal local file systems. XFS is the default and suits large, high-performance storage. Ext4 remains a mature alternative with broad tooling and the ability to shrink an unmounted file system. RHEL 8 also provides LVM for flexible allocation, NFS for network sharing, VDO for block-level deduplication and compression, and Stratis for pool-based storage management.

Storage administration affects availability and data integrity. Administrators should confirm device identities, maintain current backups, test configuration changes, and monitor capacity at every layer. Commands that create file systems, initialise physical volumes, wipe signatures, or destroy snapshots erase data on the selected devices.
## Creating and managing local file systems
### Identifying devices and file systems
Linux represents disks, partitions, LVM logical volumes, and device-mapper targets as block devices. A mounted file system attaches one of these devices to a directory in the unified directory tree. The following commands provide complementary views:

```shell
lsblk --fs
blkid
findmnt
```

`lsblk --fs` associates block devices with file-system types, labels, UUIDs, and mount points. `blkid` reads persistent identifiers directly from devices. `findmnt` reports the active mount hierarchy and its options. Device names such as `/dev/sdb` can change when hardware changes, so persistent configuration should normally use a file-system UUID or another stable identifier.

XFS and ext4 maintain metadata that describes their geometry, features, labels, identifiers, and state. The relevant inspection commands include:

```shell
xfs_info /mount-point
xfs_admin -l /dev/sda1
xfs_admin -u /dev/sda1

dumpe2fs -h /dev/sdb1
tune2fs -l /dev/sdb1
```

In `xfs_info` output, `isize` identifies the size of each on-disk inode record. It is not the data-block size assigned to a file. The `bsize` value identifies the file-system data-block size. On common x86-64 RHEL 8 systems, the memory page size usually limits an XFS block to 4 KiB. Architectures with larger page sizes can support larger XFS blocks. Small files can therefore consume more allocated space than their byte length suggests, although sparse allocation and other file-system behaviour can change the observed result.

Labels provide readable descriptions, while UUIDs provide stronger uniqueness. A label can indicate a purpose such as `SALES_DATA`, but duplicate labels can cause ambiguity. Changing a UUID or label can invalidate `/etc/fstab`, boot-loader, backup, or monitoring configuration that refers to the old value. RHEL requires an XFS file system to be unmounted before `xfs_admin` changes these attributes. After a change, `udevadm settle` allows device metadata updates to complete.

```shell
umount /boot
xfs_admin -L BOOT_RHEL8 /dev/sda1
udevadm settle
mount /boot

tune2fs -L SALES_DATA /dev/sdb1
udevadm settle
```

XFS labels contain at most 12 characters. Ext file-system labels contain at most 16 characters. Both tools can truncate an overlong label, so an administrator should verify the stored value with `blkid` or `lsblk --fs`.
### Creating XFS and ext4 file systems
`mkfs` dispatches to a file-system-specific formatter. The specialised commands make the intended type explicit:

```shell
mkfs.xfs -L SALES_DATA /dev/sdb1
mkfs.ext4 -L SALES_DATA /dev/sdb1
```

Only one of these commands belongs on a given device. Each command overwrites existing file-system metadata and can make previous data inaccessible. A whole disk can hold a file system, but a partition, logical volume, or other managed block device often provides clearer allocation and growth boundaries.

XFS uses journal recovery and `xfs_repair` for checking and repair. The generic `fsck.xfs` command does not perform a conventional boot-time scan. Ext4 uses journal recovery after an unclean shutdown and `e2fsck` for deeper offline checking. In ext4 metadata, a maximum mount count of `-1` and a check interval of `0` disable those two periodic triggers. Disabling both does not replace health monitoring, backups, or planned maintenance checks.

Repair tools require a clear diagnosis and a current backup. An administrator normally unmounts a damaged non-root file system before repair. `xfs_repair -n` performs a read-only assessment of XFS metadata, while `e2fsck -n` answers repair prompts negatively and reports ext-family problems. A read-only check can still place heavy load on failing media, so hardware errors, kernel messages, and storage-controller evidence need review before a long scan. XFS log replay normally requires mounting and unmounting the file system on the same architecture. The destructive `xfs_repair -L` option clears a corrupt log and can lose metadata updates, so it belongs only in a recovery decision that accepts that loss.

File-system metadata also supports reconstruction after documentation fails. `lsblk --fs`, `blkid`, labels, UUIDs, LVM names, and mount histories can help associate devices with intended paths. None proves that a candidate contains the latest data. Recovery work should preserve original media, record commands and results, and avoid assigning a new UUID until duplicate identifiers have been assessed.
### Securing mount points
A mount-point directory belongs to its parent file system until another file system is mounted there. After the mount, the root directory of the mounted file system supplies the visible owner, group, permissions, and contents. Existing files under the mount point become hidden until unmounting.

An inaccessible underlying directory prevents users from writing into the empty mount point when the intended file system is offline. Without that protection, applications can write data into the parent file system, fill the wrong volume, and hide the misplaced files after the intended mount returns.

```shell
mkdir -p /data/sales
chown root:root /data/sales
chmod 0700 /data/sales

mount /dev/sdb1 /data/sales
chgrp sales /data/sales
chmod 2770 /data/sales
```

The first permission change protects the underlying directory. The second applies to the root directory of the mounted file system. An administrator should unmount the file system and inspect the underlying directory if data appears to have been written while the mount was absent.
### Persistent mounts
Each `/etc/fstab` entry has six fields: device, mount point, file-system type, mount options, `dump` backup flag, and `fsck` pass number. The fifth field controls the legacy `dump` utility. The sixth controls boot-time checking order. A value of `0` disables an `fsck` pass, `1` normally identifies the root file system, and `2` identifies other file systems that require checking. XFS entries normally use `0` because XFS does not use a conventional boot-time `fsck`.

An ext4 entry can use a stable UUID and a non-root check pass:

```fstab
UUID=11111111-2222-3333-4444-555555555555 /data/sales ext4 defaults 0 2
```

After editing `/etc/fstab`, the administrator should make systemd regenerate its mount units, test the entry, and verify the result:

```shell
systemctl daemon-reload
mount /data/sales
findmnt /data/sales
```

`mount -a` tests all eligible entries, but it can also affect unrelated mounts. Testing the named mount point limits the scope. A successful `mount` command does not confirm appropriate ownership, free space, or application access, so those checks still follow.

Boot-critical entries require extra care. A typing error, an unavailable device, or an unsuitable dependency can send the system into emergency mode. The `nofail` option lets boot continue when a non-essential device is absent, while `x-systemd.device-timeout=` limits how long systemd waits for that device. Neither option suits storage that an application must have before it starts. Service dependencies should state that requirement explicitly.

The administrator should also confirm that the mounted device matches the intended UUID and that no files remain hidden beneath the mount point:

```shell
findmnt --output SOURCE,TARGET,FSTYPE,OPTIONS /data/sales
umount /data/sales
ls -la /data/sales
mount /data/sales
```
### Extending LVM storage and its file system
LVM separates physical volumes, volume groups, and logical volumes. `vgs` displays volume-group information. It does not scan for volume groups, which is the role of `vgscan`. Before assigning a new disk, the administrator must confirm that the selected device contains no required data and uses a compatible logical block size.

```shell
pvcreate /dev/sdc
vgextend rhel_rhel8 /dev/sdc
vgs rhel_rhel8
```

`lvextend` increases a logical volume. The lower-case `-r`, or `--resizefs`, also invokes the appropriate file-system growth tool. The plus sign in `+100%FREE` means to add all currently free extents. Omitting it can express a different target.

```shell
lvextend --resizefs --extents +100%FREE /dev/rhel_rhel8/root
```

Mounted XFS and ext4 file systems can grow online when their underlying logical volumes grow. XFS cannot shrink. Ext4 can shrink only while unmounted, and the operation requires a separate, carefully ordered workflow. Allocating every free extent to one logical volume leaves no reserve for other volumes, snapshots, or emergency growth. A fixed increment such as `--size +10G` often preserves greater operational flexibility.
## Collaborative directory permissions
Linux mode bits divide ordinary permissions among the owner, group, and others. A leading octal digit adds three special bits:

| Octal value | Name | Main effect |
| --- | --- | --- |
| `4` | set-user-ID | An executable can run with the file owner's effective identity |
| `2` | set-group-ID | A directory causes new entries to inherit its group ownership |
| `1` | sticky bit | A directory restricts deletion and renaming of entries |

Set-user-ID and set-group-ID on executables carry significant security risk because they grant the program an effective identity derived from the file. Set-group-ID on a directory serves a different purpose. It keeps new files and subdirectories in the directory's collaborative group instead of the creator's primary group.

The sticky bit does not mean that each user can delete only files that the user owns. In a sticky directory, deletion and renaming remain available to the file owner, the directory owner, and a suitably privileged process. The bit also does not stop an authorised user from modifying a file whose own permissions allow writing.
### Establishing a shared directory
A collaborative directory needs a group, appropriate membership, inherited group ownership, and a suitable `umask`. The following pattern gives the `sales` group full directory access, removes access for others, applies set-group-ID, and applies the sticky bit:

```shell
groupadd sales
gpasswd --add analyst sales

chown root:sales /data/sales
chmod 3770 /data/sales
```

The `3` combines set-group-ID and the sticky bit. Some teams should use `2770` instead, because the sticky bit prevents ordinary group members from deleting or renaming entries owned by colleagues. The directory owner, workflow, and retention policy determine the appropriate choice.

An existing login session does not automatically gain a newly assigned supplementary group. A fresh login establishes the new group list. `newgrp sales` can start a shell with the selected group without a complete logout, although it changes the shell's primary group.

A restrictive `umask` keeps new files private from users outside the collaboration group:

```shell
umask 0007
touch /data/sales/report.txt
ls -l /data/sales/report.txt
```

The set-group-ID directory supplies the `sales` group. The `umask` removes permissions for others while retaining owner and group permissions allowed by the creating program. A `umask` removes requested permissions, but it cannot add permissions that an application did not request.

Directory write permission controls creation, deletion, and renaming of directory entries. File write permission controls changes to file content. A user can therefore delete a read-only file from a writable, non-sticky directory even without write permission on the file. Conversely, a user who can write a file might still lack permission to rename it because the containing directory is not writable. Collaborative designs must evaluate both levels.

A short test with two non-privileged accounts exposes mistakes that a root-only test misses. Each account should create a file, read the colleague's file, attempt an authorised update, and attempt a deletion that policy should reject. `namei -l /data/sales/report.txt` displays permissions on every path component, which helps locate a missing execute bit on a parent directory.

`ls -ld` displays special bits in the execute positions. Lower-case `s` or `t` means that the corresponding execute bit is also set. Upper-case `S` or `T` means that the special bit is set while the corresponding execute bit is absent. An upper-case result often indicates a configuration that requires review.
### Finding special permissions
GNU `find` distinguishes between any requested bit and all requested bits:

```shell
find /data -type d -perm /3000
find /data -type d -perm -3000
find /data -type d -perm -1000 -perm -0002
```

The first command finds directories with either set-group-ID or the sticky bit. The second finds directories with both bits. The third finds world-writable sticky directories. These searches support audits, but the administrator must still inspect ownership, access-control lists, mount options, SELinux labels, and the operational purpose of each result.
## Sharing file systems with NFS
NFS allows a client to mount a server directory within its local directory tree. RHEL 8 clients negotiate the highest protocol version offered by the server unless a mount option selects a version. NFSv4.2 supports current NFSv4 features and can reduce firewall complexity. A true NFSv4-only server requires more than disabling UDP because NFSv3 can use both TCP and UDP.
### Configuring an NFSv4-only server
The `nfs-utils` package supplies server and client tools:

```shell
dnf install nfs-utils
```

The current Red Hat procedure configures protocol versions in `/etc/nfs.conf`. Some RHEL 8 releases also provide `nfsconf` for editing that file. Direct configuration keeps the active values visible:

```ini
[nfsd]
vers3=n
vers4.0=n
vers4.1=n
vers4.2=y
```

When individual NFSv4 minor versions appear, the configuration should not also set the broad `vers4` parameter. An administrator who needs 4.0 or 4.1 clients should enable those minor versions rather than forcing 4.2.

A complete NFSv4-only configuration also disables NFSv3-related services:

```shell
systemctl mask --now rpc-statd.service rpcbind.service rpcbind.socket
```

Red Hat's procedure additionally prevents `rpc.mountd` from listening for NFSv3 mount requests through a systemd drop-in, then reloads systemd and restarts `nfs-mountd`. Once that configuration is complete, the NFS service uses TCP port 2049 for ordinary NFSv4 traffic. NFSv3 requires extra RPC services and firewall openings.
### Exporting a directory
Exports can reside in `/etc/exports` or in files ending with `.exports` under `/etc/exports.d/`. A network should use CIDR notation rather than an address containing an asterisk:

```exports
/data/sales 192.168.33.0/24(rw,sync,root_squash)
```

`rw` permits writes, `sync` makes the server complete the required stable-storage work before replying, and `root_squash` maps client root access to an anonymous identity. `root_squash` is the default and protects the server from a client administrator who controls local root. `no_root_squash` removes that boundary and should appear only where a documented trust model requires it.

The server can load, display, and expose the configuration with:

```shell
exportfs -rav
exportfs -v
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
systemctl enable --now nfs-server
cat /proc/fs/nfsd/versions
```

The export's file-system permissions still apply. A group-writable export commonly uses set-group-ID on the exported directory. With the default `AUTH_SYS` security flavour, consistent user and group identities across clients and servers prevent files from appearing under unintended numeric IDs. Central identity management scales better than manually matching local account numbers. Kerberos security flavours such as `krb5i` or `krb5p` add stronger authentication and, with `krb5p`, privacy.

An IP-based export rule limits eligible client addresses but does not authenticate a human user. Network filtering, host controls, identity management, and NFS security flavour form separate controls. DNS names in export rules require reliable forward and reverse resolution. CIDR networks avoid name-resolution ambiguity in a small isolated lab, but production rules should grant only the required hosts or subnets.

`exportfs -rav` reports parsing and export errors when it reloads the table. `exportfs -v` confirms the effective options. The server should also verify that the intended directory is a mounted file system rather than an empty underlying mount point:

```shell
findmnt --target /data/sales
showmount --exports localhost
```

`showmount` relies on RPC services associated with older NFS discovery and can be unhelpful on a tightly restricted NFSv4-only server. The active export table and a real client mount provide stronger verification.
### Mounting from a client
The client installs `nfs-utils`, creates a mount point, mounts the export, and verifies the negotiated options:

```shell
dnf install nfs-utils
mkdir -p /mnt/sales
mount -t nfs4 server.example.com:/data/sales /mnt/sales
findmnt /mnt/sales
```

A write test should run as the intended non-root account. Testing only as client root exercises squashing rather than the identity and permissions used during normal access.

For a persistent client mount, `/etc/fstab` can identify the server export and the `_netdev` option:

```fstab
server.example.com:/data/sales /mnt/sales nfs4 rw,hard,_netdev 0 0
```

The `_netdev` option marks the mount as network-dependent. A hard mount preserves NFS retry semantics during a transient outage, but a server failure can block processes that access the mount. Operational monitoring and recovery procedures must account for that behaviour.
### Mounting on demand with autofs
`autofs` creates mounts when a process accesses a configured path and removes idle mounts later. This approach avoids maintaining many inactive NFS mounts.

```shell
dnf install autofs
```

An indirect master map in `/etc/auto.master.d/data.autofs` identifies the parent path and its map:

```text
/data /etc/auto.data
```

The `/etc/auto.data` map identifies the key, mount options, and NFS export:

```text
sales -fstype=nfs4,rw,hard server.example.com:/data/sales
```

The resulting path is `/data/sales`. The `hard` behaviour allows NFS operations to retry when the server becomes temporarily unavailable. A `soft` mount does not place a slow mount into the background. It returns an I/O error after retransmission limits expire and can expose applications to partial operations or data corruption. It therefore requires a specific application-level justification.

```shell
systemctl enable --now autofs
ls /data/sales
```

Accessing the key triggers the mount. The configured timeout controls later unmounting. Active references, open files, or current working directories can keep the mount busy.
### SELinux
SELinux is a mandatory access-control system, not a mandatory access-control list. It evaluates labels and policy in addition to discretionary owner, group, and mode permissions. An NFS configuration can therefore fail even when UNIX permissions appear sufficient.

RHEL 8 supplies service-specific SELinux documentation in the `selinux-policy-doc` package:

```shell
dnf install selinux-policy-doc
man -k _selinux
man nfsd_selinux
```

`ls -Z`, `ps -eZ`, and recent AVC records help identify label or policy failures. Administrators should use documented labels and booleans, retain enforcing mode, and avoid broad local allow rules that conceal the underlying configuration error.
## Optimising storage with VDO
VDO is a thinly provisioned block-storage target that performs block-level deduplication and compression before writing data to its backing device. It is not a file system. XFS, ext4, LVM, or another supported upper layer uses the logical device that VDO exposes under `/dev/mapper/`.

Deduplication hashes incoming blocks and uses the UDS index to identify likely duplicates. VDO verifies a match before sharing physical storage. Compression acts on unique blocks that remain after deduplication. Workloads with repeated unencrypted blocks can save substantial space. Client-side encryption makes blocks appear unrelated and should therefore sit below VDO when the design requires both encryption and data reduction. Placement changes the security boundary and requires an architecture-level decision.

These examples use the standalone `vdo` service and command set available in RHEL 8. Systems built with LVM-VDO use the corresponding LVM workflow.
### Creating and mounting a VDO volume
The packages provide the management tools and kernel module:

```shell
dnf install vdo kmod-kvdo
systemctl enable --now vdo
```

The backing device should use a persistent path and support later growth. The logical size can exceed physical capacity because VDO expects deduplication and compression to reduce physical use:

```shell
vdo create \
  --name=vdo1 \
  --device=/dev/disk/by-id/example-device \
  --vdoLogicalSize=20G
```

The overprovisioning ratio must follow measured workload behaviour. Red Hat uses 10:1 as a planning example for active virtual-machine or container storage and 3:1 for object storage. These ratios are starting assumptions, not guarantees. Encrypted, compressed, or already unique data can produce little reduction.

VDO reserves physical space for metadata, including its UDS index and at least one slab. It does not reserve a universal 4 GB decompression buffer. Available physical capacity depends on the backing size, index, slab geometry, metadata, and stored data.

The file-system formatter should suppress discard during initial creation because a new VDO volume contains no allocated blocks to release:

```shell
mkfs.xfs -K /dev/mapper/vdo1
mkdir -p /data/vdo
mount /dev/mapper/vdo1 /data/vdo
```

For ext4, the corresponding option is `mkfs.ext4 -E nodiscard`. The XFS `-K` option means no discard during formatting. It is not a general quick-format switch.

A local standalone VDO device uses a normal persistent mount entry:

```fstab
/dev/mapper/vdo1 /data/vdo xfs defaults 0 0
```

A network-dependent backing device also needs `_netdev`. The mounted file system then requires the same ownership and permission controls as any other local file system.
### Monitoring capacity and reduction
`df` reports logical file-system capacity. It cannot show how close the VDO backing store is to physical exhaustion. `vdostats` reports physical use, available capacity, and space saving:

```shell
df -h /data/vdo
vdostats --human-readable
vdo status --name=vdo1
```

Repeated copies of identical files demonstrate deduplication because the file system records every logical copy while VDO stores matching blocks once. Production workloads rarely duplicate whole files so neatly, but virtual-machine images and containers can share many identical blocks.

Thin provisioning creates a critical failure mode. Applications can see free logical space after physical capacity is almost exhausted. Administrators should alert well before the backing store fills, investigate changes in reduction ratio, and retain a tested expansion path. Red Hat uses 80 per cent physical use as an example alert threshold.

Compression and deduplication are enabled by default on a newly created standalone VDO volume. Diagnostic testing can toggle them independently:

```shell
vdo disableDeduplication --name=vdo1
vdo enableDeduplication --name=vdo1
vdo disableCompression --name=vdo1
vdo enableCompression --name=vdo1
```

Changing these settings affects new writes. It does not immediately rewrite every previously stored block. Benchmarking should use representative data, queue depth, write policy, memory pressure, and failure recovery rather than repeated copies alone.

Deleting files does not allow VDO to reclaim physical blocks until the file system issues discard requests. Periodic trimming provides a controlled approach:

```shell
systemctl enable --now fstrim.timer
```
### Growing VDO
Logical growth changes the size presented to the upper layer:

```shell
vdo growLogical --name=vdo1 --vdoLogicalSize=40G
xfs_growfs /data/vdo
```

`xfs_growfs` takes the mounted XFS path. Growing VDO alone does not enlarge the file system. An ext4 file system instead uses `resize2fs` after the logical device grows.

Physical growth follows a different order. The administrator first enlarges the underlying partition, logical volume, or other backing device, then informs VDO:

```shell
vdo growPhysical --name=vdo1
```

VDO cannot shrink its logical or physical size through these workflows. Adding an unrelated disk directly to a VDO target is not the growth operation. A multi-device design normally places VDO on expandable storage such as an LVM logical volume.
## Managing layered storage with Stratis
Stratis presents storage pools and file systems through a single management interface. The `stratisd` service builds the underlying layers from Linux device-mapper technologies and XFS. Thin provisioning, snapshots, caching, and encryption remain pool-level capabilities.

Red Hat classifies Stratis as a Technology Preview in its RHEL 8 documentation. Technology Preview features do not receive production service-level support, so production adoption requires an explicit support and risk decision.
### Creating a pool and file system
The daemon and command-line client install separately:

```shell
dnf install stratisd stratis-cli
systemctl enable --now stratisd
```

A Stratis data device must be unused, unmounted, free of required data, and at least 1 GB. Existing file-system, RAID, or partition signatures can prevent use. `wipefs --all` erases those signatures and belongs only on a verified disposable device.

```shell
wipefs --all /dev/sdd
stratis pool create pool1 /dev/sdd
stratis pool list
```

Current RHEL 8 syntax can create a named, thinly provisioned XFS file system with an explicit size:

```shell
stratis filesystem create --size 10GiB pool1 fs1
stratis filesystem list pool1
```

Stratis file systems grow within the pool subject to their configured limits and pool capacity. A large logical size does not create equivalent physical capacity. `stratis pool list`, `stratis filesystem list`, and ordinary file-system tools provide different views, so capacity monitoring must include the pool as well as mounted file systems.

The pool aggregates one or more data devices and has a fixed physical total until an administrator adds capacity. Thin provisioning allocates backing space as file systems consume data. Pool exhaustion can affect several file systems at once, so alerting should reserve enough headroom for ordinary growth, snapshots, XFS logs, and recovery work. An application quota within XFS limits a consumer but does not create additional pool space.

Stratis publishes stable device links under `/dev/stratis/`:

```shell
mkdir -p /data/stratis
mount /dev/stratis/pool1/fs1 /data/stratis
findmnt /data/stratis
```

The correct path begins with `/dev/stratis/`, not `/stratis/`. A persistent non-root mount also needs the Stratis pool setup service. After obtaining the pool UUID from `stratis pool list`, an `/etc/fstab` entry follows this pattern:

```fstab
/dev/stratis/pool1/fs1 /data/stratis xfs defaults,x-systemd.requires=stratis-fstab-setup@POOL_UUID.service,x-systemd.after=stratis-fstab-setup@POOL_UUID.service 0 0
```

An encrypted pool can pause boot for a passphrase unless an unattended unlocking method such as NBDE or TPM2 is configured. Encryption must be selected when the pool is created.

Stratis command syntax and features changed across RHEL 8 minor releases. The installed `stratis-cli` and `stratisd` versions must remain compatible, and the local `stratis(8)` manual defines the accepted syntax. A tested maintenance procedure should cover service upgrades, pool start and stop behaviour, encrypted-pool unlocking, and recovery after an interrupted boot.

Additional verified devices can extend pool capacity:

```shell
stratis pool add-data pool1 /dev/sde
stratis pool list
```

This action expands the pool but does not replace monitoring. Thinly provisioned file systems can collectively claim more logical capacity than the pool physically holds.
### Snapshots
A Stratis snapshot is a first-class, read-write Stratis file system created from another file system at a point in time:

```shell
stratis filesystem snapshot pool1 fs1 snap1
mkdir -p /backup
mount /dev/stratis/pool1/snap1 /backup
```

The snapshot and its origin have independent lifetimes. Each snapshot consumes backing space for its XFS log and for changed data. Applications that require transaction-level consistency should quiesce or flush their state before snapshot creation.

Snapshot creation captures file-system state, but it does not coordinate transactions across databases, several file systems, or remote services. A database-aware hook can flush or pause writes, create the snapshot, and release the application. The snapshot should then be mounted, checked, and copied to independent backup storage before the temporary recovery point is removed.

A snapshot can supply individual files by mounting it and copying selected content to the origin. RHEL 8 can also restore the origin name by unmounting and destroying the original, then creating a new snapshot of the retained snapshot under the original name. That destructive sequence requires a backup and a maintenance window.

A snapshot does not replace an independent backup. Origin and snapshot normally occupy the same pool, so pool loss can destroy both. A backup must place required data on separate failure domains according to the recovery policy.

The administrator must unmount a snapshot before destroying it:

```shell
umount /backup
stratis filesystem destroy pool1 snap1
```

Destroying a Stratis file system or snapshot permanently removes its accessible state. The operation requires confirmation of the pool name, file-system name, mount status, and backup status.