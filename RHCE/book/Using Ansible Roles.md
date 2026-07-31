# Using Ansible Roles
Ansible roles organise related tasks, handlers, variables, files, templates, and metadata into reusable units. A role can configure one service or implement a focused system function. This structure reduces duplication, supports testing, and keeps playbooks concise.
## Role structure
Ansible loads conventional files named `main.yml` from standard subdirectories. A role needs only the directories that its work requires.

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Defines the tasks that implement the role |
| `handlers/main.yml` | Defines handlers notified by tasks |
| `defaults/main.yml` | Provides easily overridden variables at very low precedence |
| `vars/main.yml` | Provides high-precedence role variables |
| `files/` | Stores static files for modules such as `ansible.builtin.copy` |
| `templates/` | Stores Jinja templates for `ansible.builtin.template` |
| `meta/main.yml` | Records metadata and role dependencies |
| `tests/` | Holds inventory and playbooks for basic role testing |

Large roles can divide work into additional task files and load them from `tasks/main.yml`. This separation should follow coherent functions rather than arbitrary file size. Roles can omit unused directories.

Handlers perform deferred actions such as restarting a service after a configuration change. Tasks notify them by name. Ansible normally runs each notified handler once at the relevant handler boundary, even when several tasks notify it.

Role defaults suit settings that inventories, playbooks, or command-line variables may replace. Role variables suit internal values that consumers should rarely change. Extra variables can still override them. Sensitive values belong in Ansible Vault or an approved secrets system, not in either directory.

Ansible finds roles in collections, in `roles/` beside the playbook, in the playbook directory, and in the configured `roles_path`. Its default standalone-role path is `~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles`.

The following command creates a role skeleton:

```shell
ansible-galaxy role init motd --init-path roles
```

A role task refers to content within its own resource directory without repeating that directory name. For example, a template task uses `src: motd.j2`, not `src: templates/motd.j2`. File modes should use quoted strings such as `'0444'`. Templates should use current fact notation such as `ansible_facts['hostname']`.

An MOTD role can keep its message template in `templates/motd.j2`, expose the responsible team as a default, and deploy `/etc/motd` through `ansible.builtin.template`. The playbook then supplies environment-specific contact details without changing the role.
## Applying roles
Roles listed under `roles` act as static imports. Ansible runs `pre_tasks`, their notified handlers, roles in the declared order, ordinary tasks, their notified handlers, `post_tasks`, and the remaining notified handlers. Role dependencies run before the dependent role.

```yaml
---
- name: Apply the system baseline
  hosts: all
  become: true
  roles:
    - role: organisation.motd
      vars:
        system_manager: operations@example.com
```

`ansible.builtin.import_role` also loads a role statically at a task position. `ansible.builtin.include_role` loads it dynamically, which supports run-time conditions and task ordering. A role declares dependencies in `meta/main.yml`. Dependency names and parameters must match the installed content, such as `mariadb` rather than the misspelt `mariabd`.

Dependencies suit genuine prerequisites, such as a web application requiring a database role. Loose operational sequencing belongs in the playbook because explicit orchestration remains easier to inspect and change.

Projects should keep inventories, `group_vars`, `host_vars`, playbooks, roles, and dependency files under version control. Each role should have a narrow purpose, documented inputs, sensible defaults, argument validation, and automated tests. A site playbook can coordinate several focused roles without embedding their implementation.

Separate inventories can represent development, testing, and production. Shared values belong in group variables, host exceptions belong in host variables, and `site.yml` can provide the main entry point.
## Galaxy roles and collections
Ansible Galaxy distributes standalone roles and collections. Collections now provide the main packaging model for roles, modules, plugins, and related content. Before adopting external content, administrators should inspect its publisher, source, licence, maintenance activity, supported platforms, dependencies, tests, and implementation. Download counts do not establish quality or security.

A requirements file records reviewed dependencies and pins versions so that later installations use the intended releases:

```yaml
---
roles:
  - name: namespace.role_name
    version: "1.2.3"
collections:
  - name: namespace.collection_name
    version: "4.5.6"
```

The command `ansible-galaxy install -r requirements.yml` installs both sections. Separate `role` and `collection` subcommands support searching, inspecting, listing, installing, verifying, and removing content where applicable. Teams should review updates before changing pinned versions.
## RHEL System Roles
RHEL System Roles provide supported automation for common RHEL services and settings. On a RHEL 10 control node, `ansible-core` 2.16 manages RHEL 9 and RHEL 10 nodes. The command `dnf install rhel-system-roles` installs the `redhat.rhel_system_roles` collection and `ansible-core`. The collection resides under `/usr/share/ansible/collections/ansible_collections/redhat/rhel_system_roles/`. Ansible Automation Platform can install the same collection from Red Hat Automation Hub.

Playbooks should use fully qualified names such as `redhat.rhel_system_roles.selinux` and `redhat.rhel_system_roles.timesync`.

The SELinux role manages policy state, booleans, file contexts, restored paths, port labels, login mappings, and custom modules. Its file-context variable is `selinux_fcontexts`, not `selinux_fcontext`. A pattern such as `/web(/.*)?` should omit `ftype` or set `ftype: a` when every required file type needs the label. The value `ftype: d` labels directories only. SELinux changes should grant the least access needed and should remain compatible with enforcing mode.

The TimeSync role configures `chronyd` by default on RHEL 10. It replaces the selected provider's existing configuration, so the playbook must declare every setting that must remain.

```yaml
---
- name: Configure time synchronisation
  hosts: rhel
  become: true
  tasks:
    - name: Configure the NTP source
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.timesync
      vars:
        timesync_ntp_servers:
          - hostname: time.example.com
            iburst: true
```

Administrators should validate playbook syntax, test changes on representative systems, check the resulting service state, and confirm idempotence before broad deployment.