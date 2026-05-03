# General Trouble Shooting Methods
## Exam scope and working conditions
The Red Hat Certified Specialist in Linux Diagnostics and Troubleshooting exam, EX342, tests the candidate's ability to analyse Red Hat Enterprise Linux systems, diagnose faults that degrade or stop service, correct the fault where required, and collect evidence for third-party investigation where direct repair is not the objective. The exam uses performance-based tasks rather than multiple-choice questions. Candidates work on one or more broken systems, read the stated objectives, repair or investigate each system, and ensure the configuration persists after reboot.

The current Red Hat exam page lists EX342 as based on Red Hat Enterprise Linux 8.4 and lasting four hours. Red Hat also states that an unsuccessful candidate is eligible for one exam retake. It remains a demanding exam because the starting point is often a damaged or misconfigured system, not a clean deployment. The candidate must rely on local tools, installed documentation, and the documentation supplied in the exam environment. Internet access does not form part of the exam workflow.

Candidates may sit Red Hat individual exams in a testing centre or by using the Red Hat remote exam environment where available. A testing centre reduces local hardware risk because Red Hat controls the workstation and network. The remote format requires the candidate to prepare a compatible computer, boot the Red Hat remote exam LiveUSB, use an external wired webcam, and provide a stable workspace. Red Hat advises a wired network where possible because the remote environment may not detect every wireless adapter. A laptop normally needs at least two free USB ports, while a desktop normally needs ports for the USB drive, external webcam, keyboard, and mouse.

The exam rewards disciplined practice more than memorised trivia. The candidate must know the standard utilities, the locations of key configuration files, and the patterns behind common failures. Familiarity with the Red Hat documentation, local man pages, package metadata, and troubleshooting commands matters because the exam environment removes the usual habit of searching the web.

The major skill areas are:
- General troubleshooting methods, including system information, documentation, monitoring, and centralised logging.
- System start-up faults, including service failures, root recovery, bootloader issues, hardware detection, and kernel modules.
- File system faults, including XFS and ext repair, Logical Volume Manager recovery, encrypted volumes, and iSCSI.
- Package management faults, including dependencies, repository issues, corrupted RPM data, and altered files.
- Network connectivity faults, including basic reachability, firewalld, NetworkManager, DNS, and packet capture.
- Application faults, including shared libraries, memory leaks, tracing, debugging, and SELinux.
- Authentication faults, including Pluggable Authentication Modules, SSSD, LDAP, and Kerberos client configuration.
- Investigation support, including kdump crash dumps and SystemTap modules.
## Exam-day execution
Preparation should include practice on real RHEL 8.4 systems, repeated repair of deliberately broken services, and timed work without internet search. Candidates should practise reading the objective first, identifying affected hosts, and recording the tests that prove success.

Before the exam, the candidate should run remote compatibility checks if using the remote format, verify the external webcam, prepare the required USB media, read the full Red Hat exam instructions, and avoid last-minute hardware changes. A testing centre candidate should still verify identification requirements and arrival time.

During the exam, the candidate should scan all tasks, identify quick wins, and avoid spending too long on one fault early. The environment may feel slow because the candidate works through a controlled graphical session. Slow input should not lead to rushed commands. Typos in system files, package databases, bootloader entries, PAM rules, and SELinux policy can create additional faults.

An example exam-day checklist:
- Read the objective and affected host.
- Reproduce the failure.
- Gather logs and command evidence.
- Form one likely explanation.
- Apply the smallest safe fix.
- Retest the original failure.
- Check persistence if the objective requires it.
- Move on when the task is complete.

The candidate should not rely on memory alone. Local documentation, man pages, package queries, and installed examples can resolve uncertainty faster than guesswork. The final minutes should confirm service status, file mounts, network access, authentication tests, and reboot persistence for completed tasks.

The strongest preparation method recreates failure patterns. Break a service file and repair it. Damage a test file system and recover it. Mislabel web content and diagnose SELinux. Disable a repository and resolve a package failure. Break DNS and prove the difference between name failure and address failure. Corrupt a package-owned configuration file and restore it from the package. Repetition turns scattered command knowledge into a calm sequence.

