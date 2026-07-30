# Deploying, Configuring and Maintaining Systems
> [!NOTE]
> A practical guide to deploying and maintaining RHEL systems through disciplined package management, time synchronisation, systemd boot configuration, and task scheduling.

Red Hat Enterprise Linux 8 administrators need command-line skills and root access for package management, time synchronisation, boot configuration, and job scheduling. A registered RHEL host normally receives signed software and errata from Red Hat repositories. An installation DVD can supply packages to an isolated host, but it cannot supply later bug fixes or security updates.
## Managing software packages
### DNF, YUM, and repositories
RHEL 8 uses DNF to manage RPM packages and their dependencies. The `yum` command provides a compatible interface to the same package-management stack, so most RHEL 8 documentation and scripts can use either name. Administrators should use one form consistently and should not depend on the implementation path or symbolic-link layout.

BaseOS contains the core operating-system packages. AppStream contains additional applications, language runtimes, databases, and related components. AppStream distributes ordinary RPM packages and modular content:
- A module stream selects a supported version or release line of an application stack.
- A module profile selects a package set for a purpose such as client, server, common, or development use.
- A system normally enables one stream for a module at a time. Modules expose alternative supported streams in the repositories, but they do not generally install several active versions of the same stack together.

The available streams, profiles, and support periods depend on the RHEL 8 minor release and enabled repositories. Administrators should inspect the host rather than rely on old example versions.

Registration and repository access can be checked with:

```bash
$ subscription-manager identity
$ sudo subscription-manager repos --list-enabled
$ sudo dnf repolist
```

Extra Packages for Enterprise Linux, commonly called EPEL, is a separate community repository. Red Hat does not include EPEL packages in the RHEL subscription or support them as RHEL packages.

Unprivileged users can search cached repository metadata, while installation, removal, and system-wide updates require suitable privilege. Common operations include:

```bash
$ dnf list tree
$ dnf info tree
$ dnf provides '*/bin/tree'
$ sudo dnf install tree
$ sudo dnf remove tree
$ sudo dnf check-update
$ sudo dnf update
```

`dnf search` searches package names and descriptions. `dnf list installed` inventories installed packages, while `dnf list available` shows content from enabled repositories. `dnf provides` identifies the package that supplies a path. Quoting a wildcard such as `'*/bin/tree'` prevents the shell from expanding it before DNF receives it.

Administrators should inspect the transaction summary before accepting an installation, removal, or update. The summary identifies packages, dependencies, repository sources, download size, and disk-space effects. The `-y` option accepts the proposal without prompting and therefore suits a reviewed automation workflow, not exploratory administration.

DNF checks RPM signatures against trusted keys when `gpgcheck` remains enabled. A signature failure can indicate a damaged package, an incorrect repository, or an untrusted signing key. Disabling signature checking hides the control rather than resolving the cause. Administrators should correct the repository or key configuration and should obtain keys through an authenticated channel.

`dnf remove` removes the named package and can remove dependencies that no installed package needs. RHEL 8 DNF does not provide a standard `purge` operation equivalent to the Debian command. RPM can preserve modified configuration files during package transactions, so administrators should review application data and retained files separately before deleting them.

DNF treats kernels and other packages listed by the `installonlypkgs` setting differently from ordinary updates. It installs a new kernel alongside retained kernels, subject to the configured limit. The running system continues to use the old kernel until the next boot selects the new one.
### Transaction history
DNF records package transactions. Administrators can inspect the record and reverse suitable installation or removal transactions:

```bash
$ sudo dnf history
$ sudo dnf history info 17
$ sudo dnf history undo 17
$ sudo dnf history rollback 12
```

`history undo` reverses one transaction. `history rollback` reverses later transactions until the system reaches the state associated with the selected transaction number. Both operations require the relevant package versions to remain available.

Administrators should not use transaction history as a general system rollback mechanism. Red Hat does not support downgrading core system packages such as the kernel, `glibc`, SELinux packages, and their critical dependencies through `history undo` or `history rollback`. A tested backup, snapshot, or deployment rollback process provides safer recovery for broad updates.
### Local and web repositories
Repository definitions reside in `/etc/yum.repos.d/` as files ending in `.repo`. One file can define several repositories, and `dnf repolist --all` displays enabled and disabled definitions.

