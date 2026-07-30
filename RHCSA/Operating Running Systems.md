# Operating Running Systems
> [!NOTE]
> A practical guide to operating RHEL systems safely through controlled shutdowns, root password recovery, systemd service management, evidence-based performance tuning, and comprehensive logging.
# Operating Running Systems on RHEL 8
Red Hat Enterprise Linux 8 uses `systemd` to control booting, services, targets, shutdown, and the system journal. Administrators also rely on standard Linux tools to inspect processes, adjust scheduling influence, apply TuneD profiles, manage traditional logs, and transfer files securely. Safe administration requires planned changes, verified commands, suitable privileges, and access to a test or recovery console.
## Shutting down and restarting systems
### Planned shutdowns
The `shutdown` command suits planned maintenance because it can schedule an action and send a wall message to logged-in users. It accepts an absolute 24-hour time, a delay in minutes, or `now`.

| Action | Command |
| --- | --- |
| Power off at 17:00 | `sudo shutdown --poweroff 17:00 "Maintenance begins at 17:00"` |
| Power off in 20 minutes | `sudo shutdown --poweroff +20 "Maintenance begins in 20 minutes"` |
| Reboot in 10 minutes | `sudo shutdown --reboot +10 "Kernel update requires a restart"` |
| Cancel a scheduled shutdown | `sudo shutdown -c` |

RHEL creates `/run/nologin` five minutes before a scheduled shutdown. Login programs that honour this file reject new non-root sessions, while existing sessions continue. The shutdown process removes the file if an administrator cancels the event.

An administrator can independently block new non-root logins during maintenance by creating `/etc/nologin`. Text inside the file can explain the restriction to users. The administrator must remove the file after maintenance.

```bash
sudo sh -c 'printf "%s\n" "Maintenance in progress" > /etc/nologin'
sudo rm /etc/nologin
```

Not every application checks `nologin`, so administrators must confirm the behaviour of each remote-access service. They should also preserve at least one authorised administrative session before restricting access.
### Immediate power actions
`systemctl poweroff`, `systemctl reboot`, `poweroff`, and `reboot` request an immediate action through `systemd`. `systemd` sends a wall message by default. These commands still provide no advance interval in which users can finish work, so `shutdown` remains the safer choice for planned multi-user maintenance. The `--no-wall` option suppresses the broadcast when an operational procedure specifically requires that behaviour.

```bash
sudo systemctl reboot
sudo systemctl poweroff
```

Administrators should check for interactive users, active jobs, mounted remote storage, databases, and other stateful applications before any power action. Commands such as `who`, `w`, `loginctl list-sessions`, and service-specific status checks help establish whether shutdown can proceed safely.

A controlled maintenance sequence usually starts with a change window and a rollback plan. The administrator announces the work, checks current activity, schedules the shutdown, watches for objections, and cancels the event if the agreed conditions change. Immediately before the action, the administrator confirms that backups and replication have reached the required state. After the restart, the administrator verifies time synchronisation, network connectivity, mounted file systems, failed units, application health, and monitoring.

```bash
systemctl --failed
findmnt
ip address show
timedatectl
```

`shutdown --halt` stops the operating system but does not necessarily remove power. `shutdown --poweroff` stops the system and requests power removal. `shutdown --reboot` restarts it. Explicit long options improve the readability of operational records, even though shorter forms such as `-H`, `-P`, and `-r` remain available.
## Recovering a root password
A root-password reset requires authorised console access. A remote shell does not suffice because the administrator must interrupt GRUB before the operating system starts. A hypervisor console can provide the required access for a virtual machine. Full-disk encryption can still require the storage-unlock credential during boot.

If an administrator can sign in through another account with `sudo` rights, the simplest recovery path avoids GRUB:

```bash
sudo passwd root
```

