# File System Issues
## File system recovery
File system repair must protect data. The administrator should identify the device, confirm whether it is mounted, and avoid writing to a damaged file system unless the repair path requires it. `lsblk -f`, `blkid`, `mount`, and `/etc/fstab` help identify file systems, devices, UUIDs, labels, and mount points.

XFS and ext file systems use different repair tools. For XFS, `xfs_repair -n DEVICE` performs a no-write check. `xfs_repair DEVICE` repairs the file system. If the log is corrupt and prevents repair, `xfs_repair -L DEVICE` clears the log, which may lose recent metadata changes. Use `-L` only when necessary.

Ext file systems use `e2fsck`. `e2fsck -n DEVICE` checks without writing. `e2fsck -p DEVICE` automatically repairs safe problems. `dumpe2fs DEVICE` shows detailed metadata, including superblock locations. Alternate superblocks can rescue an ext file system when the primary superblock is damaged.

The safest pattern is:
- Identify the correct device.
- Unmount the file system.
- Run a non-writing check where possible.
- Repair with the tool that matches the file system type.
- Mount the file system and verify the expected data.
- Correct `/etc/fstab` if the boot-time mount configuration caused the failure.

Storage diagnosis should follow the stack from bottom to top. First confirm the kernel sees the disk. Next confirm partitions or block mappings. Then confirm LVM, encryption, or iSCSI layers. After that, inspect the file system. Finally check mount configuration and service paths. Repairing the top layer before proving the lower layers can hide the real fault.

The `/etc/fstab` file can stop a system during boot when it references a missing device, a wrong UUID, a wrong file system type, or invalid mount options. Rescue mode or emergency mode can repair the file. The administrator should prefer UUIDs or stable mapper paths where suitable and should test with `mount -a` before rebooting. A successful manual mount does not always prove that the boot-time fstab entry is correct because credentials, network ordering, and device activation may differ during boot.

For file system repair, the device must be correct. Running a repair tool on the wrong logical volume or raw disk can damage data. `lsblk -f` and labels reduce the risk. When LVM or encryption creates mapper devices, the repair tool should target the file system-bearing mapped device, not the encrypted raw device or the physical disk underneath.
## LVM recovery
Logical Volume Manager faults add another layer between disks and file systems. The administrator must confirm physical volumes, volume groups, logical volumes, activation state, and file system health.

Useful LVM commands include:
- `pvs`, `vgs`, and `lvs` for concise status.
- `pvdisplay`, `vgdisplay`, and `lvdisplay` for detail.
- `pvscan`, `vgscan`, and `lvscan` for discovery.
- `vgchange -ay` to activate volume groups.
- `lvchange -ay` to activate logical volumes.
- `vgcfgbackup` to back up metadata.
- `vgcfgrestore` to restore metadata.

Broken LVM configurations often involve missing devices, inactive volume groups, renamed logical volumes, bad `/etc/fstab` entries, or damaged metadata. Once the volume becomes visible under `/dev/mapper` or `/dev/VG/LV`, repair shifts back to the file system layer.
## Encrypted file systems
Linux Unified Key Setup, or LUKS, protects block devices with encryption. The `cryptsetup` command manages LUKS headers and mapped devices. It does not offer the same tab completion for subcommands as many other tools, so the administrator should know the main verbs and where to find documentation.

Key operations include:
- `cryptsetup luksFormat DEVICE` to initialise a LUKS container.
- `cryptsetup open DEVICE NAME` to create a decrypted mapping under `/dev/mapper/NAME`.
- `cryptsetup close NAME` to close a mapping.
- `cryptsetup luksDump DEVICE` to inspect header information.
- `cryptsetup luksAddKey DEVICE` to add a passphrase or key.
- `cryptsetup luksRemoveKey DEVICE` to remove a passphrase or key.

Data recovery from encrypted volumes depends on an intact LUKS header and a valid key. If the system should mount the volume at boot, `/etc/crypttab` must map the encrypted device and `/etc/fstab` must mount the resulting mapper device. A failure in either file can stop boot or leave data inaccessible.

The administrator should open the LUKS device, inspect the file system on the mapped device, mount it, verify data, and then repair boot-time configuration. Repairing the encrypted layer and repairing the file system layer are separate tasks.

LVM metadata backups can save a broken volume group after accidental changes. Before restoring metadata, the administrator should inspect available backups and confirm the correct timestamp. Restoring the wrong metadata version may remove recent logical volume changes. After metadata repair, activate the volume group and verify logical volumes before mounting any file systems.

Encrypted storage adds another critical rule: protect the LUKS header. A damaged header can make data unrecoverable even when the encrypted payload still exists. Backups of LUKS headers matter in real environments. In the exam, the candidate should avoid running format or header-changing commands unless the objective clearly requires them. `luksDump` is safe for inspection. `luksFormat` is destructive.

Boot-time encrypted volumes require the correct relationship between `/etc/crypttab` and `/etc/fstab`. Crypttab opens the encrypted device and names the mapper device. Fstab mounts the decrypted mapper device. If the mapper name changes in one file and not the other, boot-time mounting fails even though manual opening and mounting may work.
## iSCSI diagnosis
iSCSI presents remote block storage over the network. Diagnosis must cover both the target and the initiator. The target exports storage, while the initiator discovers and logs into that storage.

A basic target configuration uses targetcli. The administrator creates a backstore, creates an iSCSI qualified name, maps the backstore as a LUN, configures access control, and opens TCP port 3260. The initiator uses `iscsiadm` to discover and log in.

Useful initiator commands include:
- `systemctl enable --now iscsid` to start the initiator service.
- `iscsiadm -m discovery -t sendtargets -p TARGET_IP` to discover targets.
- `iscsiadm -m node -T IQN -p TARGET_IP -l` to log in.
- `iscsiadm -m session` to list active sessions.
- `iscsiadm -m node -T IQN -p TARGET_IP -u` to log out.

Troubleshooting should prove each layer in order. Confirm the network path with `ping`, `telnet TARGET_IP 3260`, `nc`, firewall rules, and listening sockets. Confirm the target exports the correct IQN and LUN. Confirm the initiator used the correct IQN. After login, confirm the new block device with `lsblk`. Then create or repair the file system, LVM layer, or mount configuration as needed.

iSCSI faults can look like local disk faults after login succeeds. A remote LUN may appear in `lsblk`, but the file system, LVM, or mount configuration can still be broken. Conversely, a perfect local file system repair cannot help if the initiator never logs into the target. The administrator should separate target export, network access, session login, block-device appearance, and file system use.

Persistence requires more than a successful manual login. The initiator database must contain the node record, the service must start at boot, and any dependent mounts must wait for the network and iSCSI session. Where fstab mounts iSCSI-backed storage, `_netdev` or appropriate systemd ordering prevents the system from treating the device like a local disk during early boot.

Target-side access control must match the initiator. A wrong initiator IQN in the access control list can allow discovery but prevent login, or block the initiator from seeing the expected LUN. Firewalld and SELinux can also affect target access. Start with the service state and listening port on the target, then test from the initiator.