# Deploying, Configuring and Maintaining Systems
> [!NOTE]
> This file is a practical RHEL administration guide that covers software and repository management with YUM/DNF and AppStream modules, time configuration with Chrony, systemd targets and boot behaviour, and job scheduling with `at`, `cron`, and systemd timers.
## Software Management in RHEL
RHEL uses YUM as the familiar front end for software management, but YUM is backed by DNF technology. In practice, administrators can use either `yum` or `dnf` for common package tasks. The important point is consistency rather than command tribalism. Package work is repository-driven, transaction-based, and closely tied to system subscriptions.

Two repositories matter most in a standard build:
- `BaseOS` provides the core operating system content in RPM format.
- `AppStream` provides additional user-space content in RPMs and modules.

This split changes how software is selected. Traditional RPM packaging remains suitable for standard installs, removals, and queries. AppStream adds modular content, which allows several supported versions of a component to coexist as streams. A module can also expose profiles so that only the required subset of packages is installed. That is useful when an administrator wants a client package without a full server stack, or a specific language runtime without whatever happens to be the default.
### Core Package Operations
Routine package work follows a clear pattern. An administrator can query packages, identify which package provides a binary, install software, remove software, inspect transaction history, and undo a transaction when a change causes trouble.

Useful patterns include:
- `yum list <package>` to check whether a package is installed or merely available
- `yum provides <path-or-command>` to find the package behind a file or executable
- `yum install <package>` to add software
- `yum remove <package>` to remove software
- `yum history list` and `yum history undo <id>` to review and reverse transactions

Standard users can perform many read-only queries because repository metadata is local once cached. Privileged access is still required for installation, removal, enabling repositories, and any change that affects the system state.

Transaction history is one of the more valuable administrative safeguards. Package installs often pull in dependencies, and a later rollback can reverse a whole transaction more cleanly than manually removing related packages one by one. That same history mechanism is useful after a broad update if a service stops working and a quick retreat is safer than troubleshooting under pressure.

Administrative queries also become more effective when they start with metadata rather than guesswork. `yum provides` links files and executables back to packages, which helps when a missing command appears in a script or an unfamiliar binary turns up on a host. `yum list installed`, `yum list available`, and repository queries give a quick inventory of what is already present and what remains selectable. On production systems, that reduces unnecessary change because an administrator can verify state before reaching for an install command.

One important correction is necessary here. RHEL documentation describes package removal through `yum remove`, package group removal, and modular content removal. It does not present `yum purge` as a standard RHEL package-management command. The practical focus therefore stays on `remove`, modular removal, and transaction rollback.
### Repositories and Subscription Awareness
Repository access determines what can be installed and updated. On subscribed RHEL systems, repository definitions sit under `/etc/yum.repos.d/`, with Red Hat-managed content exposed through files such as `redhat.repo`. `yum repolist` shows enabled repositories, while `yum repolist --all` also exposes disabled entries. That broader view matters when a repository exists but needs to be enabled before use.

A subscription-backed system normally relies on online repositories. A disconnected or controlled environment can instead build a local repository from installation media. The usual method mounts the RHEL DVD or ISO, exposes the `BaseOS` and `AppStream` directories, and adds them through `yum config-manager --add-repo`. This approach works, but it has an obvious limitation. Media-based repositories provide the packaged content that ships on the image and do not deliver ongoing security fixes.

A better shared option is a local web repository. The same DVD content can be mounted under an HTTP document root such as `/var/www/html`, served with Apache, and published to other hosts over the network. This reduces external traffic and supports consistent package sources across a fleet. It still inherits the same limitation if the source is static installation media. Local convenience is not the same as live patch coverage.
### Security Updates and Update Selection
RHEL package metadata supports targeted updates. Administrators can inspect update information, filter security advisories, and act on severity rather than updating indiscriminately. That matters in tightly controlled environments where change windows are short and security exposure is prioritised.

