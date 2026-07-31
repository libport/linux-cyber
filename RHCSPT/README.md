# *Red Hat Certified Specialist in Performance Tuning* Course Notes
*v1.0 stable release*

These notes cover material from Pluralsight's 12 hour, self-paced [Red Hat Certified Specialist in Performance Tuning](https://www.pluralsight.com/courses/red-hat-certified-specialist-in-performance-tuning-ex442) course. They cover how to analyze the performance of a Red Hat Enterprise Linux system, build and validate your knowledge of these tools, and update and tune the performance of a Red Hat Enterprise Linux system and the applications hosted on it.
## Performance tuning method
Effective tuning begins with a defined workload and a measurable objective. Throughput, latency, response time, resource efficiency, power use, and recovery time describe different outcomes. Improving one can degrade another. A change that increases bulk throughput can increase tail latency, while aggressive caching can improve read performance at the cost of memory pressure.

An administrator should use a controlled cycle:
1. Define the workload, service-level target, test duration, and acceptable risk.
2. Record hardware, software, topology, configuration, and workload versions.
3. Establish a baseline under representative load.
4. Locate the constrained resource and form a testable hypothesis.
5. Change one relevant control at a time.
6. Repeat the same workload and compare the same metrics.
7. Check for regressions outside the target metric.
8. Retain a successful change in a persistent configuration, or restore the baseline.
9. Reboot when required and verify the configuration and result again.

Measurements need context. CPU utilisation without run-queue length, I/O wait, interrupt activity, and workload throughput can mislead. Free memory alone says little because Linux uses otherwise idle memory for caches. A disk utilisation value needs queue depth, latency, request size, and device characteristics. Short averages can also hide bursts and high-percentile latency.

Application selection follows the same discipline. Administrators should compare supported versions with identical data, load, hardware, configuration, and test duration. Selection should account for CPU, memory, storage, network traffic, dependencies, support life, operational complexity, scaling, and failure behaviour. The newest version is not automatically the best performer for a specific workload.

Tests should run in a non-production environment when tools add significant overhead or when changes can destabilise the host. Production investigation should begin with low-overhead observation. Administrators should record commands, timestamps, output, and rollback steps before changing a live service.
## Kernel behaviour
### Boot-time and runtime parameters
Kernel command-line parameters take effect during boot. Runtime parameters exposed through `sysctl`, `/proc/sys/`, and `/sys/` control behaviour after the kernel starts.

The running command line appears in:

```text
/proc/cmdline
```

The following commands show the running arguments, installed boot entries, and kernel messages that recorded the command line:

```bash
cat /proc/cmdline
grubby --info=ALL
dmesg | grep -i 'command line'
```

`/proc/cmdline` remains the direct source for the current boot. The relevant `dmesg` entry can disappear if the ring buffer wraps or an administrator clears it.

RHEL 10 stores Boot Loader Specification entries under `/boot/loader/entries/`. Administrators should normally use `grubby` instead of editing generated GRUB files. The following command adds an argument to all installed kernel entries:

```bash
grubby --update-kernel=ALL --args="parameter=value"
```

The matching removal uses:

```bash
grubby --update-kernel=ALL --remove-args="parameter=value"
```

The change requires a reboot. `grubby --info=ALL` and `/proc/cmdline` verify the stored and active states.

`sysctl -a` lists runtime parameters. A key can be read through `sysctl` or the corresponding file under `/proc/sys/`:

