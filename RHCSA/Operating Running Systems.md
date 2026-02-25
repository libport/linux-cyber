## System Reboot and Shutdown
Administrators must manage system power operations carefully to protect users and data. RHEL 8 provides three primary commands:
- `shutdown`
- `reboot`
- `poweroff`

The `shutdown` command offers the greatest control. It can schedule events, warn logged in users, and either halt or reboot the system. By contrast, `reboot` and `poweroff` execute immediately and provide no warning. Both are symbolic links to `systemctl` and ultimately to systemd, so they interact with the same service manager but with less flexibility.
### Using shutdown
The shutdown command accepts a time value and an optional broadcast message. Administrators can specify:
- +minutes for delayed execution
- HH:MM for a specific time
- now for immediate action

When fewer than five minutes remain before shutdown, the system automatically restricts new logins by creating the file /run/nologin. Existing users remain connected but cannot log in again. Administrators can also cancel a scheduled shutdown with shutdown -c.

The file /etc/nologin provides manual control over user access. When present, only the root account can log in. This approach is useful during maintenance when the system must remain running but unavailable to standard users.
### Behaviour of reboot and poweroff
The reboot and poweroff commands provide fast control but do not notify users. Because they map directly to systemctl, they are best suited to single user lab environments rather than production systems. In enterprise environments, shutdown remains the preferred method.
## Root Password Recovery in RHEL 8
Loss of the root password can render a system inaccessible. Recovery requires interrupting the boot process through the GRUB boot loader and modifying kernel parameters. The procedure temporarily bypasses normal security controls, including SELinux, so administrators must restore correct contexts afterwards.
### Boot Sequence Overview
System startup follows this path:
- BIOS or firmware initialises hardware
- GRUB loads from the boot partition
- The Linux kernel starts
- The initial ramdisk loads required drivers
- systemd begins normal service startup

GRUB allows editing of kernel parameters before boot. This capability enables password recovery.
### Recovery Procedure
The standard recovery workflow is:
- Interrupt GRUB and edit the selected kernel entry
- Append rd.break to pause at the initramfs stage
- Append enforcing=0 to place SELinux in permissive mode
- Boot with the modified parameters
- Remount the real root filesystem read write
- Use chroot to switch into the real root environment
- Run passwd to set a new root password
- Exit and continue the boot process
- Restore SELinux context on /etc/shadow
- Return SELinux to enforcing mode

This sequence restores access while maintaining system integrity.
### Importance of SELinux Context
SELinux applies mandatory access control labels to files and processes. Even the root account must obey these rules. Changing the root password outside the normal runtime environment causes the /etc/shadow file to receive an incorrect context. Without correction, authentication tools fail.

Administrators can inspect contexts with ls -Z and repair them using restorecon. After recovery, the command restorecon -v /etc/shadow must run before returning SELinux to enforcing mode with setenforce 1.
## Managing System Services
Modern Linux systems use systemd as process ID 1 and as the unified service manager. The systemctl command provides the primary interface for controlling services and other unit types.
### Core systemctl Operations
Common administrative actions include:
- start and stop services
- enable and disable automatic startup
- restart and reload services
- check status information

The --now option combines enable with start or disable with stop, reducing the number of commands required. For example, systemctl enable --now atd both enables and starts the service immediately.

Status output from systemctl status provides extensive diagnostics, including active state, enablement, process identifiers, runtime duration, and recent log entries. This command acts as the primary troubleshooting entry point.
### Privilege Requirements
Standard users can query service status, but starting, stopping, enabling, or disabling services requires elevated privileges through sudo or the root account.
### Multiple Service Control
systemctl accepts multiple unit names or wildcard patterns, allowing administrators to manage groups of services in a single command. This capability improves efficiency compared with earlier tools such as service and chkconfig.
## Socket Activated Services
systemd introduces socket activation to reduce resource consumption. In this model, systemd listens on a network socket and starts the associated service only when a connection occurs.

Enabling cockpit.socket demonstrates this behaviour. systemd holds TCP port 9090 open and launches the Cockpit web service only when needed. After inactivity, the service stops while the socket remains ready. This design lowers memory and CPU usage on lightly used systems.

Administrators can inspect listening sockets and active units with:
- systemctl list-units
- systemctl list-unit-files
- filtering with --type
## Unit File Management
Unit files define how services and other resources behave. RHEL stores vendor supplied units in:
- /usr/lib/systemd/system

Administrative overrides reside in:
- /etc/systemd/system

Files in /etc take precedence over vendor defaults. Administrators should never modify the original files directly.
### Viewing Unit Files
The command systemctl cat unit_name displays the active unit definition and its source path. This method confirms whether a custom override is in effect.
### Editing Units
systemctl edit --full unit_name creates a complete copy of the unit file in /etc/systemd/system and opens it for modification. After changes, administrators should run systemctl daemon-reload so systemd rereads the configuration.
### Masking Services
Masking prevents both manual and automatic service startup. systemctl mask unit_name creates a symbolic link to /dev/null, effectively disabling the unit. This approach is stronger than disable and is useful for preventing unwanted services from running. Unmasking reverses the process.
## Systemd Targets
Targets replace the traditional SysV runlevels. They group related units and define system states. Common targets include:
- multi-user.target for non graphical operation
- graphical.target for GUI environments
- rescue.target for maintenance mode

Administrators can determine the default target with systemctl get-default and change it when necessary. Temporary target changes during boot allow troubleshooting without permanently altering system behaviour.
## Observing and Managing Processes
Effective administration requires monitoring running processes and system performance. Key tools include:
- top for real time monitoring
- ps for process snapshots
- pgrep and pkill for searching and signalling processes
- kill for terminating processes

These utilities help diagnose resource problems and manage misbehaving applications.
## Logging in RHEL 8
RHEL 8 uses two complementary logging systems:
- traditional rsyslog based files
- the systemd journal

The journal stores structured logs that administrators can query with journalctl. Combining journal data with systemctl status output provides rapid insight into service failures and system events.

Administrators must regularly review logs to detect performance issues, service crashes, and security concerns.
## Performance Observation and Tuning
System performance analysis relies on continuous monitoring of CPU, memory, and process activity. Tools such as top and ps provide immediate visibility, while careful service management reduces unnecessary load. Socket activation, selective service enablement, and targeted troubleshooting all contribute to stable system behaviour.
## Best Practice Guidance
Effective RHEL 8 administration follows several principles:
- use shutdown rather than reboot or poweroff in shared environments
- restrict logins during maintenance using /etc/nologin when appropriate
- understand SELinux context implications when performing recovery tasks
- prefer systemctl for all service management operations
- use socket activation to conserve resources
- maintain clear separation between vendor and local unit files
- reload the systemd daemon after configuration changes
- review logs regularly for early fault detection

Adhering to these practices improves reliability, security, and maintainability.
## Conclusion
Operating RHEL 8 systems requires disciplined control over power management, authentication recovery, and service orchestration. The shutdown command provides safe system transitions, while GRUB based recovery restores lost root access when necessary. systemd and systemctl unify service control across modern Linux distributions and introduce advanced features such as socket activation and target based boot management. Administrators who master these tools can diagnose faults quickly, manage resources efficiently, and maintain stable enterprise systems.