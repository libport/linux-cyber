# Using Ansible Roles
Ansible roles package related automation into reusable units. They standardise common tasks, reduce duplication, and keep playbooks concise. A role separates tasks, variables, handlers, templates, files, and metadata into a fixed directory layout, so Ansible can load each component automatically.
## Role structure
A conventional role includes these directories:
- `defaults` for low-precedence variables that other variables may override
- `vars` for variables that should not normally be overridden
- `tasks` for the main task list
- `handlers` for triggered actions such as service restarts
- `files` for static files
- `templates` for Jinja2 templates
- `meta` for metadata and role dependencies
- `tests` for optional inventory and test playbooks

Each of these directories commonly contains a `main.yml` entry point.
## Role locations and execution order
Roles are commonly stored in a project `roles/` directory, a user role directory such as `~/.ansible/roles`, or a system role path such as `/etc/ansible/roles` or `/usr/share/ansible/roles`. Project roles take precedence over user and system roles.

Within a play, Ansible runs `pre_tasks`, then roles, then regular `tasks`, then `post_tasks`. Handlers run at the end of the play unless a playbook flushes them earlier. This order matters when a role depends on preparation work or when follow-up tasks must run after the role.
## Creating and using a custom role
`ansible-galaxy init <role_name>` creates the standard role skeleton. A focused role should do one job well and accept input through variables instead of hard-coded values.

A simple message-of-the-day role can render `/etc/motd` from a template and expose a default contact variable. A playbook can then include the role and override that variable for a specific host.

```yaml
---
- name: Apply the motd role
  hosts: ansible2
  roles:
    - role: motd
      system_manager: bob@example.com
```

A well-designed role stays generic, keeps secrets out of the role itself, and stores sensitive data in Ansible Vault. Version control supports review, reuse, and consistent change management.
## Dependencies and project organisation
A role can declare dependencies in `meta/main.yml`. Ansible runs dependent roles before the parent role and avoids running the same dependency more than once in a play. Dependencies can also receive variables and conditional logic.

```yaml
dependencies:
  - role: apache
    port: 8080
  - role: mariadb
    when: environment == 'production'
```

Larger environments benefit from project directories that keep `ansible.cfg`, inventory, playbooks, and variables together. Common practice uses `site.yml` as the top-level playbook, `group_vars/` and `host_vars/` for host-specific data, and separate inventories for staging and production.
## Using Ansible Galaxy
Ansible Galaxy provides community-maintained roles and collections. A collection is a broader distribution format that may include roles, playbooks, modules, and plugins. For most playbooks, the role is the main reusable unit.

Useful Galaxy commands include:
- `ansible-galaxy search` to find roles
- `ansible-galaxy info` to inspect a role
- `ansible-galaxy install` to install a role
- `ansible-galaxy list` to list installed roles
- `ansible-galaxy remove` to remove a role

Search results become more useful when filtered by platform, author, or Galaxy tags. Download counts, quality signals, and tags help distinguish mature roles from less established ones.

A requirements file supports repeatable installs and version pinning.

```yaml
- src: geerlingguy.nginx
  version: "2.7.0"
```

Installing from a requirements file uses `ansible-galaxy install -r <file>`. Roles may also come from Git repositories or tarballs when the source URL and, for Git, `scm: git` are specified.
## RHEL System Roles
RHEL System Roles provide supported roles for common operating system configuration tasks such as networking, storage, SELinux, time synchronisation, Postfix, and kdump. They install into the standard Ansible role path and include local documentation and sample playbooks.

The SELinux role uses variables to describe the desired state, including policy, mode, booleans, file contexts, restore paths, ports, and login mappings. File-context changes usually require both `selinux_fcontexts` and `selinux_restore_dirs`.

```yaml
vars:
  selinux_policy: targeted
  selinux_state: enforcing
  selinux_fcontexts:
    - { target: '/web(/.*)?', setype: 'httpd_sys_content_t', ftype: 'd' }
  selinux_restore_dirs:
    - /web
```

Some SELinux changes require a reboot. A robust playbook handles that case explicitly, reboots the managed host, waits for reconnection, and reapplies the role.

The TimeSync role configures time sources through `timesync_ntp_servers`. It abstracts underlying implementation differences so the same role can manage supported RHEL systems consistently. Time zone configuration remains a separate task in the playbook.
## Practical guidance
Effective roles remain small, composable, documented, and broadly reusable. They avoid embedding secrets, prefer clear variable names, and expose only the configuration surface that administrators are expected to change. That approach keeps playbooks shorter, promotes consistent system state, and makes Ansible automation easier to test and maintain.