```bash
sysctl net.ipv4.ip_forward
sysctl -n net.ipv4.ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

Dots in a `sysctl` key map to directory separators in `/proc/sys/`. A temporary change applies immediately:

```bash
sysctl -w net.ipv4.ip_forward=1
```

Writing directly to `/proc/sys/net/ipv4/ip_forward` has the same runtime effect. Neither method persists after a reboot.

Site-specific persistent settings belong in a clearly named file under `/etc/sysctl.d/`, not in `/usr/lib/sysctl.d/`. Vendor packages own `/usr/lib/sysctl.d/`, while `/etc/sysctl.d/` supplies administrator policy and overrides. For example:

```ini
# /etc/sysctl.d/60-network-forwarding.conf
net.ipv4.ip_forward = 1
```

The following command loads and checks the file without waiting for a reboot:

```bash
sysctl -p /etc/sysctl.d/60-network-forwarding.conf
sysctl net.ipv4.ip_forward
```

`sysctl --system` loads all system configuration files in precedence order. File names should reflect their purpose, and higher-numbered files can override earlier values. Administrators should check existing definitions before adding a duplicate key:

```bash
grep -R "net.ipv4.ip_forward" /etc/sysctl.d /run/sysctl.d /usr/local/lib/sysctl.d /usr/lib/sysctl.d
```

Kernel settings can exhaust memory, prevent boot, weaken isolation, or disrupt networking. A runtime test, a console or recovery path, and a rollback command reduce that risk.
### Kernel modules
Kernel modules extend the running kernel. The principal inspection and runtime commands are:

```bash
lsmod
modinfo module_name
modprobe module_name
modprobe -r module_name
```

`lsmod` shows loaded modules, their sizes, and their use counts. `modinfo` shows the module file, dependencies, aliases, signatures, and supported parameters. `modprobe` resolves dependencies when loading or unloading a module. Removing a module that a device, file system, or another module still uses can make the system unstable.

A module parameter can change driver or subsystem behaviour. The following commands list supported parameters and show active values when the module exposes them:

```bash
modinfo -p module_name
find /sys/module/module_name/parameters -maxdepth 1 -type f -print
cat /sys/module/module_name/parameters/parameter_name
```

Some parameters permit a runtime write through sysfs. Others apply only when the module loads. A safe load-time test unloads an unused module and supplies the value during reload:

```bash
modprobe -r module_name
modprobe module_name parameter_name=value
```

Persistent load options belong under `/etc/modprobe.d/`:

```text
# /etc/modprobe.d/module_name.conf
options module_name parameter_name=value
```

The administrator should verify the active value after reloading or rebooting. A built-in driver is not a loadable module and can require a kernel command-line parameter instead.

A runtime `modprobe` change does not persist. A module that must load during every boot needs a file under `/etc/modules-load.d/`:

```text
# /etc/modules-load.d/module_name.conf
module_name
```

A denylist file under `/etc/modprobe.d/` prevents automatic loading and also blocks an explicit load when it includes both directives:

```text
# /etc/modprobe.d/denylist.conf
blacklist module_name
install module_name /bin/false
```

`blacklist` alone does not stop another module from loading the target as a dependency. The `install` directive substitutes `/bin/false` for the load operation. Administrators must confirm dependencies and service requirements before denying a module.

If the module appears in the initial RAM disk, the administrator should back up the current image, rebuild it, and reboot:

```bash
cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut -f -v
reboot
```

After the reboot, `lsmod`, `systemctl status`, and relevant device checks confirm the outcome. A module required for storage, encryption, networking, or the root file system must not be denied without a tested recovery route.
## System and application measurement
### Common command-line tools
RHEL 10 provides core tools through `procps-ng`, `sysstat`, `perf`, and related packages:

```bash
dnf install procps-ng sysstat perf
```

Each tool answers a different question.

| Tool | Primary use |
| --- | --- |
| `ps` | Point-in-time process selection and reporting |
| `top` | Interactive process and host activity |
| `vmstat` | Run queues, memory, paging, block I/O, interrupts, context switches, and CPU |
| `mpstat` | Aggregate and per-CPU utilisation, interrupts, and soft interrupts |
| `iostat` | CPU and block-device throughput, queues, and latency |
| `pidstat` | Per-process CPU, memory, faults, I/O, threads, and context switches |
| `sar` | Current or archived system activity from `sysstat` |
| `sadf` | Conversion of `sar` data to machine-readable formats |
| `perf` | Hardware counters, software events, tracepoints, sampling, and call graphs |

`ps` supports precise fields and stable sorting:

```bash
ps -eo user,pid,ppid,ni,cls,psr,%cpu,%mem,stat,comm,args --sort=-%cpu
ps -C sshd -o pid,ppid,ni,psr,%cpu,%mem,cmd
```

`watch` can repeat a snapshot, although purpose-built interval tools usually preserve timing and headings more cleanly:

```bash
watch -n 2 'ps -C application -o pid,ni,psr,%cpu,%mem,cmd'
```

`top` provides live sorting, filtering, thread display, per-CPU summaries, and batch output. Batch mode supports collection without an interactive terminal:

```bash
top -b -d 2 -n 10 -p PID > top.log
```

`vmstat 2 10` reports ten samples at two-second intervals. The first line usually reflects averages since boot, while later lines describe the interval. Important fields include runnable tasks in `r`, tasks blocked on I/O in `b`, swap traffic in `si` and `so`, block traffic in `bi` and `bo`, interrupts in `in`, context switches in `cs`, user and system CPU in `us` and `sy`, idle CPU in `id`, I/O wait in `wa`, and stolen virtual CPU time in `st`.

Sustained `r` above available CPUs can indicate CPU contention, but a busy system can still meet its throughput and latency targets. Persistent non-zero `si` and `so` indicate active swapping, not simply configured swap space. High `wa` directs investigation towards storage, but it does not prove that a disk has failed.

`mpstat -P ALL 2 10` separates activity by logical CPU. Large imbalances can reveal CPU affinity, interrupt placement, a single-threaded workload, or virtual CPU constraints. `mpstat -I ALL` exposes interrupt statistics for comparison with `/proc/interrupts` and `/proc/softirqs`.

`iostat -xz 2 10` gives extended block-device statistics and suppresses devices with no activity. Administrators should correlate throughput and IOPS with `await`, queue depth, request size, and `%util`. On parallel devices such as NVMe, `%util` can reach 100 without proving saturation because the device can service concurrent requests.

`pidstat` attributes activity to processes and threads:

```bash
pidstat -u -r -d -w -p ALL 2 10
pidstat -t -p PID 2 10
```

The first command covers CPU, faults and memory, disk I/O, and context switches. The second separates threads for one process. Major faults require storage access, while minor faults do not.

`sar` records and reads a broad set of metrics:

```bash
sar -u 2 10
sar -r ALL 2 10
sar -d -p 2 10
sar -n DEV,TCP,ETCP 2 10
sar -q 2 10
```

Historical files normally reside under `/var/log/sa/`. The `-f` option selects an archive. `sadf` can export the same data as delimited text, JSON, XML, or SVG. Native output options should take precedence over text substitution because they preserve quoting and field structure.

```bash
sadf -j /var/log/sa/sa31 -- -u
sadf -d /var/log/sa/sa31 -- -r ALL
```

All interval tools need a long enough observation window to include warm-up, steady state, and relevant peaks. Synchronised clocks and timestamps allow administrators to correlate application logs, kernel messages, and infrastructure metrics.
### Correlating resource symptoms
No single counter identifies a bottleneck. A diagnosis should combine demand, delay, and completed work:

| Resource | Demand evidence | Delay or pressure evidence | Work evidence |
| --- | --- | --- | --- |
| CPU | Run-queue length, busy CPU time, and runnable threads | Scheduler latency, context switches, migrations, and CPU pressure stalls | Requests, transactions, or jobs completed |
| Memory | Working-set size, allocation rate, and cache use | Reclaim, major faults, swap traffic, and memory pressure stalls | Application throughput and allocation success |
| Storage | IOPS, bandwidth, request size, and queue depth | `await`, device latency, I/O pressure stalls, and application wait time | Bytes, files, or transactions completed |
| Network | Packets, bytes, connections, and queue load | Drops, retransmissions, receive errors, and round-trip time | Application messages or responses completed |

A host can show high utilisation and still operate efficiently. Conversely, a lightly utilised host can deliver poor service when one serial component, lock, queue, or remote dependency blocks progress. The constrained resource is the component whose additional capacity or reduced demand changes the workload result.

Pressure Stall Information reports the proportion of time in which tasks wait for CPU, memory, or I/O:

```bash
cat /proc/pressure/cpu
cat /proc/pressure/memory
cat /proc/pressure/io
```

The `some` line records periods when at least one task stalls. The `full` line records periods when all non-idle tasks stall together. CPU pressure has no `full` line because at least one runnable task always executes. The `avg10`, `avg60`, and `avg300` fields are recent percentages, while `total` accumulates stall time in microseconds. PSI helps distinguish busy but productive work from resource contention, especially when aligned with application latency and throughput.

Sampling can miss short events. Historical `sar` or PCP archives establish when a change began, while event-based tools such as `perf`, SystemTap, and eBPF identify the responsible code path. Application logs then connect the system event to a user-visible operation. Agreement across these layers provides stronger evidence than any isolated peak.
### perf
`perf` connects user-space analysis to kernel performance counters, tracepoints, kprobes, and uprobes. It can answer whether a process spends cycles executing instructions, missing caches, switching context, waiting on faults, or running particular functions.

```bash
perf list
perf stat application arguments
perf top
perf record -g -- application arguments
perf report
```

`perf stat` counts events over a run. Instructions, cycles, branches, branch misses, cache references, cache misses, task clocks, context switches, and faults help characterise a workload. `perf record` samples activity into `perf.data`, while `perf report` attributes samples to symbols. `-g` captures call graphs when the binaries, unwinding method, and permissions support them.

Missing debug symbols reduce function and source resolution. Sampling frequency, call-graph collection, and system-wide capture add overhead. An administrator should begin with a short, scoped recording and expand it only when the evidence supports a broader trace.
### Repeatable workload comparison
A useful comparison holds input data, concurrency, duration, warm-up, cache state, CPU placement, and external dependencies constant. At least several runs are needed because scheduler activity, background work, thermal behaviour, and storage state create natural variation. The result should report a distribution or confidence range, not only the best run.

The external `time` command adds process resource statistics:

```bash
/usr/bin/time -v application arguments
```

Elapsed time describes user-visible duration. User and system CPU time separate application execution from kernel work. Maximum resident set size, major and minor faults, voluntary and involuntary context switches, file-system operations, and exit status help explain a changed result. CPU time can exceed elapsed time in a multithreaded process because several CPUs execute concurrently.

System-call tracing can locate repeated failures, slow calls, and unexpected file or network activity:

```bash
strace -f -tt -T -o application.strace application arguments
strace -c -f application arguments
```

`-f` follows child processes and threads, `-tt` records timestamps, and `-T` reports time spent in each call. Summary mode aggregates calls and errors. Tracing every call changes timing and can generate sensitive arguments or data, so administrators should narrow the process, syscall set, and duration. `perf trace` offers another system-call view and integrates with kernel events.

A result becomes operationally useful only when it links a configuration change to a workload outcome. The record should include the exact command, input version, environment, start and finish times, raw data, summary method, and rollback state. A statistically visible change that does not improve the service objective should not become a production tuning policy.
## Application analysis and tracing
### Valgrind
Valgrind instruments user-space binaries and adds substantial overhead. Administrators should run it against a reproducible workload outside production whenever possible:

```bash
dnf install valgrind
```

Its principal tools include:

| Tool | Purpose |
| --- | --- |
| `memcheck` | Invalid access, uninitialised values, invalid frees, overlap, and leaks |
| `cachegrind` | Simulated cache and branch behaviour |
| `callgrind` | Function call relationships and event counts |
| `massif` | Heap consumption over time |
| `helgrind` | POSIX thread synchronisation errors and possible data races |
| `drd` | Threading errors in POSIX-threaded programs |

Memcheck runs by default:

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./application
```