A candidate should also practise recovering from mistakes. If a command changes the wrong file, restore from backup. If a service gets worse, read the journal again and compare the last known good state. If a package reinstall overwrites a local change, restore the needed part deliberately. Real troubleshooting includes controlled rollback.
## Troubleshooting discipline
Effective troubleshooting starts before any command runs. The administrator should remain calm, read the stated problem, identify the affected service or subsystem, and reproduce the fault. Reproduction prevents blind changes and gives a reliable test after each attempted repair. A useful workflow moves through four stages: observe, hypothesise, test, and fix.

The observation stage gathers the information already supplied and adds evidence from the system. Logs, service status, command output, file timestamps, package state, and recent configuration changes all belong in this stage. The hypothesis stage explains the likely cause from that evidence. The testing stage checks the hypothesis without causing further damage where possible. The fix stage changes the system, restarts or reloads the affected service, and repeats the original test.

Failed hypotheses do not indicate failure. They narrow the search. A good troubleshooter reverts unsafe changes, gathers more evidence, and tests a better explanation. The process should end with the root cause fixed, the requested objective complete, and the system stable after reboot.
## Basic system information
Core system commands help the administrator decide whether a fault comes from load, storage, memory, limits, sockets, or processes.

The `uptime` command shows how long the system has been running, how many users have sessions, and the load averages over one, five, and fifteen minutes. High load may indicate CPU pressure, blocked I/O, too many runnable tasks, or a stuck service.

The `df -h` command displays mounted file systems and their available space in human-readable units. Full partitions can prevent logging, package installation, service start-up, and application writes. The mount point matters as much as the percentage used because a full `/var`, `/tmp`, or application data path can break a service while the root file system still appears healthy.

The `free -h` command shows physical memory and swap use. `free -m` displays values in MiB and `free -g` displays values in GiB. A system that exhausts memory may swap heavily, kill processes, or slow down enough to appear hung.

Open files can also exhaust limits. `lsof` lists open files. `lsof -u USER | wc -l` counts open files for a user. `su - USER -c 'ulimit -n'` checks the user's open file limit. `lsof -i` lists open network files and helps identify unexpected listening or established connections. If `lsof` reports stale user session paths such as `/run/user/UID/gvfs`, unmount the stale path with `umount` rather than ignoring the warning.

The `top` command gives a live view of load, tasks, CPU use, memory, swap, and running processes. It is useful early in diagnosis because it combines general system pressure with process-level detail. The administrator can then switch to more precise tools.

The `ss -lpt` command lists listening TCP sockets with associated processes. It helps confirm whether a service listens on the expected port and whether an unexpected service has claimed a port.

The `ps` command displays processes. Plain `ps` limits output to the current session. `ps aux` lists processes across users, including processes without a controlling TTY. `ps ajfx` shows process relationships in a tree. `ps aux --sort -pcpu` sorts by CPU use, and similar sort keys can rank memory or other fields. The manual page lists supported sort keys.

A reliable early triage sequence combines these commands rather than treating each one in isolation. `uptime` establishes load and time since boot. `df -h` proves whether storage pressure can explain failures. `free -h` shows whether memory pressure or swap use has distorted service behaviour. `systemctl --failed` highlights units that already declared failure. `journalctl -p warning -xb` reveals warnings and errors from the current boot. `ss -lntp` shows whether expected services are listening. `ps aux --sort -pcpu` and `ps aux --sort -pmem` identify processes that consume resources.

Resource data needs context. High CPU use is normal during compilation, encryption, or package operations, but suspicious for a quiet service. High memory use may reflect caching rather than a leak, so the administrator should compare used memory, available memory, swap activity, and process growth over time. Full disks often create misleading errors because services report permission, locking, or write failures instead of a plain disk-full message. Open file exhaustion can also mimic application failure because the process cannot create sockets, logs, temporary files, or new descriptors.

Process trees help when one service launches workers. A failing web service, database, or script runner may show a healthy parent and failing children. Sorting by CPU or memory finds the noisy process, while a tree view shows the supervisor responsible for restarting it. Socket views complete the picture because a process can run without listening on the required address or port. A service bound to `127.0.0.1` may work locally and fail remotely. A service bound to the wrong interface may pass some tests and fail others.
## Documentation and package intelligence
The exam environment demands local research skills. Commands often reveal their own syntax through `--help`. For example, `ps --help` or a more specific help class can show supported options quickly.

