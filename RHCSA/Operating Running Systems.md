# Operating Running Systems
> [!NOTE]
> This file is a practical RHEL/Linux systems administration guide that explains safe shutdown and login lockout procedures, root password recovery with SELinux label repair, service management with systemd, performance monitoring and tuning, and logging, rotation, journaling, and secure log transfer workflows.
## Shutdown and restart
Shared systems need predictable shutdowns. `shutdown` is the preferred command because it schedules the event, warns logged-in users and can block new logins shortly before the system goes down. `reboot` and `poweroff` suit isolated lab work, but they provide no useful warning and give users no time to save work.

RHEL accepts several common shutdown patterns:

```bash
sudo shutdown -r +20 "System reboot in 20 minutes"
sudo shutdown -h 17:00 "Maintenance window starts at 17:00"
sudo shutdown now
sudo shutdown -c
```

`-r` requests a reboot. `-h` requests a halt or power-off path, depending on the platform and systemd handling. `now` runs immediately. `-c` cancels a scheduled event. Time values can use either an absolute 24-hour clock or a relative value such as `+20`.

On multi-user systems, the most important feature is the warning path. `shutdown` sends a wall message so users know what is about to happen. That behaviour turns shutdown into an administrative action rather than a surprise failure. It also gives operations staff room to negotiate delay if a critical task is still running.

RHEL can also block new sessions without shutting the system down. Creating `/etc/nologin` denies new logins for standard users while leaving the root account available. Existing sessions stay active. That makes `/etc/nologin` useful during maintenance when administrators need stability but do not want to terminate live work immediately.

```bash
sudo touch /etc/nologin
sudo rm /etc/nologin
```

`shutdown` automates a similar control path. When less than five minutes remain before the shutdown time, it creates `/run/nologin`. At that point, new user sessions are refused. The file disappears again when the event is cancelled. This distinction matters. `/etc/nologin` is a deliberate administrative lockout. `/run/nologin` is a temporary lockout tied to an imminent shutdown.

In practice, the workflow is simple. An administrator schedules the event, checks whether users can still log in, confirms that the lockout appears near the five-minute mark, and cancels the event if required. That sequence proves both the timing and the access controls.

`reboot` and `poweroff` remain useful, but they are blunt tools. In RHEL they are symbolic links that ultimately call `systemctl`, which passes the request to systemd. The links make the commands convenient, not special. They simply skip the flexibility that `shutdown` provides.

```bash
which reboot
ls -l "$(which reboot)"
which poweroff
ls -l "$(which poweroff)"
```

Those commands show that both utilities resolve back to `systemctl`. The practical lesson is straightforward. On a single-user VM, `sudo reboot` or `sudo poweroff` is fast and harmless. On a shared server, `shutdown` is the safer administrative interface because it handles timing, communication and login control.

Safe shutdown work also depends on privilege. Standard users generally cannot stop the system directly, so administrative commands usually run under `sudo`. That matters during demonstrations and labs because failed attempts often come from missing privileges rather than bad syntax.

A reliable shutdown policy in RHEL therefore follows three rules.

- Use `shutdown` on shared systems
- Use `/etc/nologin` when maintenance needs a login freeze without a reboot
- Reserve `reboot` and `poweroff` for cases where immediacy matters more than user notice

That policy reduces disruption and keeps maintenance predictable.
## Root password recovery
Root password recovery in RHEL depends on GRUB, the initramfs environment and SELinux label restoration. The process works because the system can pause before the main userspace stack starts, mount the real root filesystem, change the password in the correct context, then repair the security label that the password change disrupts.

The boot chain matters. Firmware starts the machine, locates the boot partition and loads GRUB. GRUB then loads the Linux kernel and its initial ramdisk. Because GRUB can edit the kernel command line before boot, it becomes the entry point for recovery work.

The recovery flow begins at the GRUB menu. An administrator highlights the current kernel entry, presses `e`, moves to the line that begins with `linux`, and appends recovery arguments.

```text
rd.break enforcing=0
```

`rd.break` stops the boot sequence in the initramfs shell. `enforcing=0` places SELinux in permissive mode for the next stage. That second argument is essential because changing the password outside the fully loaded system leaves `/etc/shadow` with the wrong context. Permissive mode allows the system to continue booting long enough to repair the label.