The error context, allocation stack, leak classification, and final summary help developers locate defects. `definitely lost` normally indicates a leak. `still reachable` can represent intentionally retained allocations, so the result needs application knowledge.

Cachegrind and Callgrind write detailed files for later annotation:

```bash
valgrind --tool=cachegrind ./application
cg_annotate cachegrind.out.PID
valgrind --tool=callgrind ./application
callgrind_annotate callgrind.out.PID
```

Massif records heap use and `ms_print` renders the snapshots:

```bash
valgrind --tool=massif ./application
ms_print massif.out.PID
```

Valgrind identifies evidence for developers and vendors. It does not repair a program, and instrumentation can alter timing enough to hide or expose concurrency behaviour.
### SystemTap
SystemTap probes kernel and user-space events through scripts. It can aggregate events, filter by process or device, and report behaviour that broad monitoring tools cannot attribute. It compiles many scripts into instrumentation modules, so the running kernel and debug packages must match.

The usual preparation is:

```bash
dnf install systemtap
stap-prep
stap -v -e 'probe begin { log("ready") exit() }'
```

The debug and source repositories must be enabled. `stap-prep` installs packages for the running kernel, so `uname -r` should match the kernel intended for tracing. Restricted deployments can build an instrumentation module on a compatible host and install only `systemtap-runtime` on the target.

SystemTap processes a script through parsing, semantic analysis, translation to C, compilation, and execution. Verbose output exposes these passes. Supplied examples under `/usr/share/systemtap/examples/` include process, network, storage, and system-call probes.

Scripts should limit scope, duration, output volume, and aggregation cardinality. A broad probe on a busy host can consume significant CPU or produce unmanageable output. Scripts that observe terminal input or user activity also require explicit organisational authority and privacy controls. SystemTap or kprobes must not run while a kernel live patch is being applied.
### eBPF, BCC, and bpftrace
eBPF runs verified programs at kernel hooks without recompiling the kernel or loading a conventional tracing module. RHEL 10 supplies pre-built BCC tools and the `bpftrace` language:

```bash
dnf install bcc-tools bpftrace
ls /usr/share/bcc/tools
ls /usr/share/bcc/tools/doc
```

Useful BCC tools include:

| Tool             | Observation                                     |
| ---------------- | ----------------------------------------------- |
| `execsnoop`      | New process executions                          |
| `opensnoop`      | File open operations                            |
| `syscount`       | System-call counts or rates                     |
| `gethostlatency` | Name-resolution latency                         |
| `tcplife`        | TCP connection lifetimes and traffic            |
| `biolatency`     | Block I/O latency distribution                  |
| `biotop`         | Processes generating block I/O                  |
| `xfsslower`      | Slow XFS read, write, open, and sync operations |

Examples:

```bash
/usr/share/bcc/tools/execsnoop
/usr/share/bcc/tools/opensnoop -n uname
/usr/share/bcc/tools/biotop 30
/usr/share/bcc/tools/xfsslower 10
```

Filters by process, user, device, port, and latency threshold reduce overhead and noise. Histograms such as `biolatency` reveal distributions that an average conceals. A high-latency tail may distinguish occasional cache misses from sustained device congestion.

`bpftrace` supports concise probes and custom aggregation:

```bash
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count() }'
```

Administrators should prefer documented tracepoints over unstable internal function names. They should also verify tool syntax against the installed version because BCC options can change between releases.
## Performance Co-Pilot
Performance Co-Pilot, or PCP, collects, stores, retrieves, and visualises metrics through a consistent namespace. It can monitor one host, archive local metrics, or centralise data from many clients. Its architecture separates metric providers from consumers:
- `pmcd` coordinates access to live metrics from Performance Metrics Domain Agents.
- `pmlogger` records selected metrics into archives.
- `pmie` evaluates rules and can report performance conditions.
- Client tools query live hosts or replay archives through the same metric names.

