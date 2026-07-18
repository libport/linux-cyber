# Analyzing System and Application Performance
> [!NOTE]
> Presents an evidence-led approach to diagnosing system and application performance by combining requirements analysis, targeted monitoring, dynamic tracing, code profiling, historical metrics, and disciplined operational practice.

System and application performance analysis starts with a clear understanding of requirements, workload and risk. Administrators make better decisions when they compare applications against measured behaviour rather than vendor claims, release dates or assumptions about newer versions. The best choice may be the latest release, an older release, a competing tool or a redesign of the environment around the application.

A useful evaluation records the operating system version, patch level, CPU, memory, storage and network requirements. It also identifies disk I/O, network I/O, scaling behaviour and dependencies such as shared storage, firewalls, load balancers and VPNs. These details reveal hidden costs and operational risks before deployment. They also define the metrics that matter during testing.

Performance testing should answer practical questions:
- Does the current infrastructure meet the minimum and recommended requirements?
- Does the workload generate heavy disk or network traffic?
- Does the application scale by adding servers, adding resources to existing systems or changing the architecture?
- Do maintenance tasks control log growth, cache growth, storage usage and temporary files?
- Does the measured performance meet user, business and support expectations?

Administrators should test applications under representative workloads and collect repeatable measurements. A controlled comparison helps separate application faults from infrastructure limits. It also helps explain performance findings to developers, vendors, managers and customers.
## Common utilities for process and resource monitoring
General Linux utilities give administrators fast visibility into processes and resources. They suit early investigation, routine checks and lightweight monitoring.

`ps` displays a point-in-time process snapshot. Administrators can select specific columns, filter by process name and retain headers in the output for readability. Combining `ps` with `watch` refreshes the snapshot on a chosen interval, which helps track process growth, CPU use or memory use during a short investigation.

`top` provides a dynamic process view with system summaries for uptime, CPU and memory. It can focus on a specific process ID and can run in batch mode, which makes it useful for writing repeated samples to a file. `htop` gives a more interactive view. It supports horizontal and vertical scrolling, mouse interaction, easier sorting and process selection.

`nmon` monitors performance interactively or records data for later analysis. It can display selected metrics on screen and can write detailed samples to a log file. Administrators often review recorded data with charting or visualisation tools rather than reading the raw file directly.

`glances` presents many metrics in one terminal view, including CPU, memory, swap, disk I/O, network activity and process data. It can run in client-server mode, expose a web interface and export metrics to systems such as InfluxDB. These features make it useful when administrators need a broad view or a simple remote monitoring option.

Common tools remain valuable because they are quick, familiar and widely available. Their main limitation is that they usually provide either a broad view or a short-term view. Detailed troubleshooting often needs more focused utilities.
## Specialised utilities for focused metrics
Focused utilities isolate particular performance areas. They give administrators a better view of CPU scheduling, storage latency, process activity, memory pressure and historical trends.

The `sysstat` suite includes tools such as `mpstat`, `iostat`, `pidstat`, `sar`, `sadf` and `sa`. The original text grouped `vmstat` with `sysstat`, but `vmstat` normally comes from the `procps-ng` family of tools. This distinction matters when administrators install packages, document procedures or build troubleshooting runbooks.

`mpstat` reports processor statistics. A default run gives a point-in-time CPU summary, while interval parameters repeat the sample and add an average. Per-CPU views show whether work spreads evenly or concentrates on a specific processor. Interrupt views help identify peripheral or driver behaviour that may affect performance.

`iostat` reports CPU and device I/O statistics. Extended output adds queue, request and utilisation details, which helps identify heavy reads, heavy writes and possible storage bottlenecks. It is especially useful when storage sits behind RAID, SAN or virtualised layers because those layers can hide latency until workload increases.

`vmstat` reports processes, memory, paging, block I/O, disk and CPU activity. Timestamped interval output supports logging and later review. It helps administrators connect memory pressure, swap activity and I/O behaviour in one compact display.

`pidstat` reports per-process statistics. It can show CPU usage, page faults, memory usage, I/O activity, child processes, threads and command paths. Filtering by process name or process ID makes it suitable for monitoring one service while a workload runs.