A RHEL installation DVD already contains repository metadata for its BaseOS and AppStream trees. An administrator can mount the media read-only and add both locations:

```bash
$ sudo mkdir -p /mnt/rhel8
$ sudo mount -o ro /dev/sr0 /mnt/rhel8
$ sudo dnf install dnf-plugins-core
$ sudo dnf config-manager --add-repo file:///mnt/rhel8/BaseOS
$ sudo dnf config-manager --add-repo file:///mnt/rhel8/AppStream
$ sudo dnf makecache
```

The directory names and URL paths are case-sensitive. A file repository remains available only while the media or copied content remains mounted at the configured path.

Administrators can also create a repository definition directly. A BaseOS definition for mounted installation media can contain:

```ini
[local-baseos]
name=RHEL 8 local BaseOS
baseurl=file:///mnt/rhel8/BaseOS/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

The bracketed identifier must remain unique across configured repositories. `name` supplies a readable label, `baseurl` identifies the repository root, `enabled` controls normal use, and `gpgcheck` enforces package-signature verification. AppStream needs a separate definition that points to its own tree.

Administrators can isolate a test to selected repositories without permanently changing the definitions:

```bash
$ sudo dnf --disablerepo='*' --enablerepo=local-baseos repolist
$ sudo dnf --disablerepo='*' --enablerepo=local-baseos list available
```

`dnf clean metadata` discards cached metadata when a repository changes. `dnf makecache` then retrieves a fresh copy. Removing a repository normally means deleting or disabling its specific `.repo` definition, followed by a metadata refresh.

An organisation can publish the same repository trees through an HTTP server and configure base URLs such as `http://repo.example.com/rhel8/BaseOS/` and `http://repo.example.com/rhel8/AppStream/`. The server must expose the RPMs and `repodata` directories, and its firewall, file permissions, and SELinux policy must permit HTTP access. `dnf makecache` tests metadata retrieval, but an installation test confirms package access.

DVD content reflects the release recorded on the media. It does not replace access to current Red Hat errata. Organisations that mirror current content should use a supported repository-management or mirroring process.
### Security updates and advisories
RHEL repository metadata identifies security advisories, bug fixes, and enhancements. Administrators can review security errata before changing the system:

```bash
$ sudo dnf updateinfo summary
$ sudo dnf updateinfo list --security
$ sudo dnf updateinfo info RHSA-YYYY:NNNN
```

DNF can apply all available security updates, restrict an update to a severity, or apply a named Red Hat Security Advisory:

```bash
$ sudo dnf update --security
$ sudo dnf update --security --sec-severity=Important
$ sudo dnf update --advisory=RHSA-YYYY:NNNN
```

An advisory can update several packages, and one package transaction can address several advisories. Administrators should review the proposed transaction, test changes where operational risk warrants it, and reboot when a new kernel or affected service requires activation.

`dnf update --security` installs the latest available versions of packages that have applicable security errata. `dnf update-minimal --security` instead installs the minimum package versions that resolve the applicable errata. An organisation should choose between those approaches through its patch policy rather than by command-line convenience.

Severity filtering supports risk-based prioritisation, but severity alone cannot express local exposure. Administrators should also consider whether the vulnerable component is installed, enabled, reachable, or protected by another control. Red Hat advisory identifiers provide a stable unit for change records, maintenance approvals, and verification.

Before an update, the host should have enough free space, a recoverable configuration, and a tested path to restart affected workloads. After the update, administrators should review the transaction, service health, relevant logs, and the running kernel version. Automated updates can use `dnf-automatic`, but production policy should define download behaviour, installation windows, exclusions, notifications, and reboots.
### Application Streams
Module commands show the content available on the current host:

```bash
$ dnf module list
$ dnf module info NAME:STREAM
$ sudo dnf module install NAME:STREAM/PROFILE
$ sudo dnf module reset NAME
```

The colon separates a module name from its stream, and the slash separates a stream from its profile. If the command omits a stream or profile, DNF uses an applicable default. A reset removes the stream selection but does not remove installed packages. Stream changes can alter dependencies, so administrators should follow the documented switching procedure and confirm that the destination stream supports the application.

