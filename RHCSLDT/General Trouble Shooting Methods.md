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