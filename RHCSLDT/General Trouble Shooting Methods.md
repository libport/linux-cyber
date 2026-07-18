# General Trouble Shooting Methods
> [!NOTE]
> Presents a disciplined RHEL troubleshooting framework that combines evidence gathering, authoritative documentation, performance monitoring, log analysis, and centralised logging to diagnose faults and implement persistent, verifiable fixes.

Red Hat EX342 candidates need a reliable method for diagnosing Linux faults, collecting evidence, using documentation, monitoring systems and forwarding logs to a central host. Red Hat identifies RHEL 8.4 as the base for EX342 and RH342, so practice should use a RHEL 8.4 system wherever possible. A Red Hat Developer account gives individual learners a no-cost path to RHEL access for study and lab work, subject to Red Hat's terms.

The general troubleshooting objective covers four capabilities:
- Collect system information that narrows the fault domain.
- Consult documentation from Red Hat and from the local command line.
- Monitor key system characteristics with standard tools and Performance Co-Pilot.
- Configure systems to send log messages to a central logging host.
## Troubleshooting Workflow
Effective troubleshooting starts with control, not speed. Administrators should pause, reduce distractions and approach the system with a repeatable process. A clear workflow prevents random changes and makes each test meaningful.

A practical sequence is:
- Review the reported symptoms and the conditions in which they appear.
- Reproduce the problem when it is safe to do so.
- Gather system, process, package, service and log evidence.
- Form a hypothesis that fits the evidence.
- Apply one targeted fix or test.
- Reproduce the original problem again to verify the result.
- Repeat the cycle when the first hypothesis fails.

The aim is to resolve the cause, not merely suppress the symptom. Administrators should record the command run, the file changed and the observed result, especially during an exam or an outage. Failed hypotheses still help because they remove possibilities and refine the next test. A stable method also reduces the risk of introducing a second fault while chasing the first one.
## Gathering System Information
Basic system commands give the first view of load, storage, memory, open files and active connections. `uptime` reports how long the host has been running, how many users are connected and the 1, 5 and 15 minute load averages. Load averages should be read beside the CPU count because a value that is harmless on a many core host may matter on a small one. `df -h` reports file system usage in readable units and helps detect full or missing mounts. A full root, `/var` or application file system can stop services, block logging and prevent package operations. `free -h` reports physical and swap memory in readable units. Use `free -m` for megabytes or `free -g` for gigabytes.

Open files can expose resource exhaustion or unexpected activity. Linux treats many resources as files, so `lsof` can show regular files, libraries, devices and sockets held by processes. `lsof -u username | wc -l` counts open files for one user. Compare that count with the user's limit by running a command such as `su - username -c 'ulimit -n'`. When the count sits far below the soft limit, open file exhaustion becomes less likely. `lsof -i` lists open network connections and can reveal suspicious or unexpected sockets.

Process discovery adds context to the raw resource figures. `top` provides a live view of load, CPU usage, memory usage and running processes. It suits an initial scan because the display updates continuously and exposes sudden spikes. `ss -lpt` shows listening TCP sockets and the processes attached to them. Its output helps confirm whether an expected service is listening on the correct port, or whether an unexpected process has opened one. `ps` lists processes for the current shell, while `ps aux` shows a fuller BSD style process list that includes user, process ID, CPU, memory, start time and command. `ps ajfx` displays a process tree, which helps trace parent and child relationships. `ps aux --sort=-pcpu` sorts processes by CPU use. Administrators can inspect the `ps` manual page for other sort keys and GNU style alternatives.
## Documentation and Package Discovery
Good troubleshooting depends on fast access to reliable documentation. Red Hat documentation lives in the Red Hat Customer Portal. A no-cost Red Hat Developer Subscription for Individuals can provide learning access to RHEL and related Red Hat resources. Systems with a paid subscription may also use `redhat-support-tool` from the command line, although EX342 candidates can succeed without it.

Local documentation often answers command questions more quickly than a browser. Most commands support `--help`, which gives syntax, options and examples. Manual pages provide deeper information for commands, configuration files, system calls and administrative tools. Manual page sections matter because the same term can describe a command, a file format or a library call. For example, an rsyslog configuration file and the rsyslog daemon appear in different sections. `man -k keyword` searches manual page descriptions. `mandb` refreshes the manual page database. Inside a manual page, `/term` searches for a term such as `OPTIONS`, which prevents slow scrolling through long reference pages.

Package tools connect commands, files and packages. `rpm -q package` shows whether a package is installed. `rpm -qf /path/to/file` identifies the installed package that owns a file. `rpm -ql package` lists files installed by a package. Documentation and examples commonly live under `/usr/share/doc`. `find /usr/share/doc -type d -iname '*example*'` can locate example directories.

