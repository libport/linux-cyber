# System Startup Issues
## Boot process and start-up failures
Start-up diagnosis requires a mental model of the boot chain. Firmware initialises hardware and hands control to the bootloader. GRUB2 loads the kernel and initial RAM disk. The kernel initialises devices, mounts the early root environment, and starts systemd. Systemd then brings the system to the requested target and starts units according to dependencies and ordering.

Failures may occur at each stage. A broken bootloader, missing kernel, bad initramfs, wrong root device, damaged file system, failed unit, missing dependency, or incorrect kernel argument can stop the system before normal access becomes available.

GRUB2 stores configuration and defaults in several locations. The administrator should recognise `/etc/default/grub`, `/boot/grub2/grub.cfg` on BIOS systems, and EFI paths such as `/boot/efi/EFI/redhat/grub.cfg` on UEFI systems. The `grubby` command updates boot entries and kernel arguments. After changing GRUB defaults directly, regenerate the GRUB configuration with the appropriate `grub2-mkconfig` target for the platform.

Rescue mode helps when the installed system cannot boot normally. Booting from rescue media can locate the installed system and mount it under a path such as `/mnt/sysroot` or `/mnt/sysimage`. The administrator can then use `chroot` to work inside the installed system. If rescue mode does not find the file systems automatically, LVM commands such as `vgscan`, `vgchange -ay`, and `lvscan` help activate volumes before mounting.

The `rd.break` kernel argument breaks into the initramfs before the system mounts the root file system normally. The typical root recovery pattern is:
- Edit the GRUB entry and append `rd.break` to the Linux line.
- Boot with `Ctrl+X`.
- Remount the sysroot read-write with `mount -o remount,rw /sysroot`.
- Enter the installed system with `chroot /sysroot`.
- Change the root password with `passwd` if that is the objective.
- Relabel SELinux contexts with `touch /.autorelabel` before reboot.
- Exit the chroot and reboot.

A more targeted SELinux recovery can load policy and relabel affected files, but `/.autorelabel` is simple and reliable where a full relabel is acceptable.

Boot repair depends on where the system stops. If the GRUB menu appears, the bootloader can read its configuration and the candidate can edit kernel arguments. If the system reaches emergency mode, systemd and the initramfs have progressed far enough to mount or partially mount the root environment. If the system starts but a service fails, the problem belongs to units, dependencies, configuration, or a later subsystem.

Initramfs problems often appear after storage, LVM, encryption, or driver changes. Rebuilding the initramfs with `dracut -f` can include the required modules and configuration for the current kernel. The administrator should confirm the running kernel and target kernel before rebuilding. A rescue environment may need mounted boot partitions and a chroot before the repair affects the installed system.

Root password recovery illustrates several boot concepts at once. The system boots into an early environment, the root file system initially appears read-only, and SELinux labelling must remain consistent after password files change. The password change itself is simple. The important steps are making the real root writable, working inside the correct root, and relabelling before the next enforcing boot.

A boot entry repair should account for BIOS and UEFI differences. BIOS systems usually rely on `/boot/grub2/grub.cfg`. UEFI systems use files under `/boot/efi`. Package restoration may require `grub2-pc` style packages on BIOS systems or `grub2-efi` and `shim` on UEFI systems. The candidate should identify the platform before reinstalling bootloader components.
## Kernel modules and hardware detection
Kernel modules extend the kernel with drivers and features. The administrator can inspect, load, unload, and configure modules when hardware or storage stacks fail.

Useful module commands include:
- `lsmod` to list loaded modules.
- `modinfo MODULE` to show module metadata.
- `modinfo -p MODULE` to show supported parameters.
- `modprobe MODULE` to load a module and dependencies.
- `modprobe -r MODULE` to remove a module where safe.
- `/etc/modprobe.d/*.conf` to persist options or blacklist modules.

Parameters may also appear under `/sys/module/MODULE/parameters`. Runtime changes made through sysfs do not necessarily persist. Persistent module parameters belong in a modprobe configuration file or in kernel arguments where appropriate.

Hardware diagnosis starts by asking whether the kernel sees the device. `lspci` lists PCI devices. `lsusb` lists USB devices. `lsblk` lists block devices and their hierarchy. `blkid` reports file system and UUID information. `dmesg` and `journalctl -k` show kernel detection messages and driver errors. Storage health may require `smartctl` from the smartmontools package.

Hardware faults can be physical or logical. A disk may fail, a virtual disk may not be attached, a network adapter may use the wrong driver, or a kernel module may be missing. The administrator should confirm detection before repairing higher layers such as file systems, LVM, iSCSI, or services.
## Service failures and systemd units
Systemd units define services, sockets, targets, mounts, and timers. A service failure may come from its own configuration, a missing dependency, a broken ordering relationship, an incorrect user, a blocked port, or a required file path.

`systemctl status UNIT` shows the active state, recent logs, and the exit code. `journalctl -u UNIT` gives more history. `systemctl list-units --failed` lists failed units. `systemctl list-dependencies UNIT` shows related units. `systemctl cat UNIT` displays the unit file and drop-ins. `systemctl edit UNIT` creates an override safely.

Unit relationships have different meanings. `Requires` starts another unit and stops the dependent unit if the requirement fails. `Wants` starts another unit but tolerates failure. `After` and `Before` control ordering rather than dependency. `Conflicts` stops an incompatible unit. Misunderstanding these relationships can create services that start too early, never start, or stop each other.

After changing unit files, run `systemctl daemon-reload`. Then restart the affected service and check the journal. If the problem appears only at boot, reboot or isolate the relevant target to confirm persistence.

Systemd service repair benefits from a fixed evidence path. Read the unit state, read the unit file, read the journal, inspect the referenced configuration, test the binary or daemon manually where safe, and restart the unit. A failed `ExecStart` path points to a missing binary or bad path. A permissions error points to Unix permissions, SELinux, or an incorrect service user. A port binding error points to an existing listener, a wrong address, or insufficient privilege.

Drop-in overrides are safer than editing vendor unit files. They also make local changes visible through `systemctl cat`. When a service requires an environment variable, alternate command, or changed dependency, a drop-in keeps the packaged unit intact. If a packaged unit file has been damaged directly, reinstalling the package or comparing against package content may be faster than hand repair.

Service ordering should match the resource being used. A service that needs the network should order itself after the relevant network target, but ordering alone does not guarantee that a remote database or application endpoint is ready. A mount-dependent service needs a mount unit or a proper fstab entry. A socket-activated service may not start until a connection arrives, so its inactive state may be normal.