Enabling a stream and installing its packages are separate operations. `dnf module enable NAME:STREAM` selects a stream without necessarily installing its profile. `dnf module remove NAME:STREAM/PROFILE` removes packages associated with an installed profile, subject to dependency review. `dnf module disable NAME` prevents DNF from using all streams of that module. Administrators should always inspect the transaction because modular and non-modular packages can share dependencies.
## Configuring time services
Accurate time supports log correlation, authentication, certificate validation, clustered services, and incident analysis. RHEL 8 implements NTP through `chronyd`, supplied by the `chrony` package. RHEL 8 does not support the former `ntpd` implementation.
### Time zones, clocks, and services
The following commands install and activate chrony, display the current time configuration, and set a time zone:

```bash
$ sudo dnf install chrony
$ sudo systemctl enable --now chronyd
$ timedatectl
$ timedatectl list-timezones
$ sudo timedatectl set-timezone Australia/Sydney
```

Linux systems should normally keep the hardware real-time clock in UTC and derive local civil time from the configured time zone. This arrangement handles daylight-saving changes without rewriting the hardware clock:

```bash
$ sudo timedatectl set-local-rtc 0
```

`timedatectl set-timezone` manages `/etc/localtime`. Administrators should not routinely replace that link by hand. If the command fails, they should investigate the error, confirm that the time-zone database exists, and repair the underlying configuration.

`systemctl` controls both the current service state and its boot-time enablement:

```bash
$ systemctl status chronyd
$ sudo systemctl restart chronyd
$ sudo systemctl disable --now chronyd
$ sudo systemctl enable --now chronyd
$ systemctl cat chronyd
```

`systemctl disable` alone changes boot-time enablement but does not stop a running service. The `--now` option also changes the current state. `systemctl cat` shows the vendor unit and any overrides. Administrators should normally use `systemctl edit chronyd` to create a focused drop-in override. `systemctl edit --full` creates a complete replacement under `/etc/systemd/system/`, which can hide later vendor-unit changes and therefore requires deliberate maintenance.
### Chrony configuration and verification
RHEL stores the main chrony configuration in `/etc/chrony.conf`. A client can use one named server or a DNS pool:

```text
server time.example.com iburst
pool 2.rhel.pool.ntp.org iburst
```

The `iburst` option accelerates initial synchronisation when a source becomes reachable. Administrators should select trusted sources that suit the organisation's network, geography, and security requirements. Network Time Security can authenticate supported NTP sources where the deployment requires it.

Comments in the packaged configuration explain important defaults and provide operational context. Removing every comment and blank line adds little value and can make later maintenance harder. Administrators should back up the file, change only the required directives, and use a configuration-management template or targeted edit when many hosts need the same settings. They should then restart `chronyd` and verify the result:

```bash
$ sudo systemctl restart chronyd
$ chronyc tracking
$ chronyc sources -v
$ timedatectl
```

`chronyc tracking` reports the selected reference, system offset, frequency correction, leap status, and stratum. `chronyc sources -v` reports all candidate sources and their selection states. A stratum 1 server connects directly to a reference source. Stratum 16 indicates an unsynchronised source, not a usable tier. The interactive `chronyc` shell exposes the same commands, but direct subcommands suit scripts and routine checks.

Several common directives shape client behaviour. `driftfile` stores the measured clock-frequency error so chrony can compensate after a restart. `makestep` permits a large initial correction during a limited number of early updates. `rtcsync` asks the kernel to copy system time to the hardware clock periodically. Administrators should retain suitable vendor defaults unless a documented requirement justifies a change.

A host that supplies NTP to other systems needs an `allow` directive for the authorised client network and firewall access to UDP port 123. The default client configuration does not grant unrestricted server access. Administrators should limit permitted networks, prefer authenticated sources where available, and avoid exposing an internal time service to the public internet.