Manual pages provide deeper documentation. The `man -k KEYWORD` command searches the manual database, which may require `mandb` if the database is stale. Manual sections identify the type of page. Section 1 covers executable programs, section 5 covers file formats, and section 8 covers system administration commands. That distinction matters for tools such as `rsyslog`, where `rsyslog.conf` documents configuration syntax and `rsyslogd` documents the daemon. Inside a manual page, `/TERM` searches forward and `n` moves to the next match.

Package tools identify where commands and files come from. `rpm -q PACKAGE` queries an installed package. `rpm -qf /path/to/file` shows which installed package owns a file. `rpm -ql PACKAGE` lists files installed by a package. The `/usr/share/doc` tree often contains examples, templates, and package-specific notes. A command such as `find /usr/share/doc -iname '*example*' -type d` can locate example directories.

DNF and Yum search repository metadata. `dnf search httpd` finds packages that match a term. `dnf provides '*/pip3'` shows which package supplies a command path. `repoquery --requires PACKAGE` lists dependencies for a package. These tools help the administrator install the missing component rather than guessing from command names.

Red Hat documentation remains a major preparation resource. A Red Hat account can provide access to developer resources and documentation, while paid subscriptions may provide additional support tools. Systems with `redhat-support-tool` can query support knowledge from the command line. The command is useful when available, but it is not necessary for passing EX342.

A useful documentation pattern starts broad and then narrows. First, use `--help` to identify the command shape. Next, use `man -k` to find the right manual page. Then search inside the manual page for the configuration directive, option, or error term. If the question involves package ownership, use RPM queries. If the question involves an uninstalled command, use DNF provides. If the question involves examples, search `/usr/share/doc`.

Package metadata can also identify accidental file changes. When a configuration file differs from the packaged version, RPM verification reports the change but does not explain whether the change is bad. The administrator must combine the package report with the service objective. A changed file may be the cause of failure, a valid local customisation, or evidence of a previous repair attempt. The safest repair keeps a copy of the current file before restoring a package-owned version.

The documentation habit matters most under time pressure. Guessing a command option can damage the system or consume time. A thirty-second manual search often beats a five-minute trial-and-error loop. The candidate should know how to search quickly, leave the pager, and copy exact option names into commands.
## Performance Co-Pilot monitoring
Performance Co-Pilot, usually shortened to PCP, provides monitoring, collection, and reporting tools that suit exam systems because it runs locally. The basic packages are `pcp` and `pcp-system-tools`. The optional `pcp-doc` package adds manual pages.

PCP uses `pmcd`, the Performance Metrics Collector Daemon, to receive metrics from Performance Metrics Domain Agents, or PMDAs. Client tools then query `pmcd`. The service can be enabled and started with `systemctl enable --now pmcd`.

The `pmlogger` service stores historical metrics under `/var/log/pcp/pmlogger/HOSTNAME` by default. Start it with `systemctl enable --now pmlogger` when historical evidence matters. Historical logs help identify whether a fault is current, recurring, or triggered at a particular time.

PCP provides several useful commands:
- `pmstat` prints common performance statistics at regular intervals.
- `pmstat -t 1sec` changes the sampling interval.
- `pmstat -t 1sec -T 10sec` samples for a fixed duration.
- `pmstat -a ARCHIVE -S '@DATE TIME' -T '@DATE TIME'` reads historical data from an archive.
- `pcp atop` or the packaged PCP atop variant presents a live top-like view with PCP metrics.
- `pminfo` lists available metrics.
- `pminfo -T METRIC` displays a short description of a metric.
- `pmval METRIC` prints raw metric values.
- `pmrep METRIC` reports metric values in a more readable format.

PCP atop can sort and filter by major resources. CPU, memory, disk, GPU, command, user, process, and busy-resource views help the administrator identify a bottleneck without assembling every query manually.

Monitoring output should answer a specific question. A CPU question needs runnable load, CPU utilisation, and the process list. A memory question needs available memory, swap use, and per-process growth. A disk question needs device throughput, I/O wait, busy devices, and free space. PCP can collect all of these areas, but the administrator should avoid reading pages of metrics without a hypothesis.