From the initramfs shell, the administrator must work on the real root filesystem, not the temporary initramfs root.

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
```

`mount -o remount,rw /sysroot` makes the real root writable. `chroot /sysroot` changes the working root so `passwd` writes to the actual `/etc/shadow`. Without `chroot`, the password change would affect the wrong filesystem and fail to solve the problem.

After changing the password, the shell must unwind cleanly.

```bash
exit
mount -o remount,ro /sysroot
exit
```

The first `exit` leaves the chroot. The remount step returns `/sysroot` to read-only mode. The final `exit` resumes the normal boot sequence.

At the next login prompt, the new root password should work, but SELinux will still complain because `/etc/shadow` no longer has its proper label. The fix happens after the main system starts.

```bash
restorecon -v /etc/shadow
getenforce
setenforce 1
```

`restorecon -v /etc/shadow` restores the correct SELinux context, typically `shadow_t`. `getenforce` confirms the current SELinux mode. `setenforce 1` returns the system to enforcing mode. The source transcript refers to `setenforce=1`, but the correct command form is `setenforce 1`.

The SELinux component deserves special attention because it explains why the method works and why it breaks if the final repair is skipped. SELinux labels files and processes with types that govern access beyond Unix mode bits. `/etc/shadow` must carry a label that authentication tools expect. If that label changes to an unrelated type, authentication can fail even for privileged users. The transcript demonstrates this by changing the label manually with `chcon` and restoring it with `restorecon`.

```bash
ls -Z /etc/shadow
chcon -t user_home_t /etc/shadow
restorecon -v /etc/shadow
```

That experiment shows the principle clearly. An incorrect label can stop `sudo` and similar tools from reading authentication data. `restorecon` resets the label according to policy and restores normal access.

The recovery method has clear operational limits.

- It requires console or bootloader access
- It should be practised on lab systems before it is needed on production hosts
- It must include the SELinux repair step
- It should return the system to enforcing mode after validation

The process is small, but it is exact. Missing any of the core steps can leave the system booted with the wrong label, the wrong root mapping or a password change written to the wrong environment. When it is executed correctly, it restores control without reinstalling the system or altering broader authentication configuration.
## Service management with systemd
RHEL uses systemd as PID 1 and `systemctl` as the main administrative interface. That unifies service handling across modern Linux distributions and replaces the older split between service control and boot-time enablement. In practice, an administrator no longer needs separate tools to start a daemon and register it for startup.

The first diagnostic step is usually `systemctl status`.

```bash
systemctl status chronyd
systemctl status atd
```

`status` shows whether the unit is loaded, enabled, active and running. It also shows the main PID, task count, recent log entries and the time since startup. That makes it the fastest way to decide whether the fault lies in startup, enablement or runtime behaviour.

Starting and enabling services follow a consistent pattern.

```bash
sudo systemctl start atd
sudo systemctl stop atd
sudo systemctl enable atd
sudo systemctl disable atd
sudo systemctl enable --now atd
sudo systemctl disable --now atd
```

`--now` is the most useful shortcut. With `enable`, it starts the service immediately and makes the change persistent across reboots. With `disable`, it stops the service immediately and removes the boot-time symlink. That reduces repetitive two-step administration.

`systemctl` can also handle more than one unit in the same command. An administrator can start or enable multiple services at once, or target matching names with simple wildcards where appropriate.

```bash
sudo systemctl restart crond.service chronyd.service
sudo systemctl enable --now chronyd atd
```

That matters during broader maintenance because service groups often change together.

Systemd is not limited to service units. It also manages sockets, targets, timers and other unit types. Socket activation is especially important because it reduces idle resource use. The course demonstrates this with Cockpit. Instead of running the full web service continuously, RHEL can enable `cockpit.socket` and let systemd hold port `9090` open until a real connection arrives.

```bash
sudo systemctl enable --now cockpit.socket
systemctl list-units --type=socket
systemctl list-unit-files --type=socket
```

In that model, systemd owns the listening socket. When a browser connects, systemd starts the backing service and hands the connection across. That design lowers baseline memory and CPU use for infrequently used services.

Unit discovery also matters. `list-units` shows loaded units. `list-unit-files` shows installed unit files whether they are loaded or not. Filtering with `--type` narrows the result set to services, sockets or targets. Administrators use those views differently. Loaded units help with current state. Unit files help with availability and startup configuration.

Unit file location determines how customisation works. Vendor-supplied defaults live under `/usr/lib/systemd/system`. Local administrative overrides live under `/etc/systemd/system`. Systemd reads the local copy first, so changes in `/etc` override vendor definitions without editing packaged files directly.

```bash
systemctl cat atd
sudo systemctl edit --full atd
sudo systemctl daemon-reload
```

`systemctl cat atd` shows the unit content and the path that systemd is reading. `systemctl edit --full atd` copies the unit into `/etc/systemd/system` and opens it for editing. `systemctl daemon-reload` tells systemd to re-read unit files after those changes. That sequence is safer than editing packaged files because updates can replace files in `/usr/lib/systemd/system`.

Masking is stronger than disabling. Disabling removes startup links, but a manual `start` can still launch the service. Masking links the unit to `/dev/null`, which prevents both manual and automatic starts until the mask is removed.

```bash
sudo systemctl mask --now atd
sudo systemctl unmask atd
```

That makes masking useful when one service must stay completely out of the way, such as during a trial migration from one web server to another.

Targets replace the older SysV runlevels. `multi-user.target` roughly matches the old runlevel 3. `graphical.target` roughly matches runlevel 5. `rescue.target` provides a minimal maintenance environment. Administrators can inspect and change target behaviour with `systemctl`.

```bash
systemctl get-default
sudo systemctl set-default multi-user.target
sudo systemctl isolate graphical.target
sudo systemctl isolate multi-user.target
```

`get-default` shows the persistent default boot target. `set-default` changes it for future boots. `isolate` switches the running system into another target immediately, starting and stopping the relevant units as needed.

A one-off boot change can also happen from GRUB by appending a kernel argument such as the following.

```text
systemd.unit=rescue.target
```

That boots into a restricted maintenance environment. The transcript shows that rescue mode omits normal networking and exposes a much smaller service set. That makes it useful for repair work, but unsuitable for ordinary users. Once maintenance is complete, `systemctl isolate multi-user.target` restores the fuller operational target without a full reboot.

Service administration in RHEL works best when each action follows a clear sequence.

- Check `systemctl status`
- Decide whether the problem is load, startup, enablement or configuration
- Change the unit with `start`, `stop`, `enable`, `disable`, `mask` or `unmask`
- Reload the manager after editing unit files
- Use targets deliberately rather than treating them as obscure boot-time settings

That approach keeps service work predictable and makes the systemd model easier to reason about.
## Performance monitoring and tuning
Performance work in RHEL begins with observation, not tuning. The system already provides enough state to tell an administrator whether the machine is overloaded, whether a single process is consuming too much CPU, and whether the host fits its intended profile.

`uptime` gives the fastest summary.

```bash
uptime
lscpu
```

`uptime` shows the current time, system uptime, logged-in user count and load averages for the last 1, 5 and 15 minutes. Those values only become meaningful when they are compared with available CPU capacity. `lscpu` provides the socket and core counts needed for that comparison.

The source explains load average as a percentage of total cores. A more precise reading is that load average reflects the average number of runnable or uninterruptible tasks. Even so, the practical rule remains useful. On a four-core system, a sustained load around `4` means the CPUs are effectively saturated. A sustained value well above that means work is queueing faster than the processors can service it.

`top` expands the same picture in real time.

```bash
top
```

It shows load averages, task counts, CPU use, memory use and the most CPU-intensive processes. Because it updates continuously and sorts by CPU consumption by default, it is often the quickest way to spot a runaway service or confirm that pressure has subsided. Quitting with `q` returns to the shell immediately.

Shell job control also supports performance work. Long-running commands can move into the background so the same terminal can keep working.

```bash
sleep 1000 &
jobs
```

`&` backgrounds the task. `jobs` lists current shell jobs. That is a small feature, but it matters when administrators need to run a probe or test workload without losing the terminal.

Process inspection in RHEL relies on `ps`, `pgrep`, `pkill` and `kill`.

```bash
ps -elf
pgrep sleep
pkill sleep
kill -15 PID
kill -9 PID
```

`ps -elf` shows a full process list. `pgrep` searches by name without the common nuisance of matching the `grep` process itself. `pkill` sends signals by process name. `kill -15` sends `SIGTERM`, which asks the process to exit cleanly. `kill -9` sends `SIGKILL`, which stops the process immediately and should remain a last resort because it does not allow cleanup.

CPU priority tuning uses nice values. Linux accepts values from `-20` to `19`. Lower values mean higher scheduling priority. Higher values mean the process behaves more politely and receives less CPU attention when the system is busy.

```bash
nice -n 12 sleep 1000 &
ps -lp "$(pgrep sleep)"
renice 19 -p PID
```

A process that starts with the default nice value `0` usually shows a normal priority. Raising the nice value to `12` or `19` reduces the process priority and moves it further back in the scheduling queue. Standard users can generally make their own processes nicer, but they cannot reduce the nice value again or assign negative values without privilege. That restriction prevents users from grabbing unfair priority over the rest of the host.

The transcript links nice values to displayed kernel priorities. That is accurate enough for day-to-day administration, but the key operational point is simpler. Lower nice values increase preference. Higher nice values reduce preference. Administrators should tune only the workloads that genuinely need special treatment.

RHEL also includes system-wide tuning profiles through `tuned-adm`.

```bash
tuned-adm active
tuned-adm recommend
tuned-adm list
sudo tuned-adm profile virtual-guest
```

`tuned-adm active` shows the current profile. `recommend` shows the suggested profile for the host. `list` displays all available profiles. `profile` applies a selected profile persistently. On a virtual machine, `virtual-guest` is usually the right fit. Other profiles prioritise throughput, latency, power saving or desktop responsiveness.

These profiles matter because they package many low-level kernel and hardware tuning choices into a single operational intent. Administrators do not need to hand-edit every setting to move the machine towards a workload pattern. They only need to choose the profile that matches the host role.

A practical performance workflow in RHEL therefore looks like this.

- Use `uptime` and `lscpu` to decide whether the load is normal for the CPU count
- Use `top` to identify the heaviest tasks
- Use `ps`, `pgrep` and `pkill` to inspect or control individual processes
- Use `nice` and `renice` to reduce contention when non-critical work can yield CPU time
- Use `tuned-adm` to align the host with its intended workload

That sequence keeps performance work evidence-based rather than reactive.
## Logging, rotation and secure transfer
Logging in RHEL uses both traditional log files and the systemd journal. Good troubleshooting starts with the service view, then moves outward into broader logs only when necessary.

`systemctl status` is the first checkpoint.

```bash
systemctl status rsyslog
systemctl status sshd
```

It reports whether the service is enabled and active and includes recent log lines from the unit. That makes it ideal for early triage. If the service is stopped, the problem may be startup configuration. If it is running but failing, the recent log excerpt usually points towards the next step.

Traditional logs live under `/var/log`, with `/var/log/messages` acting as the general system log for many services.

```bash
sudo tail /var/log/messages
sudo tail -n 4 /var/log/messages
sudo tail -n 0 -f /var/log/messages
```

`tail` reads the end of the file. `-n` selects the number of lines. `-f` follows the file as new lines arrive. Following the general log in one terminal while restarting or testing a service in another remains one of the most effective diagnostic techniques on a Linux host.

The `logger` command tests the logging path directly.

```bash
logger -p local1.warn "Hello world"
```

The `-p` value combines a facility and severity. Facilities identify the source class. Severities describe importance. `.warn` means warning and above. The transcript also notes that `=info` would mean exactly `info`, while `.info` means `info` and everything more severe.

RHEL still uses rsyslog for traditional logging. Its main configuration file is `/etc/rsyslog.conf`, which includes additional rules from `/etc/rsyslog.d/*.conf`. That split is important because local rules can live in separate files without editing the main package-managed configuration.

A simple custom rule can route facility-based messages to a dedicated file.

```bash
# /etc/rsyslog.d/my.conf
local1.warn    /var/log/my.log
```

After the change, rsyslog must restart.

```bash
sudo systemctl restart rsyslog.service
```

The destination file does not need to exist first. rsyslog creates it when the first matching message arrives. That makes local rule testing straightforward. An administrator adds the rule, restarts rsyslog, emits a test message with `logger`, and checks both the new file and `/var/log/messages`.

Log growth also needs control. RHEL uses logrotate to archive, compress and trim log history. Local rotation rules usually live under `/etc/logrotate.d/`.

```bash
# /etc/logrotate.d/my
/var/log/my.log {
    weekly
    rotate 4
    size 100M
    dateext
    compress
    copytruncate
}
```

`weekly` rotates on a schedule. `rotate 4` keeps four archived copies. `size 100M` forces earlier rotation if the file grows too quickly. `dateext` appends a date suffix. `compress` saves space. `copytruncate` copies the file and truncates the original in place so the writing service can continue using the same pathname without a restart. A manual test can run with `logrotate` against the main configuration.

```bash
sudo logrotate /etc/logrotate.conf
```

The journal provides the second logging path. Unlike traditional logs, it collects messages in a unified database rather than scattering them across many text files. That means administrators do not need to guess which file contains a given event.

```bash
sudo journalctl
sudo journalctl -n 5
sudo journalctl -f
sudo journalctl --since yesterday
sudo journalctl --unit sshd
```

Those commands read the whole journal, the most recent lines, the live stream, events since a point in time, or events for a specific unit. The journal is powerful because it supports time and unit filtering without requiring prior knowledge of file layout.

The course notes that journal storage is memory-resident by default unless the system is configured to persist it. In practice, `Storage=auto` allows persistence when `/var/log/journal` already exists. To guarantee persistence, the configuration can be set explicitly in `/etc/systemd/journald.conf`.

```bash
# /etc/systemd/journald.conf
Storage=persistent
```

After the change, journald must restart.

```bash
sudo systemctl restart systemd-journald
```

Once persistence is enabled, `/var/log/journal` appears and the journal survives reboot. That unlocks boot-aware queries.

```bash
journalctl --list-boots
journalctl -b 0
journalctl -b -1
```

`0` refers to the current boot. `-1` refers to the previous boot. This becomes indispensable when the fault happened before the current login session.

Secure log transfer is the final piece. `scp` copies files over SSH, so it provides confidentiality in transit without extra tooling.

```bash
scp file1 bob@localhost:/tmp
```

The destination syntax follows `user@host:path`. If the path is omitted, the file lands in the remote home directory. The remote user must have write permission to the destination.

The transcript demonstrates key-based authentication by creating an SSH key pair, placing the public key in the remote user's `authorized_keys`, and enforcing secure permissions.

```bash
ssh-keygen
mkdir -m 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Those permissions matter because SSH rejects keys when the directory or key file is too open.

A practical logging workflow in RHEL therefore follows a clean escalation path.

- Start with `systemctl status`
- Follow `/var/log/messages` during live tests
- Add or test rsyslog rules with `logger`
- Control file size with logrotate
- Use `journalctl` for time-based and unit-based history
- Persist the journal if reboot history matters
- Transfer logs with `scp` when analysis must move off-host

That combination gives RHEL both compatibility with classic text logging and the structured reach of the journal.
### Operational checkpoints for shutdown work
A disciplined shutdown sequence reduces avoidable faults. An administrator should confirm four conditions before the event starts.

- Critical users have notice
- Long-running jobs have either completed or been rescheduled
- New logins are blocked at the intended point
- The shutdown can be cancelled cleanly if the maintenance window shifts

This matters because a shutdown problem is rarely just a power event. It often becomes a service problem, a user access problem and a communication problem at the same time. RHEL gives enough tooling to manage all three without extra software.
### Why label repair matters
Password recovery fails most often when the administrator treats the new password as the final step. In RHEL, the password change only restores the credential. It does not restore the SELinux metadata that normal authentication expects. A host may therefore accept the new password at one point in the boot cycle and still fail later when `sudo`, `su` or other authentication paths touch the mislabeled shadow file. `restorecon` is therefore part of the recovery procedure, not an optional cleanup task.
### Unit administration as a layered model
Systemd works best when units are treated as layers rather than isolated files. Vendor definitions in `/usr/lib/systemd/system` describe the packaged service. Local overrides in `/etc/systemd/system` adapt that service to site policy. Symlinks inside target directories decide whether the unit starts automatically. Runtime state then decides whether the service is loaded, active, failed or inactive. `systemctl` exposes all four layers through a single interface, which is why it replaces several older service tools so effectively.
### Reading load without guesswork
Load figures should never be read in isolation. A load average of `2` can mean serious pressure on a single-core system and very little pressure on an eight-core system. Logged-in user count matters as well because high concurrency can explain rising load without any fault in the service stack. That is why `uptime`, `lscpu` and `top` work best as a set. One command shows trend, one shows hardware context and one shows the active consumers.
### Choosing between text logs and the journal
Traditional text logs remain useful because they are simple to tail, rotate and export. The journal remains useful because it keeps unit metadata, time filters and boot history in one place. RHEL does not force a choice between them. It expects administrators to use both. Text logs are often best for long-running follow mode and external tooling. The journal is often best for precise filtering, especially when the exact log file is unknown.