A compact local installation uses:

```bash
dnf install pcp pcp-zeroconf
systemctl enable --now pmcd pmlogger
pcp
```

The `pcp` command summarises the platform, active agents, connected clients, and archive state. Archives normally reside below `/var/log/pcp/pmlogger/<hostname>/`.

`pmlogconf` controls the metric groups and intervals in the primary logger configuration:

```bash
pmlogconf -r /var/lib/pcp/config/pmlogger/config.default
systemctl restart pmlogger
```

Logging every metric at a short interval wastes storage and CPU. The archive should retain the metrics and resolution needed for the investigation, along with enough history to compare normal and abnormal periods.

PCP tools serve distinct roles:

| Tool | Role |
| --- | --- |
| `pminfo` | Search the metric namespace and display descriptors or values |
| `pmprobe` | Test metric availability and enumerate instances |
| `pmrep` | Report live or archived metrics in tables or export formats |
| `pmdumplog` | Inspect archive labels, metadata, and contents |
| `pmlogsummary` | Summarise archive values |
| `pmdiff` | Compare archives |
| `pmlogextract` | Merge or extract archive ranges |
| `pmchart` | Plot live or archived metrics in a graphical session |

`pminfo -t metric.name` shows a description, while `pminfo -f metric.name` includes current values. `pmprobe -I metric.name` lists instances. `pmrep` supports timestamps, start and finish times, intervals, sample counts, and CSV output:

```bash
pmrep --archive archive_name --start @03:00 --interval 5seconds --samples 10 --output csv disk.dev.write
```

A central collector needs `pcp-system-tools`, client reachability to `pmcd` on TCP port 44321, an appropriate firewall rule, SELinux policy, and a control entry under `/etc/pcp/pmlogger/control.d/`. Administrators should bind `pmcd` only to required addresses and protect access across untrusted networks. `pmdumplog -L` verifies that each remote archive names the expected host and time range.

PCP provides continuity across live and historical analysis. An administrator can detect a current spike with `pmrep`, inspect the same metric before the event, compare archives, and graph the relevant interval without changing metric definitions.
## Tuning a running system
### Process priorities and scheduling
The nice value influences CPU time for normal processes under the scheduler. It ranges from `-20`, which receives the greatest weight, to `19`, which receives the least. A new process normally starts at `0`, and a child inherits its parent's value.

Administrators can inspect nice values and scheduling classes with:

```bash
ps -eo pid,ppid,comm,ni,cls,rtprio,psr,%cpu --sort=-ni
top -o NI
```

`nice` starts a process with a chosen value:

```bash
nice -n 10 application
```

`renice` changes an existing process:

```bash
renice -n 5 -p PID
```

An unprivileged user can generally increase the nice value of an owned process, which lowers its CPU entitlement. Lowering the nice value requires suitable privilege. Niceness changes relative CPU allocation among runnable normal tasks. It does not impose a CPU ceiling, reserve a core, or override I/O and memory bottlenecks.

Real-time policies need stronger safeguards. `SCHED_FIFO` runs the highest-priority ready thread until it blocks, yields, exits, or a higher-priority thread pre-empts it. `SCHED_RR` adds a time slice among threads at the same priority. Real-time priorities range from 1 to 99, with 99 highest. An unsuitable real-time thread can starve critical services.

`chrt` inspects or sets scheduling policy:

```bash
chrt -p PID
chrt --fifo 10 application
chrt --round-robin --pid 10 PID
```

Administrators should start with low real-time priorities, limit runtime, retain an administrative console, and measure scheduler latency and service health. A persistent service policy belongs in a systemd override, not an ad hoc command.
### TuneD profiles
TuneD applies coordinated settings for workload classes such as virtual guests, throughput, latency, and power saving. It uses static and dynamic tuning plugins and can manage sysctl values, devices, CPU policy, disks, network interfaces, boot arguments, and scripts.

```bash
dnf install tuned
systemctl enable --now tuned
tuned-adm list
tuned-adm recommend
tuned-adm active
tuned-adm verify
```

`tuned-adm recommend` selects a profile from detected product and workload characteristics. The recommendation provides a starting point, not a substitute for measurement. Administrators activate a profile and then verify it:

```bash
tuned-adm profile throughput-performance
tuned-adm active
tuned-adm verify
```

RHEL 10 stores distribution profiles in `/usr/lib/tuned/profiles/` and custom profiles in `/etc/tuned/profiles/`. The older custom path `/etc/tuned/<profile>/` is obsolete for RHEL 10. Administrators should not edit the distribution copy because package updates can replace it.

A custom profile can inherit a supplied profile and override selected controls:

```ini
# /etc/tuned/profiles/database-local/tuned.conf
[main]
summary=Database profile with lower swappiness
include=throughput-performance

[sysctl]
vm.swappiness=10
```

The following commands activate and verify it:

```bash
tuned-adm profile database-local
tuned-adm verify
```

Profile sections should use the most specific TuneD plugin available. The generic `sysctl` plugin suits settings that another plugin does not manage. Script plugins need idempotent actions, bounded runtime, secure ownership, and a rollback path.

`tuned-adm off` temporarily disables tunings. TuneD reapplies them after the service restarts. Permanent removal requires:

```bash
systemctl disable --now tuned
```

No-daemon mode applies settings and exits, but it loses D-Bus, hot-plug, monitoring, and rollback functions. The standard service mode suits most systems.
### PowerTOP
PowerTOP attributes wake-ups and estimated power use to processes, devices, kernel work, idle states, and frequency states. It also suggests tunables:

```bash
dnf install powertop tuned-utils
powertop
powertop --time=10
powertop --html=powertop.html
```

Hardware calibration improves estimates on supported physical systems:

```bash
powertop --calibrate
```

Virtual machines often provide incomplete energy and idle-state data. Even on physical systems, an estimate should be compared with workload throughput, latency, and external power measurements when energy is a formal target.

The `powertop.service` can apply all recommendations at boot. That broad action can affect USB devices, networking, storage latency, and wake behaviour. `powertop2tuned` provides finer control and rollback:

```bash
powertop2tuned power-custom
```

