# Profiling and Tuning System Hardware and Performance
Red Hat administrators need a clear hardware profile before they tune performance. Software often receives most attention, but hardware, firmware, kernel messages, CPU placement and memory page choices can explain bottlenecks, startup delays and unstable behaviour.
## Hardware and boot diagnostics
`dmidecode` decodes the system DMI or SMBIOS tables and presents firmware and hardware data in readable form. Its output can include BIOS version, system model, serial numbers, board details, chassis information, processor data, memory slots and supported features. Vendors supply this data, so administrators should treat it as useful but not infallible.

Common commands include:
- `sudo dmidecode | less`
- `sudo dmidecode -t 3`

The first command reviews the full table. The second filters type 3, which reports chassis information. Similar filters can target BIOS, system, baseboard, processor and memory records. On virtual infrastructure, these records can reveal host-generation differences that affect kernel behaviour or application performance.

`dmesg` prints or controls the kernel ring buffer. It helps administrators inspect boot messages, driver messages, hardware detection, warnings and errors. Useful commands include:
- `dmesg -Tx`
- `dmesg --level=err,warn`
- `dmesg -Tx | grep -i rpc`

`-T` converts timestamps to a readable format. `-x` displays facility and priority information. Filtering by level or keyword reduces noise and helps locate startup delays, driver faults or device-related warnings. On RHEL 8 and later, `/var/log/dmesg` is not created by default, so administrators usually inspect the live buffer with `dmesg` or review kernel messages through the systemd journal and system logs.

On current RHEL 8 documentation, administrators use `sos report` rather than the older `sosreport` command name. The tool collects configuration, diagnostic and troubleshooting data for Red Hat support, writes an archive in `/var/tmp`, and can upload the report to a support case. Administrators can list plugins with `sudo sos report -l` and collect selected areas with `sudo sos report -o system,xfs`. They can inspect extracted archives with tools such as `xsos` to review BIOS, operating system, memory and network-device details before or during support work.
## CPU affinity with tuna and systemd
Linux normally schedules work across available CPUs. Busy applications can suffer from extra context switching when related threads move across cores. `tuna` helps administrators inspect and tune thread priorities, scheduler policies, CPU affinity, IRQ handlers, CPU isolation and CPU inclusion on a running system.

Useful commands include:
- `tuna -P`
- `tuna -t sshd -P`
- `tuna -c 1 -t <thread-pattern> --move`
- `tuna -Q`
- `sudo tuna -q <irq> -c <cpu> -m`

`tuna -P` lists process IDs, scheduling policy, priority, CPU affinity, context switches and command names. `tuna -t` targets a process ID or command pattern. `--move` pins matching threads to the selected CPU. `tuna -Q` lists IRQs and their affinity, while `-q` targets one IRQ. Not every IRQ can move, and live `tuna` changes suit troubleshooting more than permanent configuration.

Persistent CPU affinity belongs in systemd. A service can use the `CPUAffinity=` unit property or an override created with `systemctl edit`. After the change, the administrator reloads systemd and restarts the service. This approach can keep a database, batch job or other noisy workload on selected CPUs and reduce interference with other services.
## Huge pages
Linux manages memory in pages. On x86_64 RHEL 8, the standard page size is 4 KB and the default HugeTLB page size is 2 MB. HugeTLB pages reserve large pages for workloads that benefit from fewer page-table entries and lower translation lookaside buffer pressure. Transparent Huge Pages allow the kernel to promote eligible memory automatically and can help some applications, but other workloads require them to be disabled.

Administrators can inspect huge page settings with:
- `cat /proc/cmdline`
- `cat /etc/default/grub`
- `grep -i hugepages /proc/meminfo`
- `sudo sysctl -a | grep vm.nr_hugepages`
- `cat /sys/kernel/mm/transparent_hugepage/enabled`

Static HugeTLB pages can be set at runtime with `vm.nr_hugepages`, but early boot reservation usually gives the best chance of allocating contiguous memory. Kernel parameters can reserve pages with `hugepages=10`, or with paired size and count options such as `hugepagesz=2M hugepages=10`. The order matters when multiple sizes are configured.

Transparent Huge Pages use the singular kernel parameter `transparent_hugepage`. For example, `transparent_hugepage=never` disables them at boot. TuneD can also manage huge page settings through a profile. For THP on RHEL 8, Red Hat documents a persistent TuneD bootloader configuration such as `cmdline = transparent_hugepage=never`, followed by restarting TuneD and activating the profile.