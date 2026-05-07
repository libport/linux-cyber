# System Startup Issues
## Diagnosing and Troubleshooting System Startup Issues
Red Hat Enterprise Linux startup troubleshooting requires access below the normal login layer. Cloud playground servers usually do not provide that access because the administrator must interrupt boot, edit GRUB, use rescue media, or choose an alternate boot entry. A local virtual machine, a hypervisor-backed lab server, or bare metal system suits these tasks. For EX342 practice, the environment should match Red Hat Enterprise Linux 8.4 where possible.

Startup troubleshooting focuses on five areas:
- Service failures that affect boot
- Loss of root access
- Bootloader and early boot failures
- Kernel modules and module parameters
- Hardware faults and hardware evidence

Administrators should break into the system as late in the boot process as the fault allows. A late break-in preserves more system context and often makes diagnosis faster.
## Boot Process
A server starts with firmware. The firmware performs a power-on self-test, detects hardware, and selects a boot device. The system then loads GRUB 2, which loads the Linux kernel. The kernel uses the initial RAM file system, or initramfs, to prepare early storage and mount the root file system. After that, systemd starts as PID 1 and brings the system to the configured target by starting units such as services, sockets, mounts, and targets.

BIOS and UEFI systems differ mainly in how they find boot code and where GRUB stores its configuration. A BIOS system uses boot code associated with the master boot record and normally stores the GRUB configuration at `/boot/grub2/grub.cfg`. A UEFI system uses the EFI System Partition and normally stores the Red Hat GRUB configuration at `/boot/efi/EFI/redhat/grub.cfg`.

The administrator can check the boot layout by inspecting `/boot`, `/boot/grub2`, and `/boot/efi/EFI/redhat`. A populated EFI path indicates a UEFI system. A populated `/boot/grub2/grub.cfg` path indicates the BIOS GRUB configuration location.
## Recovering from Bootloader Failure
When GRUB cannot load its configuration, the system may stop at a GRUB prompt. Direct repair from that prompt is possible but difficult because the environment is limited. Red Hat rescue mode usually provides a safer repair path.

The administrator boots from the Red Hat installation ISO, selects troubleshooting, and chooses rescue mode rather than installation. Rescue mode searches for the installed system and mounts it under the path shown on screen, commonly `/mnt/sysroot` on RHEL 8. If rescue mode cannot find the system automatically and the installation uses Logical Volume Manager, the administrator should activate volumes with `vgchange -ay`, then mount the required file systems manually.

After mounting the installed system, the administrator changes root into it:

```bash
chroot /mnt/sysroot
```

The exact mount path must match the rescue prompt. Once inside the installed environment, the administrator repairs GRUB according to the platform.

For a missing BIOS GRUB configuration:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

For a missing UEFI GRUB configuration:

```bash
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

For a damaged BIOS GRUB installation, `grub2-install` reinstalls GRUB to the correct disk. For a damaged UEFI GRUB installation on RHEL, the administrator can reinstall the relevant packages, typically `grub2-efi` and `shim`, using `dnf` or `yum`.

After repair, the administrator exits the chroot, unmounts rescue media if needed, and reboots into the normal kernel.
## Using Rescue Kernel Entries
If GRUB loads but the normal kernel does not, the administrator can select the rescue kernel entry from the GRUB menu. This entry starts a smaller working environment from the installed system. It allows normal login and repair of kernel settings, GRUB configuration, or related files. After repair, the administrator reboots and selects the standard kernel.
## Regaining Root Control
If a privileged user can still log in, the administrator should change the root password from within the running system. If no privileged login remains, the administrator can use GRUB to interrupt boot.

At the GRUB menu, the administrator selects the desired kernel entry, presses `e`, finds the Linux kernel line, appends `rd.break`, and starts the edited boot entry with `Ctrl+X`. The system stops before the normal root file system is fully mounted.

The administrator remounts the system root as writable, enters it, and changes the password:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
```

SELinux labelling must be corrected after the password change. The simplest reliable method is to create the relabel marker:

```bash
touch /.autorelabel
```