RHEL 8 uses DNF as the backend, while the `yum` command remains available for compatibility. `yum search httpd` or `dnf search httpd` searches enabled repositories. `yum whatprovides '*/pip3'` or `dnf provides '*/pip3'` identifies packages that provide a command or file. `repoquery --list package` lists package contents from repositories and `repoquery --requires package` lists dependencies.
## Monitoring with Performance Co-Pilot
Performance Co-Pilot, or PCP, collects, stores, visualises and analyses system level performance metrics. Install the core tools with packages such as `pcp` and `pcp-system-tools`. Install `pcp-doc` when local PCP manual pages are needed.

PCP centres on `pmcd`, the Performance Metric Collector Daemon. Performance Metric Domain Agents, or PMDAs, collect metrics and supply them to `pmcd`. Client tools then retrieve, report or archive those metrics on the local host or across the network. This architecture separates collection from analysis, which lets one set of metrics support live viewing, reports and historical archives. Start and enable the service with `systemctl enable --now pmcd`.

Historical data requires `pmlogger`. Start and enable it with `systemctl enable --now pmlogger`. By default, archives are stored below `/var/log/pcp/pmlogger/hostname`. Archived metrics allow retrospective analysis after a fault has passed. They also help separate short lived incidents from long term trends, such as steady memory growth, recurring CPU saturation or storage pressure that appears at the same time each day.

Useful PCP tools include:
- `pmstat`, which gives a high level performance line every five seconds by default.
- `pcp atop`, which gives a live, top like view backed by PCP metrics.
- `pminfo`, which lists metric names.
- `pminfo -t`, which adds brief metric descriptions.
- `pmval`, which dumps values for selected metrics.
- `pmrep`, which reports selected metrics in a more readable format.

Common PCP options work across several tools. `-t` sets the sample interval. `-T` sets a finish time or duration, depending on context. `-S` sets a start time. `-a` reads an archive. Timestamps must match a period when the archive contains data. A metric such as `mem.util.used` can be queried with `pmval` or `pmrep` against live data or a selected archive. `pmval` gives a value oriented dump, while `pmrep` presents report output that is usually easier to read in a terminal.
## Logging with rsyslog and journald
RHEL systems commonly use both `rsyslog` and `systemd-journald`. Rsyslog writes configured facilities and priorities to files or remote destinations. A facility identifies the subsystem that produced the message. A priority identifies severity, from emergency through debugging. Configuration rules decide which messages rsyslog stores and where it sends them. Selectors such as `mail.*` match a facility and priority pattern. Rules can send messages to files, remote hosts or logged in users through actions such as `:omusrmsg:*` for emergency broadcasts.

`systemd-journald` records structured journal entries from the kernel, services and other system components. `journalctl` displays the journal from the oldest available entry for the calling user. `journalctl -ef` jumps to the end and follows new entries. `journalctl -k` filters kernel messages. `journalctl -u sshd` filters a unit. `journalctl -p warning` filters by priority. `journalctl -xb` shows entries from the current boot. `--since` and `--until` restrict output to a time window. These filters should be combined with service restarts and reproduction tests so the relevant entries appear close together.

Journal storage needs careful handling. With volatile storage, journal entries disappear after a restart. To make them persistent on RHEL 8, set `Storage=persistent` in `/etc/systemd/journald.conf`, ensure `/var/log/journal` exists and restart `systemd-journald`. Use ownership such as `root:systemd-journal` and appropriate permissions, such as `2755`, where required by the local configuration.
## Centralised Remote Logging
Remote logging needs at least one sender and one receiving log host. On the receiving host, configure `/etc/rsyslog.conf` or a file under `/etc/rsyslog.d/` to load TCP input, listen on port 514 and write incoming messages to the chosen destination. A template can place messages into files named for each source host, for example below `/var/log/hosts/`.

On each client, configure an rsyslog forwarding rule that targets the logging server by hostname or private IP address and port 514. Storing that rule in `/etc/rsyslog.d/` keeps the configuration modular and easier to remove after a lab. Restart `rsyslog` on the receiver and sender after changes, then check service status for syntax errors. Firewalls and routing must allow the selected transport. Test forwarding with `logger 'test message'`, then confirm that the expected host log file contains the entry.

Central logging makes troubleshooting stronger because administrators can compare events across hosts, preserve evidence when a client fails and review historical logs from a single place.
## Practical Exam Discipline
Configurations should persist after reboot because Red Hat performance based exams assess the final system state. Administrators should prefer changes in the correct configuration file over temporary shell state, restart or reload the affected service, then confirm the result with an independent command. For example, a logging change should be followed by `systemctl status rsyslog`, a fresh `logger` message and a check of the destination log. A monitoring change should be followed by `systemctl status pmcd`, `systemctl status pmlogger` and a small query against live or archived metrics.

Evidence should guide every repair. Full disks, failed units, missing mounts, exhausted limits, listening ports and recent journal entries all point to different fault classes. A concise note of the symptom, command output and fix keeps the investigation coherent and makes it easier to hand findings to another administrator when the issue needs deeper support.