Useful commands include:
- `yum updateinfo` to list available update classes
- `yum updateinfo list security` to focus on security notices
- `yum update --advisory <RHSA-id>` to install fixes tied to a specific advisory
- `yum update --sec-severity=Important` to apply only updates at a chosen security severity

This method supports a more disciplined patching model. A team can respond to a single RHSA notice, apply only high-severity fixes, or inspect which packages an advisory affects before committing to change. When a larger update introduces instability, `yum history undo` provides a controlled rollback path.

The same selective approach is useful during maintenance windows. A broad `yum update` may still be appropriate for standard lifecycle maintenance, but advisory-driven updates fit environments where approval is tied to a security bulletin or where an application owner wants the smallest practical change set. RHEL makes that possible without abandoning the normal package toolchain.

Kernel handling deserves a brief distinction. RHEL generally installs new kernel packages alongside existing ones rather than replacing the running kernel in place. That preserves fallback boot entries and aligns with safer recovery practice.
### Modules, Streams, and Profiles
Modules extend package management beyond a single version track. A stream represents a supported version line of an application stack. A profile defines the package set within that stream. Administrators use `yum module list` to examine what is available and `yum module install <module>:<stream>/<profile>` to choose an explicit combination.

This matters when an application depends on a specific runtime version, such as a particular Ruby stream, or when database tooling should include only the client side. A default stream can simplify routine deployment, but explicit stream selection gives predictable results and makes builds more reproducible.
## Time Configuration and Synchronisation
Accurate time is operationally critical. Authentication systems, logs, certificates, job schedulers, and distributed services all rely on clocks that agree. RHEL uses `chronyd` from the `chrony` package as its NTP implementation. It replaces the older emphasis on `ntpd` while still using the NTP protocol.

Chrony suits modern systems well. It synchronises quickly, handles unstable or intermittent connectivity better than older approaches, and copes more gracefully with systems that suspend, resume, or carry uneven load. The package is `chrony`, the daemon is `chronyd`, and the command-line client is `chronyc`.
### Checking and Setting Time
`timedatectl` is the central administrative tool for date, time, timezone, and clock status. It can display the current local time, UTC, RTC setting, timezone, and synchronisation state. It also sets the timezone cleanly with `timedatectl set-timezone <zone>`.

A sound RHEL configuration keeps the hardware clock in UTC and applies the timezone as an offset for local display. That avoids daylight saving problems and keeps behaviour predictable across regions and dual-boot or virtualised environments. If the RTC is set to local time, `timedatectl` warns about it. Administrators should correct that instead of adjusting the hardware clock twice each year.

The distinction between local time and UTC should stay conceptually clear. Users and logs may display local time for readability, but service coordination depends on a stable underlying clock. When hosts move between regions, snapshots are restored, or virtual machines are cloned, a UTC-based RTC prevents a long tail of confusing offsets and daylight saving errors.

Timezone selection is usually simple because `timedatectl list-timezones` exposes the valid names and `timedatectl set-timezone` applies the chosen zone. Manual relinking of `/etc/localtime` should be treated as a fallback or troubleshooting step, not the normal first option.
### Managing Chrony as a Service
Chrony behaves like any other systemd-managed service. The minimum checks are whether it is installed, running, and enabled for boot. The common commands are:
- `yum list chrony`
- `systemctl status chronyd`
- `systemctl enable --now chronyd`
- `systemctl disable --now chronyd`
- `systemctl restart chronyd`

`systemctl cat chronyd` reveals the current unit definition and any local overrides. `systemctl edit` creates an override, while `systemctl edit --full` creates a full local copy of the unit file under `/etc/systemd/system/`. That distinction matters because a drop-in is usually cleaner than copying a complete service unit unless a full replacement is truly necessary.