If no privileged account remains available, Red Hat documents an `rd.break` recovery procedure:
1. The administrator reboots the system and presses `e` at the GRUB menu.
2. The administrator appends `rd.break` to the end of the kernel line that begins with `linux`.
3. `Ctrl+x` starts the modified boot entry and stops in the initial RAM file system at a `switch_root` prompt.
4. The administrator remounts the installed root file system as writable.
5. The administrator changes root into the installed system and sets a new password.
6. The administrator requests an SELinux relabel on the next boot.
7. Two `exit` commands leave the changed root and continue the boot process.

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
exit
exit
```

The SELinux relabel can take a long time on a large file system, and the system can reboot again when relabelling finishes. The administrator should wait for completion and then verify the new credential.

An alternative recovery technique adds `enforcing=0`, changes the password, runs `restorecon` on `/etc/shadow` after boot, and restores enforcing mode. That technique can work in a controlled environment, but Red Hat's current procedure requests a full automatic relabel. The full relabel reduces the risk that the recovery operation leaves another changed file with an incorrect SELinux label.

The edited GRUB entry normally applies to that boot only. The administrator should not save `rd.break` as a permanent kernel argument. Before leaving the recovery environment, the administrator should confirm that `passwd` reported success and that `/.autorelabel` exists inside the changed root. An unexpected mount layout, read-only storage error, or failed password update requires investigation before the administrator continues booting.

Physical or virtual console access can defeat an operating-system password when GRUB permits kernel-line editing. Organisations should therefore control console access, protect bootloader settings where required, encrypt sensitive storage, and audit recovery activity.
## Managing services with systemd
### Units and service state
`systemd` runs as process ID 1 and manages resources through units. Common unit types include services, sockets, timers, paths, mounts, and targets. The `systemctl` command inspects and changes their state.

The following commands distinguish current runtime state from installed unit-file state:

```bash
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service
systemctl status chronyd.service
systemctl is-active chronyd.service
systemctl is-enabled chronyd.service
```

`list-units` shows units that `systemd` has loaded. `list-unit-files` shows installed unit files and their enablement state, including disabled units that are not loaded. Running `systemctl status` without a unit shows an overall system status and a unit tree. It does not serve as the precise command for listing every running service.

Starting and enabling perform different operations. `start` activates a unit in the current boot. `enable` creates the links or dependencies that start it during a later boot, but does not normally start it immediately. The `--now` option combines the runtime and boot-time operations.

| Required change | Command |
| --- | --- |
| Start now | `sudo systemctl start atd.service` |
| Stop now | `sudo systemctl stop atd.service` |
| Restart now | `sudo systemctl restart atd.service` |
| Reload supported configuration | `sudo systemctl reload service-name.service` |
| Enable for boot | `sudo systemctl enable atd.service` |
| Enable and start | `sudo systemctl enable --now atd.service` |
| Disable for boot | `sudo systemctl disable atd.service` |
| Disable and stop | `sudo systemctl disable --now atd.service` |

`systemctl status` combines state, recent journal entries, the main process ID, resource information, and the unit-file path. A green or active state does not prove that an application works end to end. An administrator must still test the service interface and inspect its logs.

Unit states describe different dimensions. `loaded` means that `systemd` found and parsed the definition. `active`, `inactive`, and `failed` describe runtime state. `enabled`, `disabled`, `static`, and `masked` describe unit-file enablement. A static unit has no normal installation instructions of its own and usually starts as another unit's dependency. It does not indicate a fault.

When a start operation fails, the administrator can review the failure and the corresponding journal before resetting the recorded failed state:

```bash
systemctl status example.service
journalctl -b -u example.service
systemctl show example.service -p Result -p ExecMainStatus
sudo systemctl reset-failed example.service
```

`reset-failed` clears the recorded state and restart counters. It does not correct the underlying configuration, dependency, permission, resource, or application error.

`systemctl` accepts several unit names in one command:

```bash
sudo systemctl restart chronyd.service crond.service
```

Shell wildcards can expand more broadly than intended. Administrators should inspect the matching units before applying a state-changing command to a pattern.

Dependencies and ordering shape service behaviour. Directives such as `Requires=`, `Wants=`, `After=`, and `Before=` express different relationships. `Requires=` adds a stronger activation dependency, while `Wants=` adds a weaker one. `After=` and `Before=` control ordering but do not independently pull another unit into the transaction. Administrators should inspect the effective dependency graph rather than assume that ordering also causes activation.

```bash
systemctl list-dependencies example.service
systemctl show example.service -p Requires -p Wants -p After
```
### Socket activation
A socket unit can listen for connections and start its associated service on demand. For example, Cockpit commonly uses `cockpit.socket` on TCP port 9090. `systemd` holds the listening socket, then activates the web service when a client connects. This design can reduce the resources consumed by an idle service.

```bash
sudo systemctl enable --now cockpit.socket
systemctl status cockpit.socket
sudo ss -lntp
```

Socket activation does not remove the need for firewall rules, SELinux policy, authentication, or application security. An open local socket also does not prove that a remote client can reach the service.

Timer units provide another activation method. They can replace many recurring cron tasks and can link their execution history to the journal. `systemctl list-timers` shows the next and previous activation times. A persistent timer can catch up after downtime when its unit sets `Persistent=true`, subject to the timer's definition.

```bash
systemctl list-timers --all
systemctl status timer-name.timer
journalctl -u timer-name.service
```
### Unit files and overrides
RHEL 8 reads system unit files from directories with a defined precedence:

| Directory | Purpose and precedence |
| --- | --- |
| `/usr/lib/systemd/system` | Unit files supplied by installed packages |
| `/run/systemd/system` | Runtime units and overrides, which take precedence over package files |
| `/etc/systemd/system` | Administrator-managed units and overrides, which take highest precedence |

Administrators should not edit package files under `/usr/lib/systemd/system` because a package update can replace them. `systemctl cat` shows the fragments that form the effective unit:

```bash
systemctl cat atd.service
```

`systemctl edit atd.service` normally creates a drop-in override under `/etc/systemd/system/atd.service.d/`. A drop-in changes only the required directives and leaves the vendor unit intact. `systemctl edit --full atd.service` copies the complete unit into `/etc`, which replaces the vendor definition. A full replacement demands more maintenance because later vendor changes do not flow into the copied file.

After a manual unit-file change, `systemctl daemon-reload` makes the manager reread unit definitions. The administrator must then restart or reload the affected service when its changed settings require it.

```bash
sudo systemctl edit atd.service
sudo systemctl daemon-reload
sudo systemctl restart atd.service
systemctl cat atd.service
```

An empty assignment in a drop-in can reset some list-valued directives before the override supplies replacement values. Unit-specific manual pages define which directives support that pattern. `systemd-delta` identifies local overrides, and `systemd-analyze verify` can expose some unit-file errors before activation.

```bash
systemd-delta
systemd-analyze verify /etc/systemd/system/example.service
```
### Masking units
Disabling a unit removes its boot-time enablement but still permits manual or dependency-driven activation. Masking links the unit name to `/dev/null`, which blocks activation by normal means. `mask` alone does not stop an already running service. `mask --now` both masks and stops it.

```bash
sudo systemctl mask --now atd.service
systemctl is-enabled atd.service
sudo systemctl unmask atd.service
```

Masking can prevent conflicting services from starting, but it can also block a required dependency. Administrators should record the reason for a mask and confirm its effect before using it on a production system.
### Targets and recovery modes
Targets group units and represent system states. `multi-user.target` broadly corresponds to the old non-graphical runlevel 3, while `graphical.target` broadly corresponds to runlevel 5. The relationship remains an approximation because targets can express dependencies more flexibly than SysV runlevels.

```bash
systemctl get-default
systemctl list-units --type=target
sudo systemctl set-default multi-user.target
```

`systemctl isolate target-name.target` starts units required by the selected target and stops units that the target does not require. Isolation can terminate services and user sessions, so an administrator must confirm that the unit allows isolation and that console access remains available.

`systemctl rescue` changes the running system to rescue mode and broadcasts a message. `emergency.target` provides a smaller environment with the root file system mounted read-only. An administrator can select a one-off boot target by appending a parameter such as `systemd.unit=rescue.target` to the GRUB kernel line. Changing the kernel line for one boot does not alter the saved default target.

Rescue mode starts a limited base system and normally mounts local file systems, while emergency mode starts fewer units and gives the administrator an earlier repair environment. Network access may disappear in either mode. A remote-only administrator should not isolate either target without a tested out-of-band console.
## Observing performance and managing processes
### Establishing a baseline
Performance data gains meaning when administrators compare it with the same system under a known healthy workload. A useful baseline records logical CPU count, memory, swap, storage latency, network throughput, load average, and application response time. A single command rarely identifies the cause of a slowdown.

`uptime` reports the current time, elapsed time since boot, logged-in user count, and load averages over approximately 1, 5, and 15 minutes:

```bash
uptime
nproc
lscpu
```

Load average does not equal CPU utilisation. It represents the average number of tasks that are runnable or waiting in uninterruptible sleep, commonly for input or output. A high value can therefore reflect CPU demand, blocked storage activity, or both.

Load averages are not normalised by CPU count. Comparing load with the number of online logical CPUs gives a rough indication of scheduling pressure, not a percentage of CPU use. On a system with four online logical CPUs, a sustained load near four suggests roughly one runnable or uninterruptible task per logical CPU. The system can still show idle CPU time if blocked tasks inflate the load.

The `CPU(s)` field in `lscpu` already reports the total number of logical CPUs. It must not be multiplied by `Core(s) per socket`. A report of `CPU(s): 2` means two logical CPUs, even if another field reports two cores per socket. `nproc` provides a direct count of processing units available to the current process.
### Using top and process listings
`top` displays load averages, task states, CPU-use categories, memory, swap, and a refreshable process table. Its `%Cpu(s)` line measures CPU time directly and therefore complements, rather than duplicates, load average. High `wa` can indicate time waiting for input or output, while high `us` or `sy` indicates user-space or kernel CPU work.

Memory interpretation also requires care. Linux uses otherwise idle memory for caches, so a small `free` value does not alone indicate pressure. The `available` estimate, swap activity, reclaim behaviour, and application latency provide better evidence. `free -h`, `vmstat 1`, and `pidstat` can extend a first `top` inspection. Sustained swapping, blocked tasks, or increasing input or output wait calls for storage and memory analysis rather than an immediate CPU-priority change.

Process state letters help separate causes. `R` identifies running or runnable work, `S` identifies interruptible sleep, `D` usually identifies uninterruptible sleep, `T` identifies stopped work, and `Z` identifies a zombie whose parent has not collected its exit status. Killing a zombie has no effect because the process has already exited. The administrator must address its parent or the parent application's defect.

`ps` provides a snapshot. Explicit output fields make investigations easier to repeat:

```bash
ps -eo pid,ppid,user,stat,ni,pri,%cpu,%mem,comm,args --sort=-%cpu
ps -p 1234 -o pid,ppid,user,stat,ni,pri,etime,cmd
```

`pgrep` searches process names by default, and `pgrep -f` searches full command lines. Exact matching avoids accidentally selecting similarly named processes.

```bash
pgrep -a sshd
pgrep -x sleep
pgrep -af 'application --worker'
```

`pkill` sends a signal to processes selected by similar criteria. Administrators should inspect matches before signalling them.

```bash
pgrep -a -x application
pkill -TERM -x application
```

Broad name matches can affect unrelated users or service instances. Where systemd owns the workload, `systemctl stop unit-name.service` gives the service manager a coherent view of the change and follows the unit's configured stop behaviour.
### Jobs and signals
Appending `&` starts a shell job in the background. `jobs` displays jobs owned by the current shell, `fg` returns a job to the foreground, and `bg` resumes a stopped job in the background. These shell jobs differ from persistent system services and can end when the session closes.

```bash
sleep 1000 &
jobs
fg %1
```

`kill` sends `SIGTERM` by default. This signal asks a process to shut down cleanly and gives it an opportunity to flush data and release resources. If the process does not respond, the administrator should inspect its state, dependencies, and logs before escalating. `SIGKILL` stops a process without cleanup and can cause lost work or application corruption.

```bash
kill -TERM 1234
kill -KILL 1234
kill -l
```

Other signals serve application-specific purposes. `SIGHUP` often requests a configuration reload or log-file reopen, but each application defines its own response. `SIGSTOP` pauses a process and `SIGCONT` resumes it. Administrators should use the service's documented interface instead of assuming that a traditional signal has a universal meaning.
### Niceness and scheduling influence
For normal time-sharing processes, niceness ranges from -20 to 19. A lower value gives a process more favourable treatment, while a higher value gives it less favourable treatment relative to competing work. Niceness influences scheduler weighting. It does not place a process at a numbered position in a fixed queue, and the `PRI` value shown by `ps` should not be described as an ordinal queue position.

`nice` starts a command with an adjusted value. `renice` changes an existing process:

```bash
nice -n 10 long-running-command
renice -n 15 -p 1234
ps -p 1234 -o pid,ni,pri,cmd
```

An unprivileged user can normally increase the nice value of that user's own processes, which lowers their scheduling preference. Lowering the nice value usually requires the appropriate privilege or resource limit. Niceness cannot guarantee a CPU share or impose a hard CPU cap. Administrators should use `systemd` resource controls or control groups when a workload requires enforceable allocation.
### TuneD profiles
TuneD applies collections of settings for workload classes such as virtual guests, virtual hosts, desktops, throughput-oriented servers, and latency-sensitive systems.

```bash
tuned-adm recommend
tuned-adm list
tuned-adm active
sudo tuned-adm profile virtual-guest
tuned-adm verify
```

The recommended profile provides a starting point, not a substitute for measurements. Administrators should record the old profile, apply the new profile in a controlled window, verify it, and compare workload results. Some profile settings apply immediately, while settings that change boot parameters require a reboot before they take full effect.

TuneD can change several subsystems at once. A profile change can therefore improve one workload while increasing power use or latency elsewhere. `tuned-adm verify` confirms that current settings match the selected profile, but application-level measurements determine whether the profile suits the host.
## Diagnosing services and managing logs
### A focused diagnostic sequence
An efficient service investigation moves from state to evidence:
1. The administrator confirms the unit name and checks `systemctl status unit-name`.
2. The administrator checks whether the unit is active, failed, enabled, or masked.
3. The administrator reviews the unit's journal with an appropriate time or boot filter.
4. The administrator checks listening sockets, dependent services, configuration syntax, permissions, storage, and network controls.
5. The administrator changes one justified condition, tests the service, and records the result.

Disabling an unfamiliar failed service can hide the symptom while breaking authentication or another dependency. For example, SSSD can provide remote identity and authentication through LDAP, Active Directory, Kerberos, or Identity Management. An administrator must establish whether the host uses those functions before stopping or disabling `sssd.service`.

Configuration validation should precede a restart whenever the application provides a checker. Web servers, SSH, Rsyslog, and many databases can parse configuration without replacing the running process. This approach preserves service availability when a proposed file contains a syntax error. The administrator should also capture the exact error, time, unit, process ID, and recent change before log rotation or a later restart removes useful context.
### The system journal
`systemd-journald` collects structured events from the kernel, service output, syslog interfaces, and other sources. `journalctl` reads and filters the journal without requiring the administrator to know a traditional log-file path.

```bash
journalctl -b
journalctl -b -1
journalctl --list-boots
journalctl -u sshd.service
journalctl -u sshd.service --since "6 hours ago"
journalctl -p warning
journalctl -n 50
journalctl -f
```

`-b` selects the current boot, while `-b -1` selects the previous boot when persistent records exist. `-u` selects a unit, `-p` filters by priority, `-n` limits the number of entries, and `-f` follows new entries. Relative time expressions require a valid form such as `--since "6 hours ago"`. The shortened form `--since -6` does not clearly express a six-hour interval.

Journal priorities follow syslog levels from the most urgent to the least urgent: `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, and `debug`. A single threshold such as `-p warning` selects that level and all more urgent levels. Combining boot, unit, time, and priority filters usually produces more useful evidence than paging through the entire journal.

