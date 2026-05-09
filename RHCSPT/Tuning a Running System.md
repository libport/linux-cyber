# Tuning a Running System
Administrators tune a running Linux system by adjusting process priority, selecting TuneD profiles, applying custom TuneD settings, managing control groups, and reviewing power consumption. These techniques improve performance, reduce unnecessary resource use, and make workload behaviour more predictable.
## Process Priority and Niceness
Linux uses a nice value to influence scheduling priority. The range runs from -20 to 19. Lower values favour the process, while higher values make the process more willing to yield CPU time. Most processes start at 0, and child processes inherit the nice value of the parent process.

Administrators can inspect nice values with process tools:
- `ps -eo pid,comm,nice,cls --sort=-nice`
- `top -o NI`
- `htop`, sorted by the `NI` column

Commands can target all processes, a process name, or a specific process identifier (PID). The `nice` command starts a new process with an adjusted nice value, for example `nice -n 15 bash`. The `renice` command changes a running process. A corrected form is `sudo renice -n 5 -p PID`, or `sudo renice 5 PID` where supported. Decreasing a nice value usually requires elevated privileges. `top` and `htop` can also alter nice values interactively when run with suitable permissions.
## TuneD Profiles
TuneD optimises a system by applying profiles that match workloads such as virtual guests, high throughput, low latency, balanced operation, and power saving. Distribution profiles live in `/usr/lib/tuned`. Custom profiles belong in `/etc/tuned`, and a custom profile with the same name takes precedence over the distribution version.

A TuneD profile normally contains a `tuned.conf` file and may include helper scripts. Static tuning applies predefined settings, including `sysctl` and `sysfs` values. Dynamic tuning monitors system activity and adjusts selected settings while the TuneD service runs. TuneD normally runs as a systemd service so it can monitor the host and support D-Bus, hotplug handling, and rollback. No-daemon mode applies settings and exits, which reduces resident memory use but removes much of TuneD's ongoing management capability.

The main administration tool is `tuned-adm`:
- `tuned-adm active` shows the active profile.
- `tuned-adm verify` checks whether the current system settings match the active profile.
- `tuned-adm list` lists available profiles.
- `tuned-adm recommend` reports the recommended profile.
- `sudo tuned-adm profile balanced` switches to the `balanced` profile.
- `sudo tuned-adm off` temporarily disables active tunings.

A restart of the TuneD service or a reboot can reapply tunings after `tuned-adm off`. Administrators should disable the TuneD service with `systemctl` when they need a persistent stop.
## Custom TuneD Profiles
Custom TuneD profiles let administrators extend or override defaults. TuneD uses monitoring plugins to collect information for dynamic tuning and tuning plugins to apply system changes. A custom `tuned.conf` can include another profile, set kernel parameters through a `sysctl` section, and run scripts through a `script` section.

A typical custom profile process creates a directory such as `/etc/tuned/myprofile`, writes `/etc/tuned/myprofile/tuned.conf`, then activates it with `sudo tuned-adm profile myprofile`. The command `tuned-adm profile_info virtual-guest` shows profile information, while `cat /usr/lib/tuned/virtual-guest/tuned.conf` displays the profile file when permissions allow. Administrators can confirm the result with `tuned-adm active` and by checking the affected setting. For example, a profile can enable IPv6 forwarding by setting `net.ipv6.conf.all.forwarding=1`, then verify it with `sysctl net.ipv6.conf.all.forwarding`. A script action can create an audit file such as `/tmp/tuned.txt`, but production scripts should record useful operational evidence and fail safely.

Custom profiles can manage CPU latency, performance settings, isolated cores, huge pages, kernel parameters, disk behaviour, and mount options. The configuration should remain explicit, testable, and reversible.
## Control Groups with PIDs
Control groups, or cgroups, organise processes into hierarchical groups so the kernel can limit, prioritise, or isolate resources such as CPU time, memory, network bandwidth, and disk input and output.