Immediately after a restart, `chronyc tracking` can show an unsynchronised state while chrony samples and selects sources. Persistent failure requires checks of DNS, routing, firewall policy, source reachability, configuration syntax, and service logs. `journalctl -u chronyd` provides the service record.
## Working with systemd targets
A systemd target groups units into an operational state. Targets replace the central role that SysV runlevels held in earlier RHEL releases, although compatibility commands still map common targets to runlevel numbers. `multi-user.target` broadly corresponds to runlevel 3, and `graphical.target` broadly corresponds to runlevel 5.

Several targets can remain active because a high-level target pulls in supporting targets. Administrators can list active targets and inspect the configured default:

```bash
$ systemctl list-units --type=target --state=active
$ systemctl get-default
$ systemctl cat multi-user.target
```

Targets express dependencies through directives such as `Wants=` and `Requires=`, and ordering through directives such as `After=` and `Before=`. A target does not contain a linear script of services. Systemd builds a dependency graph, starts independent work concurrently, and records each unit's state.

`systemctl list-dependencies multi-user.target` shows the units pulled into a target. `systemctl is-enabled NAME.service` reports boot-time enablement, while `systemctl is-active NAME.service` reports current state. These properties differ. A service can run while disabled, or remain stopped while enabled for the next boot.

Administrators can manage ordinary services with `start`, `stop`, `restart`, `reload`, `enable`, and `disable`. `reload` asks a service to reread its configuration without a full restart and works only when the unit implements that action. `restart` stops and starts the service, which can interrupt clients. `enable --now` combines boot-time enablement with an immediate start.

The default target controls a normal boot when no kernel argument overrides it:

```bash
$ sudo systemctl set-default multi-user.target
```

`set-default` changes later boots but does not change the current state. `isolate` starts the named target and stops units that the target does not require:

```bash
$ sudo systemctl isolate graphical.target
```

Isolation can close sessions, stop network services, or disrupt workloads. An administrator should confirm console or recovery access before isolating a target on a remote or production host.

The `grubby` utility displays boot entries and edits their kernel arguments:

```bash
$ sudo grubby --info=ALL
$ sudo grubby --update-kernel=ALL --args="systemd.unit=graphical.target"
$ sudo grubby --update-kernel=ALL --remove-args="systemd.unit"
```

The correct kernel parameter is `systemd.unit`, with a full stop between the words. A temporary GRUB edit can apply the parameter to one boot. A persistent change to `ALL` affects every current kernel entry, so the default target usually provides the simpler system-wide choice. A distinct GRUB entry suits a genuine alternative boot path.
## Scheduling jobs
RHEL 8 provides three principal scheduling mechanisms. `at` runs one-off jobs, cron runs compact recurring schedules, and systemd timers integrate recurring or boot-relative work with unit dependencies and the journal.

| Scheduler | Best fit | Main inspection command |
| --- | --- | --- |
| `at` | One-off work at a specified time | `atq` |
| cron | Simple, recurring calendar schedules | `crontab -l` or file inspection |
| systemd timer | Recurring or boot-relative work that needs unit controls and journal integration | `systemctl list-timers --all` |

The scheduler starts a command, but the command still needs safe locking, error handling, idempotent behaviour where appropriate, and useful output. A scheduler should not launch a second copy of a job that cannot run concurrently. The job can use an application lock, `flock`, or a service design that rejects overlapping execution.
### One-off jobs with at
The `at` package supplies the `at` client and the `atd` service:

```bash
$ sudo dnf install at
$ sudo systemctl enable --now atd
$ at 17:00 tomorrow
at> /usr/local/sbin/archive-data >>/var/log/archive-data.log 2>&1
at> <Ctrl-D>
```

`at` accepts natural time expressions and reads one or more commands until end-of-file. It runs the saved job non-interactively, so commands should use absolute paths, explicit redirection, and any required environment settings.

Administrators and users can inspect or remove queued jobs:

```bash
$ atq
$ at -c 4
$ atrm 4
```

`/etc/at.allow` and `/etc/at.deny` control user access. If `at.allow` exists, only listed users may submit jobs. Otherwise, `at.deny` excludes listed users. If neither file exists, only root may use `at`. Root retains administrative control.