The journal records structured fields such as `_SYSTEMD_UNIT`, `_PID`, `_UID`, and the executable path. Administrators can request verbose output when those fields help correlate an event:

```bash
journalctl -b -u sshd.service -p warning -o verbose
```

On RHEL 8, the default `Storage=auto` behaviour stores the journal under `/var/log/journal` when that directory exists and otherwise uses volatile storage under `/run/log/journal`. Volatile records disappear at reboot. Administrators can create `/var/log/journal`, or they can set persistent storage explicitly in `/etc/systemd/journald.conf`:

```ini
[Journal]
Storage=persistent
```

RHEL 8 applies the explicit change after an administrator restarts `systemd-journald`:

```bash
sudo systemctl restart systemd-journald.service
journalctl --disk-usage
journalctl --list-boots
```

Persistent logging consumes disk space. Administrators should also review retention settings, available capacity, access permissions, and any central collection requirement.

The journal rotates its own binary files and does not use logrotate. Settings such as `SystemMaxUse=` and `SystemKeepFree=` constrain persistent journal storage. `journalctl --vacuum-time=` and `journalctl --vacuum-size=` remove archived journal files, but administrators should set a retention policy before an incident and preserve records subject to audit or legal requirements.
### Rsyslog and traditional files
RHEL 8 combines the journal with Rsyslog. `rsyslogd` reads syslog messages from the journal, filters them, writes selected events to files under `/var/log`, and can forward them to remote systems. Its main configuration file is `/etc/rsyslog.conf`, and administrator-supplied fragments normally use the `.conf` suffix under `/etc/rsyslog.d`.