Two cgroup versions matter. Cgroups version 1 uses separate per-resource hierarchies, which can lead to inconsistent structure and naming. Cgroups version 2 uses a single unified hierarchy with more consistent control files.

PID-based cgroup changes suit testing and troubleshooting because they act on a running process immediately. They do not provide durable configuration. A reboot, service restart, or process restart changes the PID and breaks the association. Persistent resource policy belongs at the systemd service level.

In a cgroups version 1 CPU example, an administrator can create a directory under `/sys/fs/cgroup/cpu`, set quota and period values, start a CPU-heavy `dd if=/dev/zero of=/dev/null &` process, and write that process PID to the cgroup tasks file. The process should then consume approximately the configured CPU share. The path `/proc/PID/cgroup` confirms group membership. A second `dd` process remains unrestricted unless added to the same cgroup. This distinction shows why PID-based control changes behaviour only for selected running processes, not for future instances of the same command or service.

In a cgroups version 2 example, the administrator enables relevant controllers, creates a subdirectory in the cgroup mount, sets `cpuset.cpus` and `cpu.max`, then writes the PID to `cgroup.procs`. The limits apply only to member processes.
## systemd and Cgroups
systemd integrates cgroups with unit management and moves resource control from individual PIDs to applications and services. This structure suits persistent configuration because systemd can apply resource settings whenever a unit starts.

Resource control uses several unit types:
- Service units manage processes started by systemd.
- Scope units group externally created processes.
- Slice units organise services and scopes into a hierarchy.

Administrators should use the correct tools. The commands are `systemd-cgls` and `systemd-cgtop`, not `systemd -cgls` or `systemd -cgtop`. `systemd-cgls` displays the cgroup hierarchy. `systemctl status SERVICE` shows the cgroup for a specific service. `systemd-cgtop` monitors cgroup resource use and can sort by CPU, memory, input and output, or task count.

systemd resource policy includes weights, limits, protections, and allocations. CPU weight distributes proportional CPU access. `CPUQuota=20%` caps a service at about one fifth of one CPU. For memory on cgroups version 2, current systemd documentation uses `MemoryHigh=` for throttling and `MemoryMax=` as the hard limit. Older material may show `MemoryLimit=`, but `MemoryMax=` is the clearer current property for a maximum memory cap.

`sudo systemctl set-property stopme3.service CPUQuota=20%` creates a drop-in configuration and applies the quota. A service file can also contain the property directly under its service section, followed by `sudo systemctl daemon-reload` and a service restart. Rollback removes the drop-in or deletes the property from the unit file, then reloads systemd and restarts the service.
## PowerTOP and Power Management
PowerTOP helps administrators identify components that consume power or wake the CPU frequently. It reports activity for processes, devices, kernel workers, idle states, frequency states, device statistics, tunables, and wake-up settings. It can run interactively with `sudo powertop`, use a custom refresh interval with `sudo powertop --time=10`, and generate an offline HTML report with `sudo powertop --html=powertop.html`.

PowerTOP tunables need careful handling. The `Overview` tab highlights activity that uses power or wakes the CPU. `Idle stats` reports processor idle-state use, `Frequency stats` reports performance-state behaviour, `Device stats` focuses on hardware, and `Tunables` lists suggested changes as good or bad. The `WakeUp` tab is often more relevant to laptops than servers. The `powertop` service can apply suggestions at boot, but `powertop2tuned` gives administrators better control. The utility creates a TuneD profile in `/etc/tuned`, bases it on the current TuneD profile, and disables suggested tunings by default for safety. Administrators then uncomment selected tunings or use supported enable options, activate the profile with `tuned-adm profile`, and verify the result in PowerTOP.

Rollback stays simple. Administrators can switch to another TuneD profile or edit the custom profile to remove selected tunings. This approach keeps power tuning integrated with TuneD and avoids enabling every suggestion without review.