Historical PCP data helps when a service fails before the administrator logs in. The archive can show a spike in memory, disk, or load before the fault. It can also prove that a service failed without resource pressure, which redirects attention to configuration, permissions, network, or authentication. When comparing historical windows, use times that definitely fall within the archive and account for the server's local time.

Live tools and historical tools serve different purposes. `top` and `pcp atop` help during an active fault. `pmlogger` archives help after a restart or after a transient issue. `pmval` suits a narrow metric check. `pmrep` suits a readable report. `pminfo` provides the metric names that connect the question to the data.
## Logging and centralised logs
Logging provides the historical record that turns a vague symptom into evidence. RHEL uses rsyslog and systemd-journald. Rsyslog follows the syslog model of facilities and priorities. Facilities identify the subsystem, while priorities indicate severity. Common priority levels include debug, info, notice, warning, err, crit, alert, and emerg.

The `/var/log` directory stores traditional log files. Rsyslog only writes what its configuration defines, so missing messages may reflect configuration rather than the absence of events.

The `journalctl` command reads the systemd journal. Plain `journalctl` opens the full journal in a pager. `journalctl -ef` jumps to the end and follows new entries. `journalctl -k` shows kernel messages. `journalctl -p 4` shows priority warning and above. `journalctl -xb` shows entries from the current boot. Persistent journal storage improves post-reboot diagnosis because volatile logs disappear when the system restarts.

Centralised logging lets several hosts send messages to one log server. On the receiver, `/etc/rsyslog.conf` enables TCP or UDP reception by loading the appropriate module and opening the input port, commonly TCP 514. The administrator then restarts rsyslog and opens the firewall if required. On senders, a rule such as `*.* @@LOGHOST:514` forwards logs over TCP. The single `@` form uses UDP. Configuration snippets can also live under `/etc/rsyslog.d`.

The `logger` command tests forwarding. A known message sent with `logger 'test message'` should appear on the remote log host. If it does not, check rsyslog status, syntax, firewall rules, listening sockets, name resolution, and network reachability.

Log investigation should stay close to the failing action. Restarting a service and immediately running `journalctl -u UNIT -n 50` can expose the exact error. Following the journal with `journalctl -ef` while reproducing a failure shows the new entries created by that test. Searching old logs before reproducing the issue can waste time because the system may contain stale errors from earlier exercises or earlier boots.

Rsyslog rules combine selectors and actions. A selector such as `authpriv.*` chooses messages from a facility at all priorities. A selector such as `*.crit` chooses critical messages from all facilities. An action can write to a file, forward to a host, or discard messages. Ordering matters because rsyslog processes rules in sequence. A broad rule can capture expected messages, while a discard or stop action can prevent later rules from seeing them.

Remote logging failures often come from simple mismatches. The sender may use UDP while the receiver listens only on TCP. The firewall may block port 514. The sender may point at the wrong address. The receiver may write logs to a different file than expected. A successful test proves the full chain only when the test message appears on the receiver after the sender runs `logger`.
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
## Package management faults
Package dependency failures often come from disabled repositories, incompatible versions, modular streams, partial transactions, or strict dependency resolution. DNF and Yum provide options that help diagnose these failures without forcing unsafe changes.

Useful package commands include:
- `dnf repolist` to view enabled repositories.
- `dnf repolist all` to view disabled repositories too.
- `dnf clean all` and `dnf makecache` to refresh metadata.
- `dnf install PACKAGE --nobest` to allow a non-latest candidate where policy permits.
- `dnf install PACKAGE --skip-broken` to skip packages with unsatisfied dependencies.
- `dnf downgrade PACKAGE` to return to an earlier version.
- `dnf versionlock` where the plugin is installed and version pinning matters.

An RPM database fault can break queries and installs. The RPM database lives under `/var/lib/rpm`. Repair should start with caution: stop package activity, check for locks and open RPM files, back up the database files, verify the database, dump and reload when required, and rebuild. Tools under `/usr/lib/rpm` include `rpmdb_verify`, `rpmdb_dump`, and `rpmdb_load`. The rebuild command is `rpm -vv --rebuilddb`.