Chrony configuration lives in `chrony.conf`. The file often carries many comments and blank lines, which can obscure active settings during administration at scale. A small `sed` script can strip comments, remove empty lines, insert a preferred local or regional time source, and delete obsolete defaults. This is a practical example of repeatable configuration hygiene. The aim is not cleverness for its own sake. The aim is a predictable file state across many systems.
### Monitoring Synchronisation
`chronyc` exposes the live state of the time client. `chronyc tracking` shows the current reference source, offset, frequency correction, and overall synchronisation information. `chronyc sources` shows the candidate time sources in use, and `chronyc sources -v` explains the columns more fully.

This gives administrators immediate answers to practical questions:
- Is the host synchronised
- Which source is currently preferred
- Which fallback sources are reachable
- How far off is the system clock

That visibility is valuable during incident work. It distinguishes a bad clock from an application issue and shows whether a host is drifting, isolated, or simply waiting for a better source. It also gives quick evidence when a host has network reachability but still lacks a usable time source, which is a different failure from a service that is stopped or disabled.
## Systemd Targets and Boot Behaviour
Systemd targets define system states through groups of related units. They replace the older runlevel model with names that better reflect intent, such as `multi-user.target`, `graphical.target`, `rescue.target`, and `emergency.target`.

A target is not a single service. It is a grouping and synchronisation point. When a system enters a target, systemd starts the units that target requires and the targets beneath it. That is why several targets can appear active at once. A graphical system, for example, also includes the services associated with multi-user operation.
### Reading Current and Default Targets
Legacy tools still exist. `runlevel` can display a translated view for administrators who learned earlier RHEL versions. The more accurate modern approach is through `systemctl`:
- `systemctl list-units --type target --state active` shows active targets
- `systemctl get-default` shows the default target for boot
- `systemctl cat <unit>` shows how a service is attached to a target

A service such as `chronyd` can be wanted by `multi-user.target`, which illustrates the purpose of targets. The kernel and init system do not need to name every service individually. They need to reach the correct target, and the dependency chain does the rest.
### Changing the Default or Current Target
Default and current targets solve different problems.

`systemctl set-default <name>.target` changes what the system will use on the next boot. It updates the `default.target` symlink under `/etc/systemd/system/`. The current session does not switch immediately unless the system is rebooted or the default target is isolated.

`systemctl isolate <name>.target` changes the target for the current boot. Systemd starts the new target and its dependencies and stops services that the new target does not require. This is useful when moving between graphical and command-line states or when entering a reduced environment for maintenance.

A concise rule helps here:
- Use `set-default` to change future boots.
- Use `isolate` to change the running system now.

`rescue.target` and `emergency.target` serve different recovery needs. Rescue mode provides a single-user environment with more of the base system mounted and available. Emergency mode is more minimal and is reserved for harder failure cases. Both are maintenance tools, not everyday operating states.

This distinction matters during boot failures and repair work. A host that still mounts local filesystems and starts essential services can often be repaired in rescue mode. A host with deeper startup problems may need emergency mode so that an administrator can intervene before more of the system stack comes online. Choosing the lighter or heavier mode should follow the severity of the failure, not habit.
### Kernel Arguments and grubby
Targets can also be selected through kernel arguments. `grubby --info=ALL` lists GRUB boot entries and their current arguments. Administrators can add or remove arguments across all entries or a selected kernel entry.

That is useful when a system should boot into a specific state without changing the long-term default. A graphical target can be forced for certain entries, while another entry can remain command-line only. This is safer than constantly rewriting the default target on hosts that need alternate boot paths for recovery or administrative work.

The corrected principle here is that `grubby` changes GRUB entry arguments, while `systemctl set-default` changes the systemd default target symlink. They are related but not interchangeable. One adjusts boot entry parameters. The other changes the systemd default destination after boot begins.
## Scheduling Work in RHEL
RHEL offers three practical schedulers in this scope:
- `at` for one-off work
- `cron` for recurring work in traditional Unix style
- systemd timer units for recurring work with better service integration and status visibility

