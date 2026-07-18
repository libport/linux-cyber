# Monitoring and Altering Kernel Behavior
> [!NOTE]
> Explains how RHEL administrators can safely inspect, modify, and persist kernel parameters and modules to tune performance, control functionality, and strengthen system security.
## Kernel parameters and modules
Red Hat administrators use kernel parameters and modules to control Linux behaviour, tune performance and reduce unnecessary functionality. Kernel parameters fall into two practical groups. Command-line parameters apply at boot. Runtime parameters apply after boot and usually sit under `/proc/sys`.
## Viewing kernel parameters
Administrators inspect boot-time parameters with:

```bash
cat /proc/cmdline
```

The output shows the kernel image followed by command-line options. The same information may appear in the kernel ring buffer:

```bash
dmesg | grep "Command line"
```

`/proc/cmdline` gives the more reliable view because the ring buffer can roll over or be cleared.

`sysctl` displays runtime parameters. Administrators list all available values with:

```bash
sudo sysctl -a
```

Large output usually needs filtering, paging or redirection to a file. A targeted search keeps the result usable:

```bash
sudo sysctl -a | grep ipv4.ip
```

When the exact parameter name is known, `sysctl` can query it directly:

```bash
sudo sysctl net.ipv4.ip_forward
sudo sysctl -n net.ipv4.ip_forward
```

The `-n` option prints only the value, which suits scripts and automated checks. Administrators can also read the matching file under `/proc/sys` by replacing dots in the parameter name with slashes:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

For boolean parameters such as `net.ipv4.ip_forward`, `0` means disabled and `1` means enabled. Other values may act as counters or integers, so administrators must confirm each parameter's meaning before changing it.
## Modifying runtime parameters
Runtime changes can be temporary or persistent. Temporary changes suit testing because they apply immediately and disappear after reboot:

```bash
sysctl -w net.ipv4.ip_forward=0
```

Persistent changes should use configuration files under `/etc/sysctl.d` for local administration. `/usr/lib/sysctl.d` stores vendor defaults and should not normally hold local overrides. A clear filename helps manage precedence because files load in lexical order:

```bash
echo "net.ipv6.conf.default.forwarding=1" > /etc/sysctl.d/10-network.conf
sysctl -p /etc/sysctl.d/10-network.conf
```

The first command records the persistent setting. The second command applies that file immediately, rather than waiting for the next boot. Administrators can reverse a change by deleting the configuration file, editing the value or removing the line. Kernel parameter changes can alter networking, storage and security behaviour, so administrators should test carefully, document intent and keep persistent EX442-style tasks reboot-safe.
## Managing kernel modules
Kernel modules add or remove kernel functionality without rebuilding the kernel. `lsmod` lists loaded modules with the module name, size and usage count:

```bash
lsmod
lsmod | sort
lsmod | grep bridge
```

`modinfo` shows details about a module, including its file path, description, version, author, licence, dependencies and signing information:

```bash
modinfo bluetooth
```

`modprobe` loads and unloads modules at runtime. These changes are temporary unless a persistent configuration loads the module at boot:

```bash
sudo modprobe bluetooth
sudo modprobe -r bluetooth
```

To load a module persistently, create a file under `/etc/modules-load.d` and place the module name in it:

```bash
echo bluetooth > /etc/modules-load.d/bluetooth.conf
```

After reboot, `lsmod | grep bluetooth` confirms whether the module loaded. A related service may also need checking with `systemctl status bluetooth`.

To prevent a module from loading, administrators should first check dependencies in `lsmod` or module metadata, then stop and disable related services. A blacklist entry under `/etc/modprobe.d` prevents automatic loading by alias. An `install` override blocks manual loading as well:

```bash
blacklist bluetooth
install bluetooth /bin/false
```

If the module appears in the initramfs image, administrators should back up the current image, rebuild it and reboot:

```bash
cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.$(date +%F-%H%M%S)
dracut -f -v
```

Careful module management can reduce exposed functionality, but dependency mistakes can stop hardware, filesystems or network services from working. Administrators should apply persistent module changes only after checking dependencies and planning a recovery path.