RPM verification detects changed files from installed packages. `rpm -V PACKAGE` reports altered size, permissions, ownership, group, checksum, modification time, capabilities, or link targets. `rpm --setperms PACKAGE` restores packaged permissions. `rpm --setugids PACKAGE` restores packaged ownership and group. `dnf reinstall PACKAGE` can replace damaged package files when manual restoration is slower or less reliable.

Dependency repair should avoid forcing broken packages into place. A forced RPM install can satisfy neither the package manager nor the service that needs the dependency. DNF's resolver output usually names the conflict, missing dependency, excluded architecture, disabled repository, or modular stream. The administrator should read that output before adding flags.

Repository faults often explain sudden dependency failures. The configured repository may be disabled, unreachable, mismatched with the RHEL release, or missing metadata. `dnf repolist -v` can reveal base URLs and metadata expiry. Network faults, proxy settings, subscription state, and wrong release versions can all surface as package problems. A package task may therefore require DNS, routing, certificate, or repository repair before DNF succeeds.

RPM verification symbols need careful interpretation. A changed configuration file is common and not automatically wrong. A changed binary, library, PAM file, unit file, or permission bit may be more suspicious. Restoring ownership and permissions is lower risk than replacing content. Reinstalling a package is appropriate when package-owned files are missing or damaged and local customisations are not required.

A corrupted RPM database repair should always start with a backup. Even when the exam objective expects a repair, a copy of `/var/lib/rpm` preserves a rollback point. After rebuilding, basic package queries such as `rpm -qa`, `rpm -q PACKAGE`, and `dnf history` should work again. Only then should the administrator continue with installs or reinstalls.
## Network connectivity
Network diagnosis should move from reachability to service access and then to configuration. The administrator should identify the interface, address, route, DNS servers, firewall state, and listening service.

Basic verification tools include:
- `ip addr` to show addresses.
- `ip route` to show routes.
- `ping HOST` to test ICMP reachability.
- `telnet HOST PORT` to test a TCP port.
- `nc -vz HOST PORT` to test TCP more flexibly.
- `curl URL` to test HTTP services.
- `ss -lntp` to show listening TCP ports.
- `ss -antp` to show TCP connections.

DNS diagnosis uses bind-utils. `dig NAME`, `host NAME`, and related commands show whether name resolution works and which servers answer. The resolver configuration normally appears through `/etc/resolv.conf`, often managed by NetworkManager. Do not edit generated resolver files without checking NetworkManager state.

NetworkManager configuration uses `nmcli`. `nmcli device status` shows device state. `nmcli connection show` lists connection profiles. `nmcli connection modify NAME ipv4.addresses ADDRESS ipv4.gateway GATEWAY ipv4.dns DNS ipv4.method manual` updates a profile. `nmcli connection up NAME` activates it. For temporary tests, `ip addr add`, `ip route add`, and related `ip` commands may be faster, but they do not persist unless saved in the connection profile.

Firewalld commonly blocks services that work locally. `firewall-cmd --list-all` shows the active zone configuration. `firewall-cmd --add-service=http --permanent` or `firewall-cmd --add-port=80/tcp --permanent` adds persistent access. `firewall-cmd --reload` applies permanent changes. Always check the active zone and interface before assuming the rule applies.

Packet capture clarifies ambiguous network faults. `tcpdump` and `tshark` both capture packets through libpcap style filters. Common options include `-i INTERFACE` for interface, `-w FILE` to write a capture, `-c COUNT` to stop after a number of packets, and `-r FILE` to read a capture. Captures show whether packets leave, arrive, receive replies, or fail during handshake.

Network troubleshooting should distinguish local service failure from path failure. A successful `curl localhost` proves the application can answer locally. A failed remote connection may still come from firewall rules, binding to loopback only, routing, DNS, or cloud security controls. A failed `curl localhost` points back to the service, application configuration, SELinux, file permissions, or the listening socket.

Name resolution deserves a separate test. A hostname failure with a successful IP connection usually means DNS or `/etc/hosts`. An IP failure means DNS is not the main problem. `dig` output shows the server queried, the answer section, and timing. If `/etc/resolv.conf` contains the wrong server but NetworkManager owns the file, update the NetworkManager connection rather than making a temporary edit.