RHEL 10 creates the profile under `/etc/tuned/profiles/power-custom/`. Suggested settings start disabled unless the administrator requests otherwise. The administrator should review `tuned.conf`, enable selected entries, activate the profile, and repeat the workload:

```bash
tuned-adm profile power-custom
tuned-adm verify
```

Switching to the previous profile reverses the TuneD-managed settings without requiring a reboot in most cases.
## Resource control with cgroup v2 and systemd
RHEL 10 uses the unified cgroup v2 hierarchy, and systemd always operates through cgroup v2. RHEL 8 guidance that treated cgroup v1 as the default does not apply. systemd maps services, scopes, and slices directly into `/sys/fs/cgroup/` and should manage application resources in normal administration.

The following commands expose the hierarchy and live consumption:

```bash
systemd-cgls
systemd-cgtop
systemctl status application.service
cat /proc/PID/cgroup
```

The correct utilities are `systemd-cgls` and `systemd-cgtop`, not options passed to a `systemd` command. A cgroup v2 membership line resembles:

```text
0::/system.slice/application.service
```

Resource policies use four broad models:

| Model | Examples | Effect |
| --- | --- | --- |
| Weight | `CPUWeight=`, `IOWeight=` | Divides a contested resource proportionally |
| Limit | `CPUQuota=`, `MemoryMax=` | Sets a maximum available amount |
| Protection | `MemoryLow=` | Protects memory below a threshold when pressure occurs |
| Placement | `CPUAffinity=`, `NUMAPolicy=`, `NUMAMask=` | Restricts processor or memory-node placement |

`MemoryHigh=` provides a throttling threshold before `MemoryMax=` imposes a hard limit. `CPUQuota=20%` allows the service to consume one fifth of one CPU. A quota above 100 percent permits use of more than one CPU.

`systemctl set-property` creates a persistent drop-in unless `--runtime` restricts it to the current boot:

```bash
systemctl set-property application.service CPUWeight=200
systemctl set-property application.service CPUQuota=50%
systemctl set-property application.service MemoryHigh=1G MemoryMax=1200M
systemctl show application.service -p CPUWeight -p CPUQuotaPerSecUSec -p MemoryHigh -p MemoryMax
```

A temporary experiment uses:

```bash
systemctl set-property --runtime application.service MemoryMax=1200M
```

For explicit configuration, `systemctl edit application.service` creates an override under `/etc/systemd/system/application.service.d/`:

```ini
[Service]
CPUQuota=50%
MemoryHigh=1G
MemoryMax=1200M
```

Administrators should not edit vendor units under `/usr/lib/systemd/system/`. An override survives package updates and makes the local policy clear. After a manual override change, the administrator runs:

```bash
systemctl daemon-reload
systemctl restart application.service
```

`systemctl status`, `systemctl show`, `systemd-cgtop`, and files such as `memory.current`, `memory.events`, `memory.max`, `cpu.stat`, and `io.stat` verify enforcement. A hard memory limit can invoke the cgroup out-of-memory handler. `MemoryHigh=` often provides a safer first control because it reveals throttling before a hard failure.

`systemd-run` creates a transient service or scope for a command:

```bash
systemd-run --unit=test-load.service -p CPUQuota=25% -p MemoryMax=512M application
```

Direct writes to cgroupfs can help with specialised testing, but hand-built cgroups and PID assignments are easy to lose when a service restarts. A process-specific change also misses replacement processes unless a parent unit captures them. Persistent policy should normally attach to the systemd service, scope, or slice.
### Hierarchy and I/O controls
Resource controls inherit through the systemd hierarchy. A service belongs to a slice, and the parent slice can restrict the combined resources of all child units. A child cannot escape a tighter ancestor limit. This structure supports policies such as one budget for all batch services and separate limits within that budget.

The following commands expose unit placement and inherited settings:

```bash
systemctl show application.service -p Slice -p ControlGroup
systemctl show system.slice -p CPUWeight -p MemoryHigh -p MemoryMax
systemctl status application.service
```

Weights divide a resource only when siblings compete for it. `CPUWeight=200` does not reserve CPU time when no contention exists. Limits impose a ceiling even when spare capacity remains. Memory protection influences reclaim but does not allocate or lock physical RAM.

systemd also exposes cgroup v2 block-I/O controls:

```bash
systemctl set-property application.service IOWeight=200
systemctl set-property application.service IOReadBandwidthMax="/dev/mapper/vg_data-lv_data 100M"
systemctl set-property application.service IOWriteBandwidthMax="/dev/mapper/vg_data-lv_data 50M"
```

`IOWeight=` distributes contended I/O between units. Bandwidth properties cap traffic to a named block device. Limits apply at the device layer, so an administrator must resolve the actual path through LVM, encryption, multipath, or software RAID. A limit on the wrong layer can have no useful effect or can constrain unrelated workloads that share the device.

Tests should inspect `io.stat`, application latency, device latency, and throughput before and after a control. Buffered writes can initially complete in memory and reach the device later, which can hide enforcement during a short test. A sustained workload that exceeds cache capacity reveals the steady-state effect more clearly.

Changes also need failure testing. A `MemoryMax=` value can terminate processes, a CPU quota can delay health checks, and an I/O cap can lengthen shutdown or recovery. Service start timeouts, restart policy, and dependency behaviour should remain safe when the unit reaches its limit.
## Hardware, CPU, interrupts, and NUMA
### Hardware and kernel evidence
Hardware topology and firmware can change application behaviour even when the operating-system configuration appears identical. Useful inventory commands include:

```bash
lscpu
lscpu -e
lsblk -o NAME,KNAME,TYPE,SIZE,ROTA,SCHED,MODEL,WWN
lspci -nnk
dmidecode
numactl --hardware
```

`dmidecode`, not `dmidcode`, reads SMBIOS and DMI tables. It reports BIOS, chassis, board, processor, memory-device, cache, and slot information when the platform exposes it. Virtual firmware may return synthetic or incomplete data.

`dmesg` reads the kernel ring buffer:

```bash
dmesg -T --level=err,warn
journalctl -k -b
journalctl -k -b -1
```

Human-readable wall-clock timestamps from `dmesg -T` can be imprecise after suspend or clock adjustment. `journalctl -k` preserves boot selection and structured timestamps. Errors and warnings can identify driver retries, device resets, firmware faults, allocation failures, and boot delays, but each message needs workload and hardware context.

RHEL 10 uses `sos report`, not the retired `sosreport` command:

```bash
dnf install sos
sos report --case-id CASE_ID
```