`sar` collects and reports system activity. It can display CPU, memory, paging, filesystem, network, block device and other metrics from current samples or log files. Its value increases when the system already collects data through scheduled activity, because administrators can review the state of a system before an incident was reported.

`sadf` reads `sar` data and exports it in formats that other tools can process. It can produce formats such as JSON, XML, SVG and database-friendly separated output. When a CSV file is required, administrators can post-process the separated output into comma-separated data. This workflow turns historical measurements into graphs, spreadsheets and reports.

These utilities overlap by design. Overlap gives administrators several ways to confirm a finding. If multiple tools show the same pattern, the evidence becomes stronger.
## Valgrind for application behaviour
System tools can show that an application consumes CPU, leaks memory, performs excessive I/O or handles cache poorly. They usually cannot explain the application code path that caused the behaviour. Valgrind fills part of that gap for compiled programs by providing debugging and profiling tools.

The original text described Valgrind as having six utilities. The corrected view is that the Valgrind distribution contains a modular suite with several tools. The six tools covered here are commonly used for memory, cache, threading and call analysis.

`memcheck` detects memory management errors. It can report leaks, invalid reads and writes, uninitialised values and other memory faults. Administrators may not fix the code directly, but they can give developers or vendors a clear report that shows the application has a memory problem.

`cachegrind` profiles cache behaviour. It simulates instruction and data caches and reports cache misses. On modern systems it reports first-level and last-level cache information, rather than only a simple level 1 and level 2 model. This detail helps developers find functions or source lines that cause poor cache locality.

`helgrind` and `drd` detect threading and synchronisation errors in multi-threaded programs that use POSIX threading primitives or related abstractions. They can report possible data races, conflicting loads and conflicting stores. These findings help developers investigate intermittent errors that are hard to reproduce from normal system monitoring.

`massif` profiles heap usage and can also profile stack usage when enabled. It writes snapshots that show memory consumption over time. `ms_print` renders those snapshots into a readable report with a graph, allocation details and allocation trees. This helps identify which code paths allocate memory and when growth occurs.

`callgrind` records function call relationships and execution costs. `callgrind_annotate` makes the output easier to read by summarising events and relating them to files and functions. It helps developers focus optimisation work on the functions that matter most.

Valgrind analysis can slow applications and should run in development, test or controlled diagnostic environments. Its strength lies in producing code-level evidence that complements system-level measurements.
## SystemTap for dynamic tracing
SystemTap gives administrators a way to trace kernel and user-space activity without rebuilding and rebooting the kernel. It uses scripts that define events and handlers. When an event occurs, the handler records or prints the selected data.

A working SystemTap environment needs the relevant packages and kernel support files. `stap-prep` can prepare many systems by installing required kernel debuginfo and development packages. When automated preparation fails, administrators should match the kernel version and architecture carefully before installing debug packages manually.

A simple SystemTap command can confirm that the tool can compile and run probes. Verbose output shows the passes that check scripts, translate them into C, compile temporary code and run the resulting module. This workflow explains why scripts take a few moments to start.

SystemTap scripts can monitor many behaviours:
- Disk read and write activity by process
- Event counts and rates, such as read system calls
- Network data sent and received by process
- TCP connection activity
- Process creation, termination, forks and module events
- Terminal input activity in controlled diagnostic scenarios

Scripts such as `disktop.stp`, `eventcount.stp`, `nettop.stp`, `tcp_connections.stp` and `procmod_watcher.stp` show how administrators can collect focused evidence without writing a full monitoring application. Because the scripts are text files, administrators can inspect and adjust intervals, filters and output limits.

SystemTap is powerful and should run with care. Scripts can expose sensitive activity or affect busy systems if written poorly. Administrators should test scripts in a sandbox or non-production environment, review the code and limit collection to the data required for the investigation.
## eBPF and BCC tracing tools
eBPF is a Linux kernel technology that runs verified, sandboxed programs in privileged contexts. It is no longer treated as an acronym for Extended Berkeley Packet Filter. Administrators use front ends such as BCC tools to trace system behaviour without changing kernel source code or loading traditional kernel modules.

