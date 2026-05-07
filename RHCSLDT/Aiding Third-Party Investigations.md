# Aiding Third-Party Investigations
## Kernel crash dumps and SystemTap modules
Kernel crash dumps help investigators analyse a failed Linux kernel after a panic. On RHEL 8, kdump reserves memory for a crash kernel, starts a second kernel after the failure and saves a vmcore for later analysis.

Enable the service after configuring crashkernel memory and rebooting if required:

```bash
sudo systemctl enable --now kdump.service
sudo systemctl status kdump.service
```

Use `/etc/kdump.conf` to set the dump target and path. Without a separate dump target, `path /var/crash` saves the dump under `/var/crash`. If a target such as a file system, NFS share or SSH destination is configured, the path becomes relative to that target. Test only on a non-production virtual machine. A forced panic uses SysRq:

```bash
echo c | sudo tee /proc/sysrq-trigger
```

After reboot, check the new timestamped directory under `/var/crash` and confirm that it contains the dump files.

SystemTap gives administrators scriptable probes for inspecting live Linux systems. Install the packages that match the running kernel, including SystemTap itself plus the required kernel debuginfo and development packages. On cloud images, the debuginfo repository name varies, so enable the matching BaseOS debuginfo repository before installing.

The `stap` command runs SystemTap scripts from a file, standard input or the command line. It translates the script into C, compiles it into a kernel module, loads that module and runs the probe. Common options include:
- `-v` for verbose pass-by-pass output
- `-p NUM` to stop after a specified pass
- `-m NAME` to set the generated module name

Tapsets in `/usr/share/systemtap/tapset/` provide probe aliases and helper functions for other scripts. They are libraries, not standalone session scripts, so many tapset files will not run directly with `stap`.

A reusable module can be built from a script without running it:

```bash
sudo stap -v -p4 -m ping ping.stp
sudo mkdir -p /lib/modules/$(uname -r)/systemtap
sudo mv ping.ko /lib/modules/$(uname -r)/systemtap/
sudo staprun ping
```

Store modules in `/lib/modules/$(uname -r)/systemtap/` and keep that directory owned and writable only by root. Add trusted users to `stapusr` when they only need `staprun` access to approved modules. Add users to `stapdev` only when they can be trusted with effective root access, because `stap` compiles and loads kernel code.