`sos report` gathers configuration, package, service, kernel, log, and diagnostic information into an archive for support analysis. Reports can contain hostnames, addresses, account data, and configuration secrets. Administrators should handle them as sensitive records and use `sos clean` when obfuscation is required.
### CPU frequency, idle states, and thermal limits
Processor frequency and idle-state policy can change latency, throughput, and power use without changing process demand. Firmware, the scaling driver, the governor, TuneD, thermal limits, and a hypervisor can all influence the observed speed.

RHEL 10 supplies CPU power-management tools through `kernel-tools`:

```bash
dnf install kernel-tools
cpupower frequency-info
cpupower monitor
```

`frequency-info` reports the scaling driver, available policy, hardware limits, and active governor. `monitor` samples supported idle-state, frequency, and residency counters. The sysfs view helps compare individual CPUs:

```bash
grep . /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver
grep . /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
grep . /sys/devices/system/cpu/cpu0/cpufreq/scaling_{min,max}_freq
```

Not every platform exposes every file. A virtual machine can show synthetic values, while modern scaling drivers can select frequency within a broad performance or energy policy. The `performance` and `powersave` governor names therefore do not always represent a fixed maximum or minimum clock.

A CPU-bound workload that slows as a test continues may have reached a power or thermal limit. Administrators should compare frequency, idle residency, hardware error reports, firmware settings, cooling, ambient conditions, and workload placement. Kernel messages and `/sys/class/thermal/` can expose some platform limits, but management-controller telemetry often provides the clearest physical evidence.

TuneD profiles coordinate CPU policy with other controls. `latency-performance` favours fast response, `throughput-performance` favours sustained work, and power-oriented profiles accept more wake-up or frequency transition latency. Manually changing a governor while TuneD remains active can produce a setting that TuneD later replaces.

A frequency-policy test should use the same workload, CPU set, cooling state, and duration. It should report work completed, response-time distribution, power consumption, and throttling, not clock speed alone. A higher frequency can increase heat and power enough to reduce sustained performance. A successful policy needs persistent management through TuneD or another single owner and verification after reboot.
### CPU and IRQ placement with Tuna
Tuna displays and changes thread policy, priority, CPU affinity, CPU isolation, and IRQ affinity:

```bash
dnf install tuna
tuna show_threads
tuna show_threads -t 'sshd*'
tuna show_irqs
```

RHEL 10 uses the command-oriented Tuna syntax. To move selected threads or an IRQ:

```bash
tuna move --cpus 0,1 -t 'sshd*'
tuna move --irqs 128 --cpus 3
```

To isolate or return CPUs:

```bash
tuna isolate --cpus 2,3
tuna include --cpus 2,3
```

Affinity can reduce cache migration and remote-memory access, but it can also overload a CPU and prevent the scheduler from using idle capacity. IRQ placement should account for receive and transmit queues, application CPUs, NUMA locality, and `irqbalance`. Some managed or per-CPU interrupts cannot move.

Tuna changes to running threads do not automatically cover a replacement process. A persistent service assignment uses systemd:

```bash
systemctl set-property application.service CPUAffinity=2-3
systemctl restart application.service
```

Low-latency systems can use the TuneD `cpu-partitioning` profile. Isolated CPUs should host explicitly pinned application threads, while housekeeping CPUs handle services, kernel work, timers, and movable interrupts. The administrator must verify isolation after reboot.
### NUMA locality
Non-Uniform Memory Access systems provide lower latency when a thread accesses memory attached to its local CPU node. A scheduler migration can leave memory on the original node and turn local access into remote access.

```bash
numactl --hardware
numastat
numastat -p PID
```

High `numa_hit` and low `numa_miss` values generally indicate effective placement. `numa_miss`, `numa_foreign`, and per-process allocation across distant nodes can reveal locality problems. CPU saturation, insufficient local memory, and deliberate interleaving can also produce remote allocation, so the figures need application context.

`numactl` can launch a test with CPU and memory placement:

```bash
numactl --cpunodebind=0 --membind=0 application
```

A strict bind can fail or cause reclaim when the selected node lacks memory. `--preferred` allows fallback, while `--interleave` distributes pages across nodes. For services, systemd supports persistent `NUMAPolicy=` and `NUMAMask=` settings. A strict memory policy should align with `CPUAffinity=`.
## Memory tuning
### Observation and pressure
Memory analysis should separate free memory, available memory, file cache, anonymous memory, reclaim, swap activity, faults, and cgroup pressure:

```bash
free -h
grep -E 'MemAvailable|AnonPages|Cached|Dirty|Writeback|Swap|Huge' /proc/meminfo
vmstat 2 10
sar -r ALL 2 10
sar -B 2 10
cat /proc/pressure/memory
```

`MemAvailable` estimates memory available for new work without swapping. Low `MemFree` is normal when useful cache occupies RAM. Persistent swap-in and swap-out activity, direct reclaim, major faults, or memory pressure stalls provide stronger evidence of contention.
### Overcommit and swap preference
`vm.overcommit_memory` controls virtual-memory allocation:

| Value | Policy |
| --- | --- |
| `0` | Kernel heuristic, which is the default |
| `1` | Always permit overcommit |
| `2` | Enforce a commit limit |

With policy `2`, the limit derives from swap plus a proportion of RAM controlled by `vm.overcommit_ratio`, unless `vm.overcommit_kbytes` supplies an absolute value. Strict accounting suits workloads that must fail an allocation early instead of risking later out-of-memory termination. Policy `1` suits selected workloads that reserve large sparse address spaces, but it increases the need for application-level control.

The relevant values appear through:

```bash
sysctl vm.overcommit_memory vm.overcommit_ratio vm.overcommit_kbytes
grep -E 'CommitLimit|Committed_AS' /proc/meminfo
```

`vm.swappiness` ranges from 0 to 200 in RHEL 10. A lower value favours retaining anonymous memory and reclaiming file cache. A higher value favours retaining file-backed cache and swapping less-active anonymous pages. Setting `0` aggressively avoids anonymous swapping and can raise the chance of out-of-memory termination. Administrators should choose from workload evidence rather than applying a universal low value.

Persistent settings belong in `/etc/sysctl.d/`:

```ini
# /etc/sysctl.d/60-memory-policy.conf
vm.overcommit_memory = 2
vm.overcommit_ratio = 80
vm.swappiness = 20
```