Choosing the right scheduler is mostly about intent. A command that should run once next Tuesday belongs in `at`. A nightly rotation or weekly report belongs in `cron` or a timer unit. Administrators should not treat the tools as rivals. They solve slightly different problems.
### One-Off Scheduling with at
The `at` package provides both the `at` command and the `atd` daemon. If it is not installed, it must be added and the service enabled. Access is governed by `/etc/at.allow` and `/etc/at.deny`. If `at.allow` exists, only listed users may schedule jobs. If it does not exist, users not denied in `at.deny` may use the scheduler.

`at` opens an interactive prompt for commands to be executed at a chosen time. After entering one or more commands, `Ctrl+D` ends input and queues the job. Administrators can then:
- list jobs with `atq`
- inspect a queued job with `at -c <id>`
- remove a job with `atrm <id>`

This is well suited to ad hoc administrative work such as deferred file copies, a one-time report run, or a temporary corrective command that should execute outside business hours.
### Recurring Scheduling with Cron
Cron remains the classic recurring scheduler. Its system-wide configuration is built around `/etc/crontab`, the extension directory `/etc/cron.d`, and convenience directories such as `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, and `/etc/cron.monthly`.

System cron entries define schedule fields, the user account that should run the job, and the command. User crontabs differ in one key way. They omit the user field because the crontab already belongs to a specific user.

The schedule fields are:
- minute
- hour
- day of month
- month
- day of week

Administrators can use numbers, ranges, lists, or wildcard patterns to express timing. That flexibility supports fine-grained schedules, but it also makes mistakes easy. Reading `/etc/crontab` and the `crontab(5)` manual page remains one of the quickest ways to confirm the exact field order before committing a change.

User cron access is controlled by `/etc/cron.allow` and `/etc/cron.deny`. The `crontab` command manages the per-user file:
- `crontab -e` edits it
- `crontab -l` lists it
- `crontab -r` removes it

Editor choice can matter in mixed teams. If the default editor is not suitable, an `EDITOR` environment variable can override it for a single command or persistently through shell configuration.

The practical difference between system and user cron entries should stay explicit. A file under `/etc/cron.d` behaves like system policy and names the user account that runs the job. A personal crontab behaves like user-owned automation and inherits the user identity automatically. Mixing those two models carelessly can produce permission problems or run work under the wrong account.
### Systemd Timer Units
Timer units provide a more service-oriented scheduling model. A timer activates a corresponding service unit, which means the scheduled action is tied directly into systemd. The configuration lives in timer unit files rather than in the cron field syntax.

The main administrative views are:
- `man 5 systemd.timer` for format details
- `systemctl list-unit-files --type=timer` to see timer definitions
- `systemctl cat <timer>.timer` to inspect timer contents
- `systemctl status <service>.service` to inspect the service the timer triggers
- `systemctl list-timers --all` to see when timers last ran and when they will run next

The built-in `dnf-makecache.timer` is a useful example. It refreshes package metadata after boot and then on a recurring interval. By inspecting the timer and its paired service, an administrator can tell when metadata last refreshed and when the next run is due.

That visibility is where timer units stand out. Cron can schedule work effectively, but `systemctl list-timers --all` provides a clearer operational dashboard. Timer units suit administrators who want schedules integrated with service management, dependency handling, and systemd reporting.
## Operational Patterns That Matter
Across all four areas, the same administrative habits keep RHEL manageable.

Choose explicit state over assumption. Select the required module stream instead of accepting a default. Set the timezone directly instead of inferring it from an image. Confirm the default target rather than assuming the host boots to the expected state. Inspect queued and recurring jobs instead of trusting memory.

Prefer reversible changes. Package history, advisory-scoped updates, service enablement through systemd, and isolated target changes all support controlled rollback. Where a host needs alternate operating modes, separate GRUB entries or explicit timer units are clearer than fragile one-line edits made under pressure.