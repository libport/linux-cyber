# Managing Storage
Red Hat Enterprise Linux 10 includes `ansible-core` 2.16 for Red Hat automation content. The `redhat.rhel_system_roles.storage` system role provides the supported, high-level interface for local file systems, LVM, mounts, swap, RAID, LUKS2, and LVM-VDO. Storage changes can destroy data, so automation must confirm device identity, current use, capacity, and recovery arrangements before changing a disk.
## Current automation interfaces
| Interface | Purpose |
| --- | --- |
| `ansible.builtin.setup` | Gathers device, mount, and LVM facts |
| `ansible.builtin.assert` | Stops work when a storage precondition fails |
| `community.general.parted` | Creates, inspects, resizes, or removes partitions |
| `community.general.lvg` | Creates or changes LVM volume groups |
| `community.general.lvol` | Creates or changes LVM logical volumes |
| `community.general.filesystem` | Creates or grows file systems |
| `ansible.posix.mount` | Controls active mounts and `/etc/fstab` entries |
| `redhat.rhel_system_roles.storage` | Manages an integrated RHEL storage configuration |

The old short names `parted`, `lvg`, `lvol`, `filesystem`, and `mount` conceal their collection dependencies. Fully qualified collection names make those dependencies explicit. The `community.general` and `ansible.posix` modules do not ship with `ansible-core`, so the control node needs collection releases compatible with its Ansible version. A RHEL 10 controller should not install a newer collection that has dropped support for `ansible-core` 2.16. The RHEL system role comes from the `rhel-system-roles` package and supplies a stable interface across supported RHEL releases.
## Discovering and validating storage
Normal fact gathering records disks in `ansible_facts['devices']`, alternative device links in `ansible_facts['device_links']`, and active mounts in `ansible_facts['mounts']`. The `devices` and `mounts` gather subsets can limit collection. The `setup` module's `filter` option accepts shell-style patterns but filters only first-level fact keys.

Kernel names such as `sdb`, `vdb`, and `nvme0n1` depend on hardware and virtualisation. Inventory should declare each intended device, preferably through a stable `/dev/disk/by-id/` path when the automation accepts it. A playbook should not guess that the second enumerated disk is unused.

Membership or definition tests handle absent devices without generating ignored failures:

```yaml
- name: Confirm that the declared disk exists
  ansible.builtin.assert:
    that:
      - storage_disk_name in ansible_facts['devices']
    fail_msg: "The declared storage disk is absent"
```

An assertion should also reject a disk that hosts the root file system, an active mount, swap, an LVM physical volume, RAID, encryption, or required data. Empty partition facts alone do not prove that a device is unused because whole-disk signatures can remain. Blindly overwriting the first megabytes with `dd` is not a safe reset procedure.

Facts represent a snapshot. A play that creates a volume group must gather the relevant facts again before reading its new size. Capacity values returned as strings should use the `float` filter when a fractional value affects a decision. Converting 5.9 GiB to an integer produces 5 and can select the wrong branch. A requested 6 GiB logical volume also requires more than a nominal 5 GiB threshold, so the condition must compare available extents with the requested allocation and overhead.
## Managing partitions
`community.general.parted` operates through the `parted` utility on the managed host. Its key settings have precise effects:

| Setting | Behaviour |
| --- | --- |
| `device` | Identifies the disk to change |
| `label: gpt` | Selects GPT and can replace an existing partition table |
| `name` | Supplies the partition name required by the module for GPT |
| `number` | Identifies the partition |
| `part_start` | Sets an offset from the start of the disk |
| `part_end` | Sets the ending offset, not the partition length |
| `flags` | Records attributes such as `lvm` |
| `state` | Uses `info`, `present`, or `absent` |

A first partition commonly starts at `1MiB` for alignment. A second 2 GiB partition after a first 2 GiB partition therefore ends near `4GiB`, rather than at `2GiB`. Omitting the boundaries defaults to the whole available span. Changing the disk label can erase prior partition information, and `state: absent` removes a partition. The `lvm` flag records intended use, but LVM can initialise either a whole block device or a partition.