The administrator then exits the chroot and reboots. The next boot may take longer while SELinux relabels the file system.
## Kernel Modules
The kernel manages hardware, memory, processes, storage, and device access. Kernel modules extend kernel functionality without rebuilding the whole kernel. Some modules are built in, while others load when required.

Administrators use several commands to inspect kernel state:
- `uname -r` prints the running kernel release
- `dmesg` displays kernel ring-buffer messages
- `cat /lib/modules/$(uname -r)/modules.builtin` lists built-in modules
- `lsmod` lists loaded modules
- `modinfo <module>` displays module metadata
- `modinfo -p <module>` lists module parameters
- `find /lib/modules/$(uname -r) -type f -name '*.ko*'` finds available module files

The `modprobe` command loads and unloads modules. `modprobe <module>` loads a module and its dependencies. `modprobe -r <module>` removes a module when no active dependency prevents removal.

Module parameters can be set at load time:

```bash
modprobe <module> <parameter>=<value>
```

A parameter change made this way only applies when the module loads. If the module is already active, the administrator may need to unload it first or change a writable runtime parameter under `/sys/module/<module>/parameters/`. Persistent module options belong in configuration files under `/etc/modprobe.d/`.
## Service Failures at Boot
Systemd controls startup through units. Service unit files define how services start, what they require, and how they relate to other units. Vendor unit files live under `/usr/lib/systemd/system`. Local overrides and administrator-created units live under `/etc/systemd/system`.

Unit suffixes identify purpose. A `.service` file defines a service. A `.target` groups units into a boot state. Directories such as `.wants` and `.requires` express relationships between units.

Dependency and ordering directives have distinct meanings:
- `Requires=` starts required units and fails the dependent unit if a required unit fails
- `Wants=` starts related units but tolerates failure
- `After=` orders a unit after another unit without creating a dependency by itself
- `Before=` orders a unit before another unit without creating a dependency by itself
- `Requisite=` requires another unit to be active already
- `Conflicts=` prevents two units from running together

The command `systemctl list-dependencies` displays dependency trees. After editing a unit file, the administrator must run `systemctl daemon-reload` so systemd reads the updated unit metadata.

When startup hangs, the debug shell can provide emergency access. Enabling `debug-shell.service` starts a root shell on `tty9`. This is useful for diagnosis but dangerous because anyone with console access can obtain root access. It must be disabled after troubleshooting.

Useful commands from the debug shell include:
- `systemctl list-jobs`
- `systemctl status <service>`
- `systemctl show <service>`
- `journalctl -xe`
## Hardware Evidence
On virtual machines, the visible hardware may be abstracted, but hardware information still helps diagnose faults and drift. The administrator can collect hardware details with these commands:
- `lscpu` for CPU information
- `lsblk` for block devices
- `lsscsi` for SCSI devices when available
- `lspci` for PCI devices
- `lsusb` for USB devices
- `dmidecode` for firmware and DMI data
- `dmidecode -t memory` for memory information

The kernel also reports hardware-related messages through `dmesg`. The `mcelog` service can record machine-check exception information and send it to the systemd journal on supported systems. After installation, the administrator enables it with `systemctl enable --now mcelog` and reviews entries with:

```bash
journalctl -u mcelog.service
```

Memory testing requires boot-level access. Installing `memtest86+`, running `memtest-setup`, and regenerating the GRUB configuration can add a memory-test entry to the GRUB menu. The administrator then reboots and selects the memory test entry.
## Startup Troubleshooting Principles
Effective startup diagnosis follows a consistent order. The administrator identifies the latest accessible stage, enters the system there, checks the relevant logs and configuration, repairs the smallest likely fault, and then reboots to confirm normal startup.

The most important checks are:
- Confirm whether the system uses BIOS or UEFI before repairing GRUB
- Use rescue media when GRUB cannot provide a usable environment
- Use the rescue kernel entry when GRUB works but the normal kernel does not
- Use `rd.break` only when early boot access is required
- Relabel SELinux after changing the root password from an early boot environment
- Check systemd unit dependencies when services block boot
- Disable the debug shell after use
- Treat hardware tools as diagnostic evidence rather than physical repair mechanisms