The BCC tools package provides many ready-made commands under paths such as `/usr/share/bcc/tools` on many Red Hat systems. These tools capture kernel and application events with focused output.

`execsnoop` traces new processes created through `exec` system calls. It can add timestamps and user IDs, and it can filter by executable name. This helps administrators find who launched a command, when it ran, which arguments it used and whether a process repeatedly spawned.

`opensnoop` traces file open calls. It reports process IDs, commands, file descriptors, errors and paths. User filtering helps show which files a specific account or process touches. This is useful when an application fails because it cannot open a library, configuration file or data path.

`ttysnoop` attaches to a terminal session and provides a read-only view of activity. It can help support staff observe a user session during troubleshooting. It also raises privacy and governance concerns, so administrators should use it only under approved operational processes.

`tcplife` reports TCP sessions that open and close while the tool runs. It shows process details, local and remote addresses, transferred bytes and session lifespan. Port and process filters help isolate a web service, agent or application.

`killsnoop` traces kill system calls and reports the source process, target process, signal and result. It helps investigate services that terminate unexpectedly or scripts that kill the wrong process.

`gethostlatency` traces slow name lookups through hostname resolution calls. The original text used `githostlatency`, which is incorrect. The corrected tool helps identify DNS latency by reporting the process, latency and host name or address involved in lookups.

`biotop` works like a top-style view for block device I/O. It reports the responsible process where available, I/O direction, device, request count, throughput and average request time. Adjusting the interval helps avoid empty displays on quiet systems and excessive output on busy systems.

eBPF and BCC tools provide detailed, low-level observability. They still require privilege, care and context. Administrators should collect only the event stream needed for the problem and stop tracing once evidence is sufficient.
## Performance Co-Pilot for centralised metrics
Performance Co-Pilot, or PCP, provides tools, services and libraries for collecting, storing, visualising and analysing system-level metrics. It can monitor local and remote systems, review live metrics and replay historical archives. This makes PCP useful for both incident investigation and capacity planning.

A minimal setup often starts with `pcp` and `pcp-zeroconf`. Core services include the Performance Metrics Collector Daemon, `pmcd`, the archive logger, `pmlogger`, and the Performance Metrics Inference Engine, `pmie`. Administrators should verify that these services are enabled and running before relying on PCP data.

`pmlogconf` provides an interactive method for configuring archive logging. This is safer than editing logger configuration files directly unless the administrator already understands the file format. Log configuration commonly sits under `/var/lib/pcp/pmlogger`, while archive data commonly sits under `/var/log/pcp/pmlogger`.

PCP can operate as a central monitoring service. A central host can collect logs for remote systems, while each remote system runs the required PCP services and listens on the configured port. Administrators should configure firewall rules, SELinux settings and service bindings deliberately. They should also confirm archive creation by checking the remote host directory and logger files.

PCP includes tools that resemble older utilities and tools that are more specific to PCP. The legacy-style tools help administrators transition from familiar commands:
- `pcp-iostat` provides disk I/O statistics through PCP metrics.
- `pcp-vmstat` presents a vmstat-like view from PCP data.
- `pcp-mpstat` provides processor statistics with access to PCP archives when used with suitable options.
- `pcp-atop` gives a top-like interactive view with system, process, thread, disk, memory, network and container perspectives.
- `pmchart` presents live or archived metrics in a graphical interface when a desktop environment is available.

PCP also includes tools for metric discovery, reporting and archive analysis. `pmdumptext` can combine selected metrics in one live or archived view, such as CPU load, memory use and disk writes. `pminfo` reports metric information at the time of execution. `pmprobe` checks metric instances, such as active network interfaces or container interfaces. `pmdumplog` reports metadata about an archive, including source, time range and state information. `pmlogextract` combines or extracts archive data for later review. `pmrep` reports selected metrics to standard output and can target specific time windows.

PCP differs from tools that rely only on ad hoc commands or scheduled snapshots. Its services collect metrics consistently, its archives preserve history and its tools can replay the same data through terminal and graphical interfaces. These features support root cause analysis, trend review and comparison across hosts.
## Selecting tools by evidence pattern
Different symptoms require different evidence. Administrators should avoid treating one favourite command as a universal answer. A process that consumes CPU, a storage device that queues requests, a memory leak that appears after hours and a service that fails during name resolution all need different observations. The strongest investigation connects the user symptom to a measurable resource and then to the responsible process, configuration or code path.