The administrator should calculate the resulting commit limit, test allocation failure behaviour, and monitor paging and service latency before deployment.
### HugeTLB and transparent huge pages
Larger pages reduce translation lookaside buffer pressure for memory-intensive workloads. They also consume physically contiguous memory, reduce allocation flexibility, and can waste space when applications use only part of each page.

Base page size depends on architecture and installed kernel. On x86-64 it is normally 4 KiB. Common HugeTLB sizes are 2 MiB and 1 GiB. The following commands show the active base and huge-page configuration:

```bash
getconf PAGESIZE
grep -i huge /proc/meminfo
cat /sys/kernel/mm/transparent_hugepage/enabled
numastat -cm
```

Static HugeTLB pages can be reserved at runtime:

```bash
sysctl -w vm.nr_hugepages=10
```

Runtime allocation can fail after memory fragments. RHEL recommends boot-time reservation when a workload needs a dependable pool:

```bash
grubby --update-kernel=ALL --args="hugepagesz=2M hugepages=10 hugepagesz=1G hugepages=1"
reboot
```

`HugePages_Total`, `HugePages_Free`, `HugePages_Rsvd`, `HugePages_Surp`, and `Hugepagesize` in `/proc/meminfo` verify the pool. On NUMA systems, administrators should inspect per-node allocation and align application placement.

Transparent huge pages, or THP, promote eligible mappings automatically. The available modes include `always`, `madvise`, and `never`. `madvise` restricts promotion mainly to applications that request it and often balances benefit against compaction latency.

A RHEL 10 TuneD profile can persist THP policy:

```ini
# /etc/tuned/profiles/thp-madvise/tuned.conf
[main]
include=throughput-performance

[bootloader]
cmdline=transparent_hugepage=madvise
```

After activating the profile, the administrator reboots if the boot argument changes, checks the active mode, and measures page faults, compaction, memory use, and application latency. Disabling THP by habit can remove a useful optimisation.
### System V shared memory
Applications such as databases can depend on System V shared memory. The key controls are:

| Parameter | Function |
| --- | --- |
| `kernel.shmmax` | Maximum bytes in one shared-memory segment |
| `kernel.shmall` | Total shared-memory pages allowed system-wide |
| `kernel.shmmni` | Maximum number of shared-memory segments |

```bash
sysctl kernel.shmmax kernel.shmall kernel.shmmni
ipcs -lm
ipcs -m
```

Values must match application allocation patterns, page size, container limits, and available RAM. Excessive limits do not reserve memory, but they can permit one workload to consume resources needed by others.
### Dirty page writeback
Dirty pages hold modified file data that has not reached storage. The kernel starts background writeback at `vm.dirty_background_ratio` or `vm.dirty_background_bytes`. A generating process begins synchronous writeback at `vm.dirty_ratio` or `vm.dirty_bytes`.

Only one control from each ratio or byte pair should define the threshold. Byte values often behave more consistently on hosts with very large or changing memory sizes. Additional timers include `vm.dirty_writeback_centisecs`, which controls periodic writeback wake-ups, and `vm.dirty_expire_centisecs`, which determines when dirty data becomes old enough for writeback.

Lower thresholds smooth writeback and reduce the size of a sudden flush, but they can increase write frequency. Higher thresholds support bursts and aggregation, but they can create long stalls and more data at risk before a crash. Administrators should correlate `Dirty` and `Writeback` in `/proc/meminfo` with application latency, `iostat`, and storage-controller behaviour.
## Disk and file subsystems
### I/O schedulers
RHEL 10 block devices support multi-queue scheduling. The available scheduler depends on the kernel, driver, and device:

| Scheduler | Characteristics and common use |
| --- | --- |
| `none` | Minimal FIFO-style scheduling in the block layer, often best for NVMe and fast devices |
| `mq-deadline` | Read and write batches with latency deadlines, suitable for many servers and virtual guests |
| `bfq` | Per-process budgets and low interactive latency, useful for desktops and mixed interactive I/O |
| `kyber` | Latency targets for fast devices such as SSDs and NVMe |

The active scheduler appears in brackets:

```bash
cat /sys/block/sda/queue/scheduler
```

RHEL recommends retaining `none` for NVMe unless measurements justify a change. The kernel's device-specific default is normally a sound baseline.

A runtime test writes a supported value to sysfs:

```bash
echo mq-deadline > /sys/block/sda/queue/scheduler
```

A persistent TuneD profile selects stable device identifiers:

```ini
# /etc/tuned/profiles/storage-latency/tuned.conf
[main]
include=throughput-performance

[disk]
devices_udev_regex=ID_WWN=0x5000000000000000
elevator=mq-deadline
```

WWNs take precedence over changing kernel names such as `sdb`. `udevadm info --query=property --name=/dev/sdb` lists suitable identifiers.

An equivalent persistent udev rule can set the scheduler:

```udev
ACTION=="add|change", SUBSYSTEM=="block", ENV{ID_WWN}=="0x5000000000000000", ATTR{queue/scheduler}="mq-deadline"
```

The administrator reloads the rules, triggers a device change when safe, and checks sysfs:

```bash
udevadm control --reload-rules
udevadm trigger --type=devices --action=change
cat /sys/block/sdb/queue/scheduler
```

Changing the scheduler can reorder or delay I/O. Tests should use representative read and write mixtures, queue depths, request sizes, synchronisation patterns, and latency percentiles.
### File-system selection and layout
File-system design begins with workload characteristics:
- Local, shared, or network storage
- Small files, large files, or a mixture
- Sequential or random access
- Metadata rate and directory scale
- Synchronous write and durability requirements
- Required growth, snapshots, backup, and recovery
- Cluster access and failover model

XFS is the default general-purpose local file system in RHEL 10. It scales to large files, large volumes, parallel I/O, and large directory trees. It grows online but does not shrink. RHEL 10 requires newly created XFS file systems to be at least 300 MiB and does not mount the old XFS V4 format. Data from an old V4 file system needs backup, recreation as V5, and restoration.

Ext4 supports broad general-purpose workloads and can grow online. It can shrink only while unmounted. It remains useful for small file systems, workloads with modest scale, and deployments that require offline shrinking.

RHEL 10 no longer supports the GFS2 file system or the Resilient Storage Add-On. Guidance that recommends GFS or GFS2 shared storage for RHEL 10 is obsolete. NFS provides network file access between Unix-like systems. SMB shares use the `cifs-utils` package and the `cifs` mount type:

```bash
dnf install cifs-utils
mount -t cifs -o credentials=/root/smb.cred,seal,vers=3.0 //server/share /mnt/share
```

Credential files need restrictive permissions. Persistent NFS and SMB mounts require suitable `/etc/fstab` entries, network ordering, timeouts, and failure policy.
### XFS creation, growth, and repair
`mkfs.xfs` creates an XFS file system:

```bash
mkfs.xfs /dev/mapper/vg_data-lv_data
```

RAID alignment can reduce read-modify-write work:

```bash
mkfs.xfs -d su=256k,sw=6 /dev/md0
```

`su` describes the stripe unit and `sw` the number of data units across the stripe. Incorrect values can reduce performance, so administrators should derive them from the array rather than copy an example.

After extending the underlying logical volume, `xfs_growfs` grows the mounted file system. A mount point gives the clearest target:

```bash
lvextend -L +100G /dev/vg_data/lv_data
xfs_growfs /data
```

`resize2fs` grows or shrinks ext4, subject to the online and offline rules. The block device or logical volume must already provide the target capacity.

`xfs_repair` must operate on an unmounted file system. A non-modifying check comes first:

```bash
umount /data
xfs_repair -n /dev/vg_data/lv_data
```

After reviewing the report, removing hardware faults, and securing a backup where possible, the administrator can run:

```bash
xfs_repair /dev/vg_data/lv_data
mount /data
```

`journalctl -k`, `dmesg`, SMART or vendor diagnostics, and controller logs help distinguish file-system damage from failing storage. Repairing metadata without correcting the device fault can produce another failure.
## Network performance
### Baseline and bottleneck location
Network tuning should begin with the failing layer. Packet drops can occur in NIC rings, driver queues, the network backlog, IP processing, socket buffers, or the application. Latency can arise from physical links, congestion, interrupt moderation, routing, DNS, transport retransmission, or application delay.

Useful observations include:

```bash
ip -s link show dev enp1s0
ethtool enp1s0
ethtool -S enp1s0
ethtool -g enp1s0
ss -s
nstat
sar -n DEV,TCP,ETCP 2 10
```

Driver-specific drop counters and ring occupancy direct investigation towards the NIC. TCP retransmissions and resets direct it towards the path or peer. UDP receive errors can indicate socket-buffer or application-consumption limits.

`iperf3` measures controlled TCP or UDP throughput between hosts. A test needs the same path, duration, streams, protocol, window, message size, and CPU placement as the target scenario. Application encryption, framing, and storage can produce a lower ceiling than a synthetic network test.
### TCP buffers and the bandwidth-delay product
The bandwidth-delay product estimates the data in flight needed to fill a path:

```text
BDP in bytes = connection speed in bytes per second x round-trip time in seconds
```

A 10 Gbit/s path with a 17 ms round-trip time has:

```text
(10,000,000,000 / 8) x 0.017 = 21,250,000 bytes
```

RHEL 10 uses three values for `net.ipv4.tcp_rmem` and `net.ipv4.tcp_wmem`: minimum, default, and maximum. A maximum around two to three times the measured BDP often provides a test starting point. The default should increase cautiously because large defaults multiply across every socket and can cause buffer pressure and latency spikes.

```ini
# /etc/sysctl.d/60-tcp-buffers.conf
net.ipv4.tcp_rmem = 4096 262144 42500000
net.ipv4.tcp_wmem = 4096 262144 42500000
```

```bash
sysctl -p /etc/sysctl.d/60-tcp-buffers.conf
```

Changing the default value requires affected applications to reopen sockets. TCP autotuning can apply a changed maximum to established connections. Administrators should verify throughput, retransmissions, memory use, and latency after the change.
### UDP buffers
UDP has no transport-level retransmission or flow control. When the application or kernel cannot drain a receive buffer quickly enough, new datagrams can be dropped. `net.core.rmem_max` and `net.core.wmem_max` set the largest receive and send buffers that an application can request:

```ini
# /etc/sysctl.d/60-udp-buffers.conf
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

The application must request the larger buffers through `setsockopt()`, or it continues to use default values. Linux accounts up to twice the requested socket memory internally, so a large limit across many sockets can consume substantial RAM. Administrators should increase buffers only after observing drops and should restart the application after changing its configured socket size.

UDP tests should use an unfragmented payload. For an MTU of 1500 bytes, a common IPv4 payload ceiling is 1472 bytes after the 20-byte IP header and 8-byte UDP header. IPv6 requires a 40-byte base header. Path encapsulation can lower the effective ceiling.
### NIC queues, interrupts, and profiles
If NIC drop counters rise because the receive ring fills, `ethtool -g` shows current and maximum ring sizes. RHEL 10 stores persistent ring settings in NetworkManager profiles:

```bash
nmcli connection modify CONNECTION ethtool.ring-rx 4096
nmcli connection modify CONNECTION ethtool.ring-tx 4096
nmcli connection up CONNECTION
```

Larger rings can absorb bursts but can also add queueing latency. Interrupt coalescence trades fewer interrupts and higher throughput against faster packet delivery. Receive-side scaling, queue count, IRQ affinity, CPU frequency, and NUMA placement should keep packet processing near the application and NIC.

TuneD supplies `network-throughput` and `network-latency` profiles. A custom profile can inherit one and add measured site-specific controls. Profile names express goals, but the administrator still needs workload validation.
## Persistence and verification
A correct tuning change has four states:

| State | Required evidence |
| --- | --- |
| Stored | The intended value exists in `/etc`, a boot entry, or another persistent administrator-controlled location |
| Loaded | The running kernel, service manager, TuneD, udev, or application has accepted it |
| Effective | Runtime interfaces report the intended value and the workload uses it |
| Beneficial | Repeated measurements improve the target without unacceptable regressions |

Configuration checks should use the subsystem's runtime view:
- `sysctl key` for runtime kernel parameters
- `/proc/cmdline` for active boot arguments
- `lsmod` and `modinfo` for modules
- `tuned-adm active` and `tuned-adm verify` for TuneD
- `systemctl show` and cgroupfs files for resource controls
- `/sys/block/<device>/queue/scheduler` for I/O scheduling
- `/proc/meminfo` and THP sysfs files for huge pages
- `ss`, `nstat`, `ethtool`, and application telemetry for networking

A final reboot proves persistence. The administrator should repeat the verification commands and a representative performance test after the reboot. A configuration that survives restart but fails its service-level target is persistent, not successful.