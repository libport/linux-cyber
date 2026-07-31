# Managing Services and the Boot Process on RHEL 10
Red Hat Enterprise Linux 10 uses systemd to manage services, scheduled work, boot targets, and shutdowns. Ansible can maintain this state consistently across managed hosts.
## Managing systemd services
`ansible.builtin.systemd_service` is the preferred module for systemd units. Its earlier name, `ansible.builtin.systemd`, remains an alias. `ansible.builtin.service` provides a generic interface for different init systems, but it exposes fewer systemd-specific controls.

| Requirement | Ansible interface |
| --- | --- |
| Start, stop, enable, disable, mask, or unmask a unit | `ansible.builtin.systemd_service` |
| Control services through a generic init-system interface | `ansible.builtin.service` |
| Collect service state and enablement facts | `ansible.builtin.service_facts` |
| Manage units, unit files, and drop-ins at scale | `redhat.rhel_system_roles.systemd` |

```yaml
- name: Keep the web service enabled and running
  ansible.builtin.systemd_service:
    name: httpd.service
    enabled: true
    state: started
```

The `started` and `stopped` states are idempotent. The `restarted` and `reloaded` states always act, and `reloaded` starts an inactive unit. Setting `masked: true` prevents any start operation, while `masked: false` removes the mask. A task should set `daemon_reload: true` only after changing a unit file or drop-in, commonly through a handler.

`service_facts` must run separately from normal fact gathering. It records properties such as `state` and `status`. Bracket notation safely accesses names that contain punctuation:

```yaml
ansible_facts.services['sshd.service'].state
```

Service facts do not report package versions. `ansible.builtin.package_facts` supplies that information.
## Scheduling work
Systemd timers provide the native RHEL 10 scheduler for service-aware tasks. A timer activates a matching service unit. `OnCalendar` defines calendar schedules, `Persistent=true` catches up a missed calendar event after downtime, and a randomised delay can spread work across a fleet. Ansible can deploy the unit and timer files, then enable and start the timer with `systemd_service` or the RHEL system role.

`ansible.builtin.cron` remains suitable for established cron workloads. It requires a compatible implementation such as `cronie`, but the module does not install the package or start `crond`. Each job needs a unique `name`, which Ansible stores in a `#Ansible:` marker. Reusing the name updates the job, and `state: absent` removes it. `special_time: reboot` represents an `@reboot` entry. A system-wide `cron_file` also requires a `user`.

Cron normally runs commands through `/bin/sh`, unless its environment selects another shell. A literal `%` in a command requires escaping. A scheduled command should call `/usr/bin/date` or `logger` when it needs its actual execution time because `ansible_date_time` reflects the earlier fact-gathering instant.

Historical `at` automation depends on an external collection and the host's `at` service. A systemd one-shot or transient timer usually provides a better RHEL 10 design.
## Configuring the default boot target
Systemd reads `/etc/systemd/system/default.target` to select the default boot target. Ansible can manage this path as a symbolic link to `/usr/lib/systemd/system/multi-user.target` for a text-oriented server or `graphical.target` for a graphical system. The `ansible.builtin.file` module with `state: link` keeps the link idempotent.
## Rebooting managed hosts
`ansible.builtin.reboot` initiates a restart, waits for the host to disconnect and return, verifies that the boot identifier changed, and runs a validation command. The default validation command is `whoami`, while an application readiness check provides stronger assurance.

`reboot_timeout` defaults to 600 seconds and applies separately to reboot verification and post-boot validation. `connect_timeout` limits each connection attempt. On Linux, `pre_reboot_delay` converts to whole minutes, so values below 60 seconds become zero. `post_reboot_delay` pauses before validation. A custom `reboot_command` bypasses the normal message and delay handling.

Production fleets should reboot in controlled batches with `serial`, drain affected nodes, validate application health, and restore traffic before continuing. A handler can restrict reboots to hosts whose configuration changed. Unconditional cron-based reboots should not replace controlled maintenance.