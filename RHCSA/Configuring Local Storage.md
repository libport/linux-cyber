# Configuring Local Storage
## Linux Block Storage Fundamentals
Linux represents storage devices as files under the /dev directory. This design allows standard file operations to interact with hardware through kernel modules.
### Key Concepts
- Physical devices include SATA, SCSI, USB, SSD, and iSCSI storage.
- Device drivers are kernel modules that enable hardware access.
- Device files such as /dev/sda provide the interface used by the system.
- Major numbers identify the driver.
- Minor numbers identify specific devices or partitions.

The lsmod command lists loaded kernel modules. The modinfo command shows driver details. The lsblk command displays block devices, their sizes, and mount points.
## Working with Block Devices
The lsblk utility provides a tree view of disks, partitions, and logical volumes. Administrators use it to understand device relationships and verify mounts.

Typical workflow:
- Identify drivers with lsmod.
- Inspect driver metadata with modinfo.
- View storage layout with lsblk.

This process links hardware, drivers, and device files into a clear operational picture.
## Loop Devices
Loop devices allow regular files to behave like block devices. They are useful for labs and environments without extra physical disks.
### Common Uses
- Mounting ISO images
- Simulating additional disks
- Testing partitioning and filesystems

A loop device is created with losetup. The -f option selects the next available loop device.

Example workflow:

1. Download an ISO file.
2. Associate it with a loop device using losetup.
3. Mount the loop device to access contents.
4. Unmount when finished.
5. Detach the loop device if required.

Loop devices use major number 7, which identifies the loop driver.
## Mounting ISO Files
Mounting an ISO through a loop device provides read-only access to its contents without physical media.

Steps:
- Verify existing block devices with lsblk.
- Create a loop mapping with losetup.
- Mount the device to /mnt.
- Confirm contents with ls.
- Unmount when finished.

Unmounting removes the filesystem mount but does not detach the loop device. Detachment requires losetup -d.
## Creating Additional Disk Capacity
When no spare disks exist, administrators can create disk image files and attach them as loop devices.
### Creating Disk Files
Two common tools are used:
- dd creates files by writing zeros. It is flexible but slower.
- fallocate allocates space quickly and is usually preferred.

Timing tests show fallocate is significantly faster for large files.
### Attaching Disk Files
Use losetup to map disk files to loop devices. Either:
- Use -f for the next free device, or
- Specify a device name for consistency.

Detachment options:
- losetup -d removes one mapping.
- losetup -D removes all mappings.
## Disk Partitioning Concepts
Partitioning divides a disk into logical sections. While optional, it improves organisation and fault isolation.
### Reasons to Partition
- Separate boot files
- Protect the root filesystem
- Isolate user data
- Manage variable data such as /var
### Partition Tables
Two main partition table types exist.
#### MBR
- Maximum disk size of 2 TB
- Up to four primary partitions
- Supports extended and logical partitions
- Historically common
#### GPT
- Supports extremely large disks
- Up to 255 partitions
- Part of the UEFI ecosystem
- Preferred for modern systems

Driver limits may still restrict the number of usable partitions.
## Partitioning Tools
Linux provides several utilities.
### fdisk
fdisk is interactive and menu driven. It is easy for manual use but harder to automate. Modern versions support both MBR and GPT.

Typical fdisk workflow:
- Launch fdisk on the target device.
- Create a new partition table if required.
- Add partitions with the n command.
- Write changes with w.

After writing, the kernel may still use the old table. Administrators often run partprobe or kpartx to refresh partition information.
### parted
parted operates directly from the command line and supports scripting. It is suited to automation tools such as Ansible.

Common parted tasks:
- mklabel creates a partition table.
- mkpart creates partitions.
- print displays current layout.

Because parted is non interactive, it scales better in automated environments.
## Filesystems and Device Naming
After partitioning, a filesystem must be created before storing data. The mkfs command performs this task.

Example filesystem:
- XFS is commonly used in RHEL 8.
- Labels can be assigned during creation.
### The Device Name Problem
Device names such as /dev/sda can change between boots, especially when USB devices are present. This can break mounts if configurations rely on device names.
### Persistent Identifiers
Two better options exist.
#### Filesystem Labels
- Human readable
- Not guaranteed unique
- Optional
#### UUID
- Universally unique identifier
- Automatically generated
- Preferred for reliability

The blkid command displays UUID information.
## Mounting Filesystems
Filesystems can be mounted using:
- Device names
- Labels
- UUIDs

Mounting by UUID is recommended for stability.
### Temporary Mounts
The /mnt directory is typically used for temporary mounts. For persistent storage, create dedicated directories such as /data.
## Persistent Mount Configuration
To mount filesystems automatically at boot, entries are added to /etc/fstab.

Each entry defines:
- Source device or UUID
- Mount point
- Filesystem type
- Mount options
- Dump setting
- Filesystem check order

After editing fstab, the mount -a command validates the configuration without rebooting.

Loop based filesystems usually should not be added permanently unless the loop device itself is recreated during boot.
## Persisting Loop Devices
Loop devices created manually disappear after reboot. To automate their creation, administrators use systemd service units.
### Manual Recovery Process
After reboot:

1. The disk file still exists.
2. The loop device mapping is missing.
3. Recreate the mapping with losetup.
4. Run partprobe to reload partitions.
5. Mount filesystems if required.

This demonstrates why UUID based mounts are important. Even if the loop number changes, the filesystem can still be mounted correctly.
## Automating with systemd
A custom systemd service can recreate loop devices during boot.
### Unit File Structure
Key sections include:
#### Unit
- Description identifies the service.
- DefaultDependencies is often disabled.
- Ordering ensures execution before local filesystems mount.
- systemd-udevd.service is typically required.
#### Service
- Type is usually oneshot.
- ExecStart runs losetup.
- Additional ExecStart lines can run partprobe.
- The service exits after configuration.
#### Install
- WantedBy=local-fs.target ensures the service runs during boot.

After creating the unit:
- Reload systemd.
- Enable the service.
- Test with reboot.
## Logical Volume Management Preview
The course next moves into LVM2, which provides flexible storage management. LVM introduces abstraction layers that allow administrators to resize storage dynamically and manage space more efficiently than traditional partitioning.

Core LVM components include:
- Physical volumes
- Volume groups
- Logical volumes

These layers enable online resizing and improved storage agility.
## Key Takeaways
- Linux treats storage devices as files accessed through kernel drivers.
- lsblk, lsmod, and modinfo are essential diagnostic tools.
- Loop devices provide flexible virtual disks for testing.
- fallocate is faster than dd for creating disk files.
- GPT is the modern partition table standard.
- parted is better suited to automation than fdisk.
- UUID based mounts provide reliable persistence.
- /etc/fstab controls automatic mounting.
- Loop devices require systemd automation to persist across reboots.
- LVM2 enables dynamic and scalable storage management.
## Conclusion
Effective storage administration in RHEL 8 requires understanding the relationship between hardware, kernel drivers, partitions, filesystems, and mount configuration. By combining loop devices, modern partitioning tools, persistent identifiers, and systemd automation, administrators can build reliable and flexible storage environments that scale with operational needs.