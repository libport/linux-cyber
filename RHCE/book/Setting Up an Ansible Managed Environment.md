# Setting Up an Ansible Managed Environment
## Ansible project layout
Ansible works best when each project keeps its automation in one place. A project directory can hold playbooks, inventory, variable files, included task files, and an `ansible.cfg` file. This layout makes delegation easier and keeps project-specific settings separate from system-wide defaults.

A small environment can share one configuration for everything. Larger environments usually benefit from separate project directories. Teams can also keep common settings in a user home directory or under `/etc/ansible` when every project uses the same defaults.
## Static inventory
Inventory defines the hosts Ansible manages. A simple inventory can list hostnames or IP addresses. It can also use ranges such as `server[1:6].example.com` to address multiple systems. In small deployments, `/etc/ansible/hosts` can serve as the default inventory and remove the need to pass `-i` on every command.

Groups make large inventories easier to manage. A host can belong to more than one group, and groups can contain other groups. Common grouping patterns include:
- functional groups such as `web` and `db`
- regional groups such as `apac` or `europe`
- stage-based groups such as `test`, `development`, and `prod`

Ansible also provides implicit targets. `all` includes every host in inventory, `ungrouped` includes hosts outside named groups, and `localhost` refers to the current machine.

`ansible -i inventory all --list-hosts` lists hosts that match a pattern. `ansible-inventory --list` returns structured output, and `ansible-inventory --graph` shows the group hierarchy.

Variables can live in the inventory source, but separate `host_vars/` and `group_vars/` files are usually clearer and easier to maintain.
## Dynamic inventory
Dynamic inventory suits cloud and other fast-changing environments. Current Ansible practice prefers inventory plugins over older custom scripts because plugins integrate directly with ansible-core and support external data sources more cleanly. Legacy scripts still exist, but they are no longer the preferred approach.

When multiple inventory sources are needed, Ansible can read them from a directory. Static files can be placed there directly. Legacy script-based sources also need execute permission.
## ansible.cfg
`ansible.cfg` stores persistent defaults for connecting to managed hosts. Common settings include:
- `remote_user` for the login account
- `inventory` for the default inventory path
- `host_key_checking` for SSH host key validation
- `become`, `become_method`, `become_user`, and `become_ask_pass` for privilege escalation

A typical file uses INI syntax with sections such as `[defaults]` and `[privilege_escalation]`. Project-specific configuration can live beside the playbooks, while broader defaults can live in a user configuration file or in `/etc/ansible/ansible.cfg`.

More specific settings override broader ones. Playbooks and other higher-precedence sources can override many configuration values when required.
## Core practice
A maintainable Ansible setup keeps inventory close to the project that uses it, groups hosts by operational need, stores reusable variables in `host_vars/` and `group_vars/`, and uses `ansible.cfg` to define safe defaults for connections and privilege escalation. In dynamic environments, inventory plugins provide the most reliable way to keep host data current.