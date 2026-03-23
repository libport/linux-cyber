# Managing Services and the Boot Process
Ansible manages service state, scheduled jobs, boot targets, and reboots with a small set of focused modules. The generic `service` module works across several init systems. `service_facts` collects service state information. Systemd-specific behaviour such as `daemon_reload` and `masked` belongs in `ansible.builtin.systemd_service`.
## Managing services
Use the generic service interface for portable playbooks across mixed environments. Use the systemd-specific module when a playbook must reload unit files, mask a unit, or work with unit-specific behaviour that the generic module does not expose clearly.

```yaml
- name: Start and enable httpd
  ansible.builtin.systemd_service:
    name: httpd
    enabled: true
    state: started
    masked: false
    daemon_reload: true
```
## Scheduling jobs
`cron` manages recurring jobs. Ansible identifies each cron entry by `name`, so every managed job needs a unique name. Without `user`, the module edits the current user's crontab. `special_time` can express nicknames such as `daily` or `reboot` when a fixed schedule is unnecessary. A valid trim example uses `fstrim -a`, not bare `fstrim`.

```yaml
- name: Run trim twice daily
  ansible.builtin.cron:
    name: run fstrim
    minute: "5"
    hour: "4,19"
    job: "fstrim -a"
```

The `at` module schedules a one-off job in the future. It uses `count` with `units`, and it supports `state: present` and `state: absent`. `unique: true` prevents duplicate queued jobs and keeps one-off automation idempotent.

```yaml
- name: Run a command once in five minutes
  ansible.posix.at:
    command: "date > /tmp/my-at-file"
    count: 5
    units: minutes
    unique: true
    state: present
```
## Managing boot behaviour
Ansible does not provide a dedicated module for changing the default boot target. Systemd stores that setting through the `default.target` alias, so a playbook can manage the symlink directly with `file`. In practice, that usually points to `multi-user.target` or `graphical.target`.

```yaml
- name: Set the default boot target
  ansible.builtin.file:
    src: /usr/lib/systemd/system/graphical.target
    dest: /etc/systemd/system/default.target
    state: link
```

The `reboot` module restarts a host and resumes the play when the machine becomes reachable again. `test_command` checks readiness after boot and defaults to `whoami` if omitted. `pre_reboot_delay`, `post_reboot_delay`, `connect_timeout`, and `reboot_timeout` control how patiently the play waits for shutdown and recovery.

```yaml
- name: Reboot the host
  ansible.builtin.reboot:
    msg: reboot initiated by Ansible
    test_command: whoami
```