Routing faults often show through asymmetry. The client may reach the server, but replies leave through the wrong gateway or interface. `ip route get DESTINATION` shows the chosen route. `tcpdump` on both sides can prove whether packets arrive and whether replies leave. This evidence prevents unnecessary service changes when the real issue is path selection.

Firewall repair should use services where possible because firewalld service definitions capture standard ports and protocols. Custom ports require `--add-port`. Runtime rules disappear after reload or reboot unless made permanent. Permanent rules do not affect the current runtime until reload. The administrator should verify both the runtime state and the persistence requirement.
## Application library and memory faults
Third-party and first-party programs depend on shared libraries. When a binary fails to start, the administrator should check whether required libraries exist, whether the dynamic linker can find them, and whether a package must be installed or restored.

`ldconfig -p` lists libraries in the linker cache. `ldd /path/to/program` lists shared library dependencies for a binary and reports missing libraries. A missing library may require installing the package that owns it, refreshing the linker cache with `ldconfig`, correcting a library path, or replacing a damaged file.

Memory leaks require observation over time. Valgrind provides a direct way to analyse memory use for a process during a test run. `valgrind PROGRAM` runs the default memcheck tool. `valgrind --tool=memcheck PROGRAM` makes the tool explicit. `valgrind --leak-check=full PROGRAM` performs fuller leak analysis. `valgrind --log-file=FILE PROGRAM` writes results to a file. Valgrind slows execution, so it suits reproduction and analysis more than normal production operation.

Tracing tools reveal what an application does before it fails. `ltrace` traces library calls. `strace` traces system calls. Both support similar workflows: `-o FILE` writes output, `-p PID` attaches to a running process, and `-e EXPRESSION` filters events. Use `ltrace` when library interaction seems relevant, and use `strace` when file access, permissions, signals, sockets, or kernel calls seem relevant.

Application debugging should start with the smallest reproducible command. If a program fails only under systemd, compare the systemd environment, working directory, user, SELinux context, limits, and permissions with a manual run. If it fails manually as the same user, tracing and library checks become simpler.

Shared library errors may also come from architecture mismatch or wrong library versions. A binary built for one architecture or linked against a missing soname will not run merely because a similarly named package exists. `file PROGRAM` identifies the binary type. `rpm -qf LIBRARY` identifies the owning package for installed libraries. `dnf provides '*/LIBRARY'` locates a package for a missing library path or soname.

Tracing output can be large. Filters make it useful. For file and permission problems, `strace -e trace=file PROGRAM` can reveal missing paths and denied access. For network problems, socket-related traces may show connection attempts. For library behaviour, `ltrace` can show repeated calls or failed library functions. The trace should answer the hypothesis rather than collect every possible event.
## SELinux context and policy faults
SELinux enforces access through labels, policy, booleans, and port mappings. A service can have correct Unix permissions and still fail when SELinux blocks access. Diagnosis should prove SELinux involvement before changing policy.

The `ls -Z` option shows SELinux context. A context contains user, role, type, and level. The type field drives many access decisions. For example, Apache configuration files, log files, and web content use different expected types. Copying or moving files into service paths can leave them with unsuitable labels.

Useful SELinux discovery commands include:
- `semanage login -l` to list SELinux user mappings.
- `semanage user -l` to list SELinux users and roles.
- `semanage fcontext -l` to list file context rules.
- `semanage port -l` to list labelled ports.
- `seinfo -u`, `seinfo -r`, `seinfo -t`, and `seinfo -p` for policy information where setools is installed.

The `chcon` command changes a label immediately, but it does not change the persistent labelling rule. It suits tests. The `restorecon -Rv PATH` command restores labels from policy and persistent file context rules. Persistent custom file labels use `semanage fcontext -a -t TYPE 'PATH_REGEX'` followed by `restorecon`.

SELinux troubleshooting starts with proof. Temporarily setting permissive mode with `setenforce 0` and repeating the failing action can show whether SELinux caused the block. Restore enforcing mode with `setenforce 1` after the test. Do not leave the system permissive as a fix.

SELinux denial evidence appears in `/var/log/audit/audit.log` when auditd is active. `ausearch -m avc -ts recent` finds recent Access Vector Cache denials. `sealert -a /var/log/audit/audit.log` gives interpreted messages and often suggests commands. Apply suggestions only after confirming they match the service and path.