High CPU use needs both summary and per-process evidence. `top`, `htop` and `pcp-atop` quickly show whether the whole system or one process drives the load. `mpstat` and `pcp-mpstat` then show whether the load affects all CPUs or only a subset. If CPU time concentrates in system time or interrupt handling, administrators should inspect interrupts, drivers, storage or network activity before blaming the application. If CPU time sits mostly in user space, per-process tools and application profiling become more relevant.

Memory pressure needs a distinction between allocation, use and paging. `vmstat` shows free memory, swap activity and paging behaviour in a compact form. `pidstat` shows memory use and page faults for a selected process. PCP archives help confirm whether growth is steady, periodic or tied to a workload. If memory grows without returning to a stable level, Valgrind `memcheck` or `massif` can help developers confirm leaks or allocation patterns in compiled applications.

Storage problems need latency and throughput context. `iostat -x`, `biotop`, `pcp-iostat` and PCP archive views help separate high request volume from slow service time. A system can have many reads and writes without a fault if devices handle them quickly. A smaller workload can still hurt performance when queue length, await time or average I/O time rises. Administrators should compare results with the storage design, including local disks, virtual disks, RAID, SAN, network storage and cloud volumes.

Network problems need traffic, connection and name resolution data. General tools can show interface throughput and TCP activity. SystemTap scripts can show process-level send and receive counts. `tcplife` shows TCP sessions that open and close while tracing runs. `gethostlatency` helps identify slow DNS lookups. These details matter because an application that appears slow may spend time waiting for name resolution, external services or short-lived network connections rather than local CPU or disk.

Process churn needs tracing rather than periodic sampling. A short-lived command may start and exit between `top` or `ps` samples. `execsnoop` captures new `exec` calls in real time, while process watcher scripts can show forks, exits and module activity. This evidence helps find scripts, agents or daemons that repeatedly start child processes and create bursts of CPU, I/O or log activity.

File access faults need path-level evidence. `opensnoop` shows which process tries to open which file and whether the attempt succeeds. This can reveal missing configuration files, wrong permissions, broken library paths and unexpected file locations. It also helps during application upgrades, when a new version may read different paths from an older version.

Threading faults need application-level tools. System metrics may show stalls, high CPU or uneven throughput, but they rarely prove a race condition. Valgrind `helgrind` and `drd` can report data races and conflicting access patterns. Administrators should treat those reports as diagnostic evidence for developers, not as a complete fix. The source code owner still needs to confirm the cause and test the correction.

Cache and call-path issues need profiling. `cachegrind` and `callgrind` help developers see cache misses, instruction counts and function relationships. Administrators can use this information to support a performance case when ordinary monitoring shows that hardware is healthy but one application still runs poorly.

Historical incidents need archives. A problem reported after the fact cannot be reconstructed from a current `top` screen. `sar` logs and PCP archives provide the past state of CPU, memory, disk, network and other metrics. This historical context helps decide whether a problem was a one-off spike, a daily pattern, a capacity trend or a change introduced by a deployment.
## Key terminology and package boundaries
Accurate naming prevents troubleshooting errors. `sysstat` is a performance monitoring suite that centres on tools such as `sar`, `sadf`, `mpstat`, `iostat` and `pidstat`. `procps-ng` provides several common process and system utilities, including `ps`, `top`, `watch` and `vmstat`. These packages often coexist on Red Hat systems, but they serve different installation and documentation paths. Runbooks should name the correct package so administrators can install missing tools quickly during an incident.

eBPF also needs precise wording. It began from Berkeley Packet Filter work, but current usage treats eBPF as a kernel technology rather than a simple acronym. The technology allows verified programs to run in privileged contexts. Tools such as BCC and bpftrace provide user-facing ways to write or run those programs. The ready-made BCC commands are tools built on eBPF, not eBPF itself.

