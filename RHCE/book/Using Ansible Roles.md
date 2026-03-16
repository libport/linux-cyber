# Using Ansible Roles
Ansible roles package reusable automation components into a standard structure. They reduce duplication, keep common tasks consistent across environments, and make playbooks easier to maintain. Roles may come from local development, Ansible Galaxy, or Red Hat's supported RHEL System Roles.
## Role structure and storage
A role stores related content in predictable directories. `tasks` holds the task entry point, `handlers` stores handlers, `templates` contains Jinja2 templates, `files` stores static files, `defaults` holds low-precedence variables, `vars` holds higher-precedence variables that usually stay fixed, and `meta` records metadata and dependencies. Optional `tests` content supports validation.

Ansible searches for roles in a defined order. Project roles in `./roles` take precedence, followed by `~/.ansible/roles`, then `/etc/ansible/roles`. Packaged roles in `/usr/share/ansible/roles` usually come from RPM installation and suit vendor-provided content better than custom development.

`ansible-galaxy init` creates the base directory tree for a new role. That scaffold speeds development and encourages a consistent layout.
## Using roles in playbooks
A playbook includes roles in a `roles` list. Ansible runs included roles before the playbook's normal `tasks` section. When the playbook must prepare a system first, `pre_tasks` runs before roles. `post_tasks` runs after roles, regular tasks, and any triggered handlers.

Custom roles stay most effective when they stay narrow in scope. A simple message-of-the-day role, for example, can manage `/etc/motd` through a template and a default contact variable, while the playbook overrides that variable for a specific host or environment.
## Dependencies and file organisation
Roles may depend on other roles. Dependencies belong in `meta/main.yml`. Ansible runs dependent roles before the role that requires them, and it runs a shared dependency only once even when multiple roles request it. Dependencies may also receive variables and conditional `when` clauses. A web application role may depend on Apache and MariaDB roles rather than duplicate that setup.

Large Ansible estates benefit from project directories with a local `ansible.cfg`, inventory, playbooks, `group_vars`, `host_vars`, and a top-level `site.yml`. Roles should live in version control, avoid embedded secrets, and stay generic enough for reuse. Sensitive data belongs in Ansible Vault, not inside the role itself.
## Ansible Galaxy
Ansible Galaxy publishes community content, including roles and collections. Collections bundle several content types such as roles, modules, plug-ins, and playbooks. When a playbook only needs reusable task sets, the role often remains the most direct unit to install and include.

Galaxy helps administrators assess community content through tags, download counts, and quality indicators. The `ansible-galaxy` command supports the same workflow from the shell. `ansible-galaxy search` filters by keywords, platform, author, and Galaxy tags. `ansible-galaxy info` shows details for a selected role. `ansible-galaxy install` installs a role, and `ansible-galaxy list` and `ansible-galaxy remove` manage installed roles.

A requirements file simplifies repeatable installation. It lists each role, its source, and an optional version. Roles may come from Galaxy, a Git repository, or another downloadable archive. When a role comes from Git, the source and SCM settings must match that repository.
## RHEL System Roles
RHEL System Roles provide supported automation for common operating system configuration tasks such as networking, storage, SELinux, time synchronisation, Postfix, and kdump. They give administrators a consistent interface across supported RHEL releases and install into the standard Ansible role path with the `rhel-system-roles` package. Their documentation and example playbooks also install locally.

These roles rely on variables rather than long custom task lists. The SELinux role uses variables for policy, mode, booleans, file contexts, restore paths, ports, and login mappings. When a change requires a reboot, the playbook can wrap the role in a block, reboot the managed host, wait for reconnection, and apply the role again.

The TimeSync role configures NTP sources through `timesync_ntp_servers` and can work alongside a separate timezone task. The role handles the underlying implementation for the relevant RHEL version, which reduces version-specific branching in playbooks.
## Practical outcome
Roles turn repeated automation into maintainable building blocks. Custom roles standardise local practice, Galaxy roles accelerate common deployments, and RHEL System Roles provide supported operating system management. Together they help Ansible environments stay consistent, reusable, and easier to scale.