`at -c JOB_ID` displays the generated job script, including the captured environment and working directory. This output helps confirm the exact command but can also expose environment values, so administrators should handle it carefully. Secrets should not appear on a command line or in a saved job body.
### Recurring jobs with cron
The `crond` service reads system schedules from `/etc/crontab` and `/etc/cron.d/`, and user schedules managed by `crontab`. A system entry contains six scheduling and identity fields before the command:

| Field | Valid form |
| --- | --- |
| Minute | `0` to `59` |
| Hour | `0` to `23` |
| Day of month | `1` to `31` |
| Month | `1` to `12`, or a supported name |
| Day of week | `0` to `7`, with Sunday as `0` or `7`, or a supported name |
| User | Account that runs a system job |

This system schedule runs a backup at 07:15 each Saturday:

```cron
15 7 * * 6 root /usr/local/sbin/weekly-backup
```

A user crontab omits the user field because the owner supplies the execution identity:

```bash
$ EDITOR=nano crontab -e
$ crontab -l
$ crontab -r -i
```

An every-five-minutes expression uses `*/5`, not `*-5`:

```cron
*/5 * * * * /usr/local/bin/check-space
```

When both day-of-month and day-of-week contain restricted values, traditional cron runs the job when either field matches. A job that requires both conditions should test the second condition in a script or use a systemd calendar expression that states the intended date.

Files in `/etc/cron.d/` should use simple names containing letters, digits, hyphens, and underscores. A name such as `sales.cron` can be ignored because of its full stop. System cron files should belong to root, should not permit writes by untrusted users, and should include the execution user. Jobs should use absolute paths because cron supplies a limited environment. Administrators should also confirm that `crond` runs and should direct output to a log, monitoring system, or mail destination.

`/etc/cron.hourly/` contains executable scripts that `run-parts` invokes. RHEL commonly uses Anacron to run daily, weekly, and monthly work that does not require an exact clock time. Anacron can run missed periodic work after a host returns to service, while a plain cron entry normally loses an activation that occurred during downtime. Script filenames in these directories must satisfy the `run-parts` naming rules.

A crontab can define variables such as `SHELL`, `PATH`, and `MAILTO` before job entries. Explicit values reduce differences between an interactive shell and the scheduled environment. Commands should send failures to a monitored destination, and administrators should test the underlying script directly before testing the schedule.
### Recurring jobs with systemd timers
A systemd timer activates a service unit with the same base name unless `Unit=` names another unit. Calendar timers use `OnCalendar=`, while monotonic timers use settings such as `OnBootSec=` or `OnUnitInactiveSec=`. `Persistent=true` causes an eligible missed calendar activation to run after downtime. `RandomizedDelaySec=` spreads load across hosts.

Calendar expressions can describe weekdays, dates, and times. For example, `OnCalendar=Mon..Fri 03:30` runs on weekdays at 03:30. Administrators can validate an expression before installing a unit:

```bash
$ systemd-analyze calendar 'Mon..Fri 03:30'
```

Monotonic timers measure time from an event. `OnBootSec=10m` schedules an activation ten minutes after boot, while `OnUnitInactiveSec=1h` schedules an hour after the activated service last became inactive. Combining calendar and monotonic settings can create more than one trigger, so each directive needs an intentional purpose.

For example, `cache-report.service` defines the work:

```ini
[Unit]
Description=Generate cache report

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/cache-report
```

`cache-report.timer` defines the schedule:

```ini
[Unit]
Description=Run the cache report daily

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=10m
Unit=cache-report.service

[Install]
WantedBy=timers.target
```

Both files belong in `/etc/systemd/system/`. The administrator then reloads the manager, enables the timer, and checks its schedule and service log:

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now cache-report.timer
$ systemctl list-timers --all
$ systemctl status cache-report.timer
$ journalctl -u cache-report.service
```

`systemctl list-timers --all` shows the last and next activations, the timer unit, and the service it triggers. The journal records service output and failures. These features make timers suitable when jobs need dependency ordering, missed-run handling, central logs, resource controls, or clear operational status.

Enabling a timer does not enable its service as a normal boot service. The timer activates the service at the scheduled point. A timer does not restart a service that remains active when the activation occurs, so a repeatable oneshot service should normally become inactive after completion. Administrators should test both units, inspect `systemctl status`, and confirm a successful second activation.