Rsyslog selectors combine a facility with a severity. A rule such as `local1.warning` selects warning and more urgent messages from the `local1` facility. `local1.=warning` selects only warning messages.

```text
local1.warning    /var/log/application.log
```

Before restarting the service, the administrator should validate the complete configuration:

```bash
sudo rsyslogd -N 1
sudo systemctl restart rsyslog.service
logger -p local1.warning "application log test"
sudo tail -n 20 /var/log/application.log
sudo tail -f -n 0 /var/log/application.log
```

The same event can appear in the journal and in more than one traditional file because several rules can match it. Duplicate storage can support different operational needs but also affects retention and disk use.

Facilities classify broad event sources, while severities describe urgency. The local facilities `local0` through `local7` give applications configurable namespaces. Rsyslog processes matching rules in configuration order, and an action does not automatically stop later rules from matching the same event. Administrators should use a descriptive fragment name, document custom facilities, validate syntax, and confirm file ownership and SELinux context.

For central logging, TCP provides delivery feedback that UDP lacks, and RELP can reduce the risk of message loss further. TLS protects logs in transit when the configuration authenticates peers correctly. A secure deployment also protects stored logs, synchronises system clocks, and monitors forwarding queues.
### Rotating logs correctly
Logrotate prevents traditional log files from growing without control. A file under `/etc/logrotate.d` can define rotation frequency, retention, compression, ownership, and application-specific post-rotation actions.