Performance Co-Pilot names also deserve care. PCP refers to the whole framework of services, libraries, metrics, archives and tools. `pmcd` collects and serves metrics. `pmlogger` records archives. `pmie` evaluates metric rules. Tools such as `pmdumptext`, `pmrep`, `pmchart`, `pminfo` and `pmprobe` query, display or report those metrics. This separation helps administrators diagnose whether a problem lies in collection, logging, archive access or display.

Valgrind terminology should distinguish the framework from its individual tools. Valgrind provides a core execution environment and multiple debugging or profiling tools. `memcheck` focuses on memory correctness. `massif` focuses on heap use. `cachegrind` and `callgrind` focus on cache and call behaviour. `helgrind` and `drd` focus on thread correctness. Treating these as separate tools helps teams choose the right test and avoids overloading a memory leak investigation with unrelated profiling output.

Command names need the same discipline. `pidstat`, `gethostlatency`, `pmdumptext`, `pcp-mpstat` and `opensnoop` should be written exactly. Small spelling errors waste time, especially when an administrator copies a command from a procedure during an outage. Paths also matter. PCP logs commonly live below `/var/log/pcp`, while configuration and runtime details can live elsewhere. BCC tools commonly appear below `/usr/share/bcc/tools` on many distributions. Local packaging can vary, so procedures should include a verification step rather than assuming one path fits every host.
## Operational discipline for performance work
Performance analysis can disrupt systems when administrators collect too much data, trace sensitive activity or run heavy profilers in production. A disciplined approach limits scope before tools run. Administrators should know the affected host, workload, time window, expected behaviour and business impact. They should also know whether the investigation needs live tracing, historical data or controlled reproduction.

Safe investigation starts with low-impact observation. Broad commands such as `ps`, `top`, `vmstat`, `iostat`, `mpstat` and PCP summaries usually create little overhead. Tracing and profiling tools need more care. SystemTap and eBPF tools can expose sensitive information, including file paths, commands, network endpoints and terminal activity. Valgrind can slow an application significantly. These tools belong in production only when the risk is understood, approved and limited.

Evidence should remain reproducible. Administrators should record the command, options, sample interval, host, time zone and time window. They should keep raw logs when practical and summarise the key findings in plain language. A good summary states what happened, when it happened, what resource changed, which process or component appears responsible and what evidence supports that conclusion.

Tool output should lead to an action. A disk queue points to storage review, workload scheduling or application I/O changes. Repeated process creation points to scripts, services or automation. DNS latency points to resolver configuration, network path or upstream service review. Memory leaks and threading reports point to developers or vendors. PCP and `sar` trends point to capacity planning and change management.

Administrators should separate symptoms from causes. High load average does not automatically mean CPU saturation. Low free memory does not automatically mean a fault on Linux systems that use memory for cache. High network traffic may be expected during backup, replication or deployment. A process that appears in a trace may be the caller, the victim or merely a related component. Good analysis checks each interpretation against multiple metrics.

Communication matters as much as collection. Developers need command output, affected versions, reproduction steps and concise technical findings. Managers need risk, impact and options. Customers need clear status without unnecessary internal detail. Strong performance work translates raw measurements into decisions about tuning, scaling, version choice, defect repair or architecture.
## Practical monitoring strategy
Effective performance analysis uses layers. Administrators should start with the question they need to answer, then choose the lightest tool that can answer it accurately.

A practical sequence works well:
1. Define the expected behaviour and the affected workload.
2. Check broad system health with common utilities.
3. Use focused tools to isolate CPU, memory, storage, network or process behaviour.
4. Use tracing tools when ordinary metrics do not explain the cause.
5. Use application profiling tools when the evidence points to code behaviour.
6. Preserve historical data through PCP or `sar` archives where ongoing review matters.
7. Share concise findings with the team that can act on them.

Administrators should avoid collecting every metric by default. Too much data can obscure the fault, increase system load and create privacy risks. Good monitoring captures the right evidence at the right time and keeps the result understandable.

The tools covered here fit different stages of the same discipline. Common utilities reveal symptoms quickly. Specialised utilities quantify the affected resource. Valgrind explains application-level faults. SystemTap and eBPF expose dynamic kernel and process behaviour. PCP centralises collection and turns live observations into historical evidence. Together, they support informed decisions about application versions, system tuning, troubleshooting and long-term capacity planning.