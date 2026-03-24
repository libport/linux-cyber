# Managing Storage
Ansible manages storage in two stages. It first discovers the devices, partitions, volume groups, logical volumes and mount points that already exist. It then changes only the required parts of that state. Reliable storage playbooks therefore start with fact gathering, use conditionals to select the correct device name for each host and apply idempotent tasks that can run more than once without breaking the system.

The main storage modules are `parted` for partitions, `lvg` for volume groups, `lvol` for logical volumes, `filesystem` for file systems and `mount` for active mounts and `/etc/fstab` entries. Device names vary across platforms, so safe playbooks test for presence before they act. Common names include `/dev/sdb` on many virtual machines, `/dev/vdb` on KVM guests and `/dev/nvme0n1` on NVMe devices.
## Discovering storage facts
Ansible collects storage facts during normal fact gathering. Three facts matter most:
- `ansible_devices` lists block devices and partitions
- `ansible_device_links` shows alternate device references such as IDs and UUIDs
- `ansible_mounts` lists active mounts and their source devices

The setup filter must match the fact name exactly.

```bash
ansible ansible1 -m setup -a 'filter=ansible_devices'
```

That output exposes the device model, size, partition table details, UUIDs and any LVM holders. Storage logic should use those facts instead of assuming a fixed device name. Device facts also reveal whether a device is rotational, removable, virtual or already claimed by a device-mapper target, which helps the playbook avoid destructive changes on the wrong block device.
## Choosing control flow
A storage playbook usually needs one of two control paths. It can fail loudly when a required device is missing, or it can skip that host and continue elsewhere. `assert` suits hard requirements because it prints a specific failure message. `when` suits conditional execution when absence is acceptable. For mixed fleets, the cleanest pattern is to report the issue and end that host before the play touches partitions, file systems or LVM metadata.
## Selecting the correct device safely
Storage playbooks fail when they test undefined keys directly. A safe conditional checks membership first. When a required device is absent, the host should stop cleanly before any partitioning task runs.

```yaml
- name: report missing device
  debug:
    msg: device sdb not present
  when: "'sdb' not in ansible_facts['devices']"

- name: stop work on hosts without sdb
  meta: end_host
  when: "'sdb' not in ansible_facts['devices']"
```

When hosts may expose different second-disk names, the playbook can detect the first matching candidate and store it in a variable.

```yaml
- name: detect second disk
  set_fact:
    disk2name: "{{ item }}"
  loop:
    - sdb
    - vdb
    - nvme0n2
  when: item in ansible_facts['devices']
```

This pattern avoids undefined-variable errors and removes the need for `ignore_errors`. It also makes later tasks easier to read because the playbook can refer to `disk2name` instead of duplicating several device tests.
## Creating partitions and LVM
Partitioning needs explicit boundaries. The `parted` module does not infer the next free region automatically, so each partition requires `part_start` and `part_end`. GPT partitions also require a `name`. The `label` argument sets the partition table type, not a file system label.

A consistent layout uses the first partition for swap and the second for LVM data.

```yaml
- name: create swap partition
  parted:
    device: /dev/sdb
    label: gpt
    number: 1
    name: swap
    state: present
    part_start: 1MiB
    part_end: 2GiB

- name: create LVM partition
  parted:
    device: /dev/sdb
    label: gpt
    number: 2
    name: data
    state: present
    part_start: 2GiB
    part_end: 100%
    flags:
      - lvm
```

The `flags` setting marks the second partition for LVM. Without that marker, the storage layout becomes harder to inspect and less consistent with standard Linux administration practice.

The LVM layer uses the partition marked for LVM, not the swap partition.

```yaml
- name: create volume group
  lvg:
    vg: vgdata
    pvs: /dev/sdb2
    pesize: 8

- name: create logical volume
  lvol:
    vg: vgdata
    lv: lvdata
    size: 6g
```

`lvg` creates or extends the volume group from the listed physical volumes. `lvol` creates the logical volume inside that group and can later resize it. Absolute sizes keep the play idempotent. Relative values such as `100%FREE` can succeed on the first run and fail on the second run because the free space has already been consumed.
## Creating file systems, mounts and swap
After the block device layer exists, the system can create a file system and mount it. The `filesystem` module writes the on-disk structure, while the `mount` module controls both the live mount state and the matching `/etc/fstab` entry.

```yaml
- name: create XFS file system
  filesystem:
    dev: /dev/vgdata/lvdata
    fstype: xfs

- name: mount file system
  mount:
    src: /dev/vgdata/lvdata
    path: /data
    fstype: xfs
    state: mounted
```

`state: mounted` ensures the file system is mounted now and recorded for future boots. `state: present` only writes the `/etc/fstab` entry. This distinction matters when the playbook prepares storage for later use but should not activate it immediately.

Swap uses a different workflow. The device first receives a swap signature. It then becomes active only when the host runs `swapon`, and it remains persistent across reboots only when `/etc/fstab` contains a swap entry.

```yaml
- name: add swap when total swap is low
  block:
    - name: create swap signature
      filesystem:
        dev: /dev/sdb1
        fstype: swap

    - name: record swap in fstab
      mount:
        src: /dev/sdb1
        path: none
        fstype: swap
        state: present

    - name: activate swap now
      command: swapon /dev/sdb1
  when: ansible_swaptotal_mb < 256
```

This approach keeps the configuration conditional and persistent. The threshold can change to suit the host role.
## Refreshing facts after storage changes
Facts gathered at the start of a play do not update automatically after Ansible creates a partition or volume group. Any later task that reads `ansible_facts['lvm']` must refresh facts first.

```yaml
- name: refresh facts
  setup:
- name: store volume group size
  set_fact:
    vgsize_g: "{{ ansible_facts['lvm']['vgs']['vgfiles']['size_g'] | float }}"
```

Volume-group size comparisons should use numeric values, not raw strings. `float` preserves decimal precision better than `int` when the logic depends on a threshold such as 5 GB.

```yaml
- name: create larger logical volume
  lvol:
    vg: vgfiles
    lv: lvfiles
    size: 6g
  when: vgsize_g > 5.0

- name: create smaller logical volume
  lvol:
    vg: vgfiles
    lv: lvfiles
    size: 3g
  when: vgsize_g <= 5.0
```

The final XFS logical volume mounts on `/files`.

```yaml
- name: format logical volume
  filesystem:
    dev: /dev/vgfiles/lvfiles
    fstype: xfs

- name: mount logical volume
  mount:
    src: /dev/vgfiles/lvfiles
    path: /files
    fstype: xfs
    state: mounted
```
## Operational rules
- Wipe reused lab disks before repartitioning them
- Reboot a host if stale kernel partition state prevents reuse
- Refresh facts after creating or changing LVM objects
- Prefer absolute LV sizes in repeatable playbooks
- Use clear failure or skip behaviour before destructive tasks run

These rules reduce false positives during testing and keep repeated runs predictable across different lab hosts.