SELinux booleans enable supported policy behaviour without custom policy. `getsebool -a` lists booleans. `getsebool httpd_enable_homedirs` checks one boolean. `setsebool httpd_enable_homedirs on` changes it at runtime. `setsebool -P httpd_enable_homedirs on` makes it persistent. Booleans are the right fix when the policy already supports the behaviour, such as allowing httpd to serve content from home directories.

A common SELinux pattern involves web content outside the default document root. The service may need the correct file type, suitable Unix permissions, and an enabled boolean. Changing only one layer can leave the service broken. For Apache serving content from a home directory, the administrator must consider the content label, directory execute permissions, and the `httpd_enable_homedirs` boolean.

Another common pattern involves ports. A service configured to listen on a non-standard port may fail under SELinux even when the firewall allows the port. `semanage port -l` shows which types own which ports. If policy supports the service on that port type, `semanage port -a -t TYPE -p tcp PORT` adds a persistent mapping. If the port already exists under another type, use `-m` to modify it only when that change is correct.

SELinux denials can include suggestions that are technically valid but conceptually too broad. Generating a custom policy from every denial may grant more access than needed. Prefer the smallest supported fix: restore labels, set a documented boolean, or add a proper file context or port mapping. Custom policy belongs after those options fail and after the denial clearly matches the intended behaviour.
## PAM and authentication faults
Pluggable Authentication Modules, or PAM, control authentication, account checks, password updates, and session setup. PAM configuration lives under `/etc/pam.d`. Each service file contains ordered rules. A rule includes a module type, control value, module path or name, and module arguments.

The four major PAM module types are:
- `auth`, which authenticates credentials.
- `account`, which checks account validity and access.
- `password`, which manages password changes.
- `session`, which sets up and tears down session state.

Control values determine how PAM reacts to success or failure. Common values include `required`, `requisite`, `sufficient`, and `optional`. More detailed bracket syntax can make decisions based on exact return codes. A missing, reordered, or damaged PAM rule can block a service even when users and passwords are correct.

RHEL 8 uses authselect rather than the old authconfig workflow. Authselect manages selected authentication profiles and generates related PAM configuration. Manual changes to generated files may be overwritten or may place the system outside a supported profile. `authselect current` shows the selected profile. `authselect list` lists available profiles. `authselect select PROFILE` changes profile. `authselect apply-changes` applies updates.

A service-specific PAM problem often shows in that service's file under `/etc/pam.d`. If a package-owned PAM file has been damaged, move the broken file aside and reinstall the owning package. For example, a damaged Samba PAM file can be restored by reinstalling the relevant Samba package after preserving the faulty file for comparison. Then restart the service and retest authentication with the client utility.

Authentication failures should be split into identity, authentication, account authorisation, and session setup. A user absent from `getent passwd` has an identity lookup problem. A user present in `getent passwd` but unable to authenticate may have a password, Kerberos, PAM auth, or SSSD issue. A user who authenticates but cannot access a service may fail account rules. A user who starts login but receives a session error may hit session modules, home directory problems, SELinux, or resource limits.

PAM order matters because modules run in sequence. A `sufficient` success can short-circuit later modules when no prior required module has failed. A `required` failure may allow later modules to run but still fail the stack. A `requisite` failure stops immediately. These control values explain why adding one misplaced line can change the whole authentication result.

Authselect should be treated as the owner of generated authentication configuration. If a task requires SSSD, select an SSSD-capable profile. If local custom PAM changes are required, use an authselect custom profile rather than editing generated files in a way that future authselect operations overwrite. In a repair scenario, comparing generated files to the active profile can reveal manual damage.
## SSSD, LDAP, and Kerberos client configuration
LDAP and Kerberos identity problems on RHEL 8 normally involve SSSD and authselect. OpenLDAP server setup is not the focus. The system acts as a client that uses SSSD to reach identity and authentication services.

SSSD configuration lives in `/etc/sssd/sssd.conf`. The file must use correct permissions, normally root ownership and mode 600, because it can contain sensitive configuration. It defines services, domains, providers, LDAP URIs, TLS certificate paths, Kerberos realms, and related identity settings.