Current releases default `state` to `info`, so a creation task must state `present`. A disk label, a GPT partition name, and a file system label are separate properties. The storage role sets a file system label with `fs_label`, while the lower-level file system module passes format-specific label options through `opts` where required.

The RHEL storage role manages whole disks and logical volumes directly. It does not create a file system on a partition. An explicit partition layout still requires a partitioning module or another approved provisioning layer.
## Building LVM and file systems
LVM combines physical volumes into a volume group and allocates logical volumes from that pool. `community.general.lvg` uses `pvs` for the backing devices, `vg` for the group name, and `pesize` for the physical extent size. The extent size defaults to 4 MiB, must meet LVM alignment rules, and cannot be changed on an existing volume group through the module.

`community.general.lvol` uses `vg`, `lv`, and `size` to define a logical volume. An absolute target such as `6G` supports repeatable convergence. Relative values and percentages of free space, including `+100%FREE`, are not idempotent. `resizefs: true` can grow a supported file system with its logical volume. Shrinking needs explicit protection and workload-specific preparation. XFS can grow but cannot shrink.

Storage layers have a strict dependency order. A normal creation path prepares a disk or partition, initialises the physical volume, creates the volume group, allocates the logical volume, creates the file system, and mounts it. Each task should expose a stable desired size instead of recalculating from whatever free capacity remains during that run.

The RHEL storage role can express the pool, logical volume, XFS file system, and persistent mount as one desired state:

```yaml
- name: Create and mount an XFS logical volume
  ansible.builtin.include_role:
    name: redhat.rhel_system_roles.storage
  vars:
    storage_pools:
      - name: vgfiles
        disks:
          - /dev/disk/by-id/<dedicated-disk>
        volumes:
          - name: lvfiles
            size: "6 GiB"
            fs_type: xfs
            mount_point: /files
```

`storage_safe_mode` defaults to protection against automatic removal or formatting. Disabling it belongs only in an authorised workflow that has already verified the target and accepted the destructive change.

For lower-level control, `community.general.filesystem` creates XFS, ext4, swap, and other supported formats. `force: true` can overwrite an existing file system, while `state: absent` wipes detected signatures. XFS is the default general-purpose local file system on RHEL 10. Its growth operation requires the file system to be mounted.

`ansible.posix.mount` with `state: mounted` creates the mount point, writes the `/etc/fstab` entry, and mounts the file system. `state: present` changes only `/etc/fstab`, while `state: ephemeral` mounts without changing that file. A UUID, label, or stable device link avoids dependence on a transient kernel name. `findmnt` verifies both the source and mount options. The declared mount path must remain consistent, such as `/files` throughout the play.
## Configuring swap and VDO
Swap provides disk-backed virtual memory under pressure, but it does not replace adequate RAM. Its size depends on workload, memory, crash-dump requirements, and hibernation policy. The storage role creates and activates a swap volume with `fs_type: swap`, avoiding an unconditional `command: swapon` task that reports a change on every run. Changes to active swap can require `swapoff`, sufficient free memory, and a maintenance window.

RHEL 10 implements deduplication and compression through LVM-VDO. The former standalone VDO description and `vdo` module no longer represent the preferred design. The storage role can create an LVM-VDO pool and volume with declared physical capacity, virtual capacity, compression, deduplication, file system, and mount point.
## Verification and repeatability
Syntax checking confirms YAML and module syntax but cannot detect a wrong yet valid disk path. A safe run resolves the device, checks existing signatures and consumers, confirms backups, applies the smallest required change, refreshes facts, and verifies the result with `lsblk --fs`, `pvs`, `vgs`, `lvs`, `findmnt`, and `/proc/swaps`. A second run in a test environment should report no unintended changes. Removal workflows must reverse dependencies from mount and file system through logical volume, volume group, physical volume, and partition.