```text
/var/log/application.log {
    weekly
    rotate 4
    maxsize 100M
    compress
    missingok
    notifempty
    create 0640 root root
}
```

`maxsize 100M` works with `weekly` and permits an earlier rotation when the file exceeds the threshold. The `size` directive behaves differently because it is mutually exclusive with time-based criteria, and the last conflicting directive takes precedence. Combining `weekly` with `size 100M` does not reliably express "weekly or earlier at 100 MB". `maxsize` expresses that policy.

Many daemons keep a file descriptor open after logrotate renames a file. Their rotation policy should send the documented reload or signal in a `postrotate` block so the daemon reopens its log. `copytruncate` copies the current file and truncates it in place, but a small interval between those operations can lose records. Administrators should use `copytruncate` only when the application cannot reopen its log safely.

The administrator can test a policy without rotating files, then force a controlled test when appropriate:

```bash
sudo logrotate --debug /etc/logrotate.conf
sudo logrotate --force /etc/logrotate.conf
```

Production testing must account for current state, ownership, SELinux labels, application behaviour, available storage, and any ingestion agent that tails the file.

Rotation frequency depends on how often the scheduler invokes logrotate. A `maxsize` threshold cannot trigger until logrotate runs. A rapidly growing application may therefore require a more frequent timer or an application-native rotation mechanism. Retention counts also need enough storage for compressed archives and enough history for incident investigation.
## Transferring files with OpenSSH
`scp` transfers files through SSH encryption and authentication. `sftp` provides an interactive alternative. Key-based authentication avoids reusable account passwords in automated transfers, but the private key still requires protection.

An administrator can generate a key pair with a passphrase, install only the public key on the destination account, verify the host key, and test the login before copying data. `ssh-agent` can cache an unlocked private key for a session.

```bash
ssh-keygen -t rsa -b 3072
ssh-copy-id bob@server.example.com
ssh bob@server.example.com
scp report.log bob@server.example.com:/tmp/
```

The destination path follows the colon after the host name. The remote account must have write permission to that directory. Administrators should avoid blank private-key passphrases outside tightly controlled automation, restrict key scope where possible, preserve correct `~/.ssh` permissions, and verify the destination host fingerprint through a trusted channel.

`scp -r` copies directory trees, but `rsync` over SSH can efficiently update large trees and retain partial transfers with suitable options when it is available. Sensitive log collections should preserve required metadata, restrict destination access, and use checksums or another integrity check when an investigation depends on exact bytes. Administrators must never copy a private key to the remote account as a substitute for installing its public key.