Common checks include:
- `systemctl status sssd` to confirm the daemon runs.
- `journalctl -u sssd` to inspect service errors.
- `sssctl config-check` where available to validate configuration.
- `getent passwd USER` to test identity lookup.
- `id USER` to test user resolution.
- `kinit USER` to test Kerberos where applicable.
- `authselect current` to confirm the selected profile matches SSSD use.

LDAP faults may involve DNS, TLS certificates, bind credentials, base DN, firewall access, or wrong provider settings. Kerberos faults may involve realm names, time synchronisation, KDC reachability, DNS records, or keytab issues. The administrator should validate identity lookup and authentication separately because one can work while the other fails.

SSSD caches identity information, which can confuse testing. A cached user may appear after the directory service becomes unavailable. Clearing cache is sometimes useful, but it should not replace fixing the connection. Logs under SSSD and the journal show provider, domain, TLS, and Kerberos errors. Debug levels can increase detail when ordinary logs do not explain the failure.

Time synchronisation is critical for Kerberos. A client with significant clock drift may resolve users through LDAP but fail Kerberos authentication. DNS also matters because realms, KDCs, and service discovery often rely on names. Therefore, an identity management fault may require checking `chronyd`, DNS, routes, certificates, and SSSD together.

Permissions on `/etc/sssd/sssd.conf` are a frequent source of failure. SSSD refuses unsafe configuration files. The file should be owned by root and readable only by root. After repair, restart SSSD and test identity lookup with commands that do not require a full login first. Then test authentication and service access.
## Kernel crash dumps
Kdump captures kernel crash dumps for later analysis. It reserves memory for a crash kernel, boots that kernel after a panic, and writes a vmcore. The administrator enables it with `systemctl enable --now kdump`.

The main configuration file is `/etc/kdump.conf`. The default dump location is usually under `/var/crash`. Configuration can direct dumps elsewhere, including local paths or remote targets. After changing configuration, restart kdump and confirm status.

Crash dumps support third-party investigation because they preserve kernel state at failure time. For safe testing, use a controlled virtual machine rather than a production host. Forced crashes can damage work and interrupt services.
## SystemTap modules
SystemTap instruments a running Linux system by compiling scripts into kernel modules or running probes through supported mechanisms. It helps gather diagnostic information for issues that ordinary logs and metrics do not expose.

SystemTap setup may require enabling debug repositories and installing packages that match the running kernel, including kernel debuginfo or kernel-devel packages where required. Version mismatch is a common source of failure.

Default tapsets live under `/usr/share/systemtap/tapset`. Not every `.stp` file is a standalone script. Some files provide reusable probe definitions for other scripts. The administrator must distinguish runnable scripts from library-style tapsets.

Important commands include:
- `stap SCRIPT.stp` to compile and run a script.
- `stap -v SCRIPT.stp` to show verbose compilation and run information.
- `stap -p4 -m NAME SCRIPT.stp` to stop after module generation and name the module.
- `staprun NAME.ko` or `staprun NAME` to run a compiled SystemTap module where appropriate.

A generated `.ko` file is a kernel module. Running it requires suitable permissions and compatible kernel support. If SystemTap reports missing kernel packages, install the exact package variant suggested for the running kernel. If a probe hangs or produces no output, confirm that the probed event actually occurs during the test.

Crash dump collection has two distinct goals. The first goal is readiness before a crash. The second goal is evidence after a crash. Readiness means kdump is enabled, has reserved memory, has a valid target, and starts cleanly. Evidence means the vmcore exists after a crash and can be handed to the team that will analyse it.

SystemTap should be used with the same restraint as any kernel-level diagnostic tool. A probe can generate overhead, produce large output, or fail when kernel symbols do not match. The administrator should run a narrow script, reproduce the event, capture the data, and stop the probe. This keeps investigation focused and reduces the risk of creating a new performance issue.

For EX342, the important SystemTap skill is operational. The candidate should know how to install the needed packages, recognise missing kernel support, locate scripts, run a script with `stap`, compile a module with `-p4 -m`, and run a compiled module with `staprun`. The candidate does not need to become a full SystemTap developer to gather useful diagnostic evidence.