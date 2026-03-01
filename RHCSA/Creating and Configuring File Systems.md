# Creating and Configuring File Systems
## Managing Local File Systems
Effective storage management in RHEL 8 begins with understanding disks, partitions, and mount points. Administrators typically inspect block devices using lsblk. The command reveals device structure, mount locations, and file system types. Using the -f option provides additional file system details such as UUIDs and formats.

Mount points are directories within the root file system that provide access to other file systems. Administrators create these directories, format storage with mkfs, and mount the file systems either manually or through persistent configuration in /etc/fstab. UUID based mounting is preferred because device names may change across boots.
## Working with XFS File Systems
XFS is the default file system in RHEL 8 and is optimised for scalability and performance. Administrators inspect XFS metadata using xfs_info. Important fields include:
- isize, which defines inode size
- bsize, which defines data block size
- allocation and geometry details that influence performance

On common x86_64 systems, the maximum block size is typically 4 KB. Larger block sizes may be available on enterprise hardware platforms.

The xfs_admin tool manages file system labels and identifiers. Administrators can:
- Display the label with xfs_admin -l
- Assign or change the label with xfs_admin -L
- Display or regenerate the UUID

Labelling file systems improves recovery and documentation because the label describes the file system purpose. Changes to labels require the file system to be unmounted.
## Creating and Inspecting EXT4 File Systems
EXT4 remains widely used across Linux distributions. Administrators create EXT4 file systems using either mkfs -t ext4 or the wrapper command mkfs.ext4.

Metadata inspection uses dumpe2fs, which produces detailed output about the superblock and file system configuration. Because the output is extensive, administrators often pipe it to less or filter it with grep.

Important EXT4 maintenance parameters include:
- Maximum mount count
- Check interval

These values determine when automatic file system checks occur. Unplanned checks can significantly delay boot times on large systems. Administrators often disable automatic checks and instead schedule maintenance windows.

Labels can be assigned or modified with tune2fs -L. Metadata can be reviewed with tune2fs -l, which provides a concise summary compared to dumpe2fs.
## Securing Mount Points
A mount point exists as a normal directory until a file system is mounted on it. Proper security requires two distinct stages:
- Secure the mount point directory itself
- Secure the mounted file system after mounting

The recommended approach is:
- Create the mount point directory
- Restrict access to root only using mode 0700
- Mount the file system
- Apply the required permissions to the mounted file system

This method prevents users from writing files to the underlying directory when the file system is unmounted. Without this precaution, users may create files that later disappear when the file system is mounted.

Persistent mounts are configured in /etc/fstab. Best practice includes:
- Using UUID identifiers
- Specifying the correct file system type
- Setting dump and fsck fields appropriately
## Extending Logical Volumes and File Systems
RHEL 8 commonly uses Logical Volume Manager. LVM allows administrators to expand storage without downtime.

The workflow includes:
- Inspect volume groups with vgs
- Add new physical volumes with vgextend
- Expand logical volumes with lvextend
- Resize the file system using the -R option

When the underlying file system is XFS or EXT4, resizing can occur online while the system remains in use. This capability allows administrators to respond quickly to storage pressure without service interruption.

Careful planning of volume groups and free extents ensures future scalability. Administrators should monitor available space regularly to avoid emergency expansions.
## Managing Permissions for Collaboration
Collaborative environments require careful control of ownership and deletion rights. Linux permissions include three standard classes and a leading special permission digit.

The four digit octal mode represents:
- Special permissions
- User permissions
- Group permissions
- Other permissions

The special permission digit enables advanced behaviour such as setuid, setgid, and the sticky bit.
## Special Permissions Overview
The three special permission values are:
- 4 for setuid
- 2 for setgid
- 1 for the sticky bit

These values can be combined. For example, a value of 3 sets both setgid and the sticky bit.

Setuid primarily applies to executables. Setgid and the sticky bit are commonly used on directories to support collaboration.

Administrators view permissions with ls -l and can locate files with special permissions using the find command with the -perm option.
## Using the Sticky Bit
The sticky bit controls deletion in shared directories. Normally, any user with write access to a directory can delete files within it. When the sticky bit is set, users can delete only their own files.

This behaviour is essential for shared writable locations. The sticky bit is applied with chmod o+t or by setting the leading octal value.

Typical implementation steps include:
- Assign correct group ownership
- Grant write access to the group
- Apply the sticky bit

After configuration, users can create files but cannot remove files owned by others. The ls output displays a lowercase t when execute permission for others is present, or uppercase T when it is not.
## Using the SGID Bit on Directories
The setgid bit ensures consistent group ownership within collaborative directories. Without SGID, new files inherit the creator’s primary group. This behaviour can block access for other collaborators.

When SGID is set on a directory:
- New files inherit the directory’s group ownership
- Group collaboration becomes predictable
- Permission management becomes simpler

Administrators apply SGID using chmod g+s or by setting the appropriate octal value. Verification is performed with ls -l.

SGID works best alongside an appropriate umask. For example, a umask of 007 removes permissions for others while preserving group access.
## Creating and Managing Groups
Groups are essential for controlled collaboration. Administrators create groups with groupadd and manage membership with gpasswd.

After adding a user to a group, the user must log out and back in to activate the new membership. The id command verifies effective group membership.

Proper group design reduces the need for complex permission changes and supports scalable access control.
## Combined Collaborative Configuration
A robust shared directory typically includes:
- Correct group ownership
- SGID enabled
- Sticky bit enabled when deletion control is required
- Appropriate umask for users

This combination ensures:
- Predictable ownership
- Controlled deletion
- Secure multi user collaboration

Testing with multiple users confirms that permissions behave as intended.
## Introduction to NFS in RHEL 8
Network File System allows file systems to be shared across hosts. RHEL 8 focuses on NFS version 4.2 and introduces the nfsconf management tool.

Key characteristics of NFSv4 in RHEL 8 include:
- Centralised configuration in /etc/nfs.conf
- Simplified firewall requirements
- Primary use of TCP port 2049

A typical lab environment uses separate server and client systems, often provisioned through Vagrant. Private networking enables communication between the hosts.

Administrators must also consider SELinux policies when configuring NFS exports to ensure correct access behaviour.
## Key Administrative Practices
Effective file system management in RHEL 8 depends on disciplined operational practices:
- Use UUIDs for persistent mounts
- Label file systems clearly
- Secure mount points before use
- Disable or schedule disruptive fsck checks
- Monitor LVM free space
- Apply SGID and sticky bit appropriately
- Validate group membership after changes
- Test shared directory behaviour with multiple users

These practices improve reliability, simplify troubleshooting, and reduce the risk of data loss or permission errors.
## Conclusion
RHEL 8 provides a mature and flexible storage stack that supports both standalone and collaborative workloads. Administrators who understand XFS and EXT4 behaviour, mount point security, LVM expansion, and special permission bits can manage storage confidently in production environments.

Mastery of these tools enables safe scaling of storage, predictable multi user collaboration, and efficient system recovery. Continued practice at the command line remains essential for developing operational fluency.