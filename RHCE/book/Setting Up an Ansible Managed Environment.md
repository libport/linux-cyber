# Setting Up an Ansible Managed Environment
Ansible scales best when each project keeps its own playbooks, variables, inventory, and configuration in a self-contained directory. Small environments can share one system-wide configuration, but larger environments usually benefit from separate project directories because teams can hand over, test, and version work more cleanly.

Common locations for configuration are:
- a `project` directory for project-specific settings
- a user `home` directory for user-specific settings
- `/etc/ansible` for shared system-wide settings
## Static inventory
Inventory defines the hosts Ansible manages, groups related hosts, and can also store variables. Modern practice usually keeps variables in YAML inventory, `host_vars`, or `group_vars` because that is clearer than packing complex values into INI files. Inventory can use hostnames, IP addresses, or aliases with connection variables such as ansible_host.

A simple INI inventory can list hostnames, IP addresses, ranges, groups, and child groups:

```ini
ansible1
ansible2

[web]
web1
web2

[db]
db1
db2

[servers:children]
web
db
```

Useful group patterns include:
- functional groups such as `web` or `db`
- regional groups such as australia or europe
- stage groups such as `dev`, `test`, or `prod`

Ansible always provides the all and ungrouped groups. It also supports an implicit localhost when needed.
## Dynamic inventory
Dynamic inventory suits environments where hosts change often, such as cloud platforms. Inventory plugins are the preferred approach. Legacy scripts still work through the built-in script inventory plugin, but plugins integrate better with current `ansible-core` behaviour.

A minimal inventory script must be executable, return JSON, and respond to `--list`. It should also handle `--host`, or provide `_meta` to avoid per-host lookups.

```python
#!/usr/bin/env python3
import json, sys

data = {"all": {"hosts": ["web1", "db1"]}, "_meta": {"hostvars": {}}}

if sys.argv[1:] == ["--list"]:
    print(json.dumps(data))
elif len(sys.argv) == 3 and sys.argv[1] == "--host":
    print(json.dumps({}))
else:
    raise SystemExit("use --list or --host <name>")
```

Ansible can also load multiple inventory sources from a directory, which helps combine static files and dynamic sources in one environment.
## ansible.cfg essentials
The `ansible.cfg` file sets default connection and privilege-escalation behaviour. A project can keep a local `ansible.cfg`, or rely on `~/.ansible.cfg` or `/etc/ansible/ansible.cfg`.

```ini
[defaults]
remote_user = ansible
inventory = inventory
host_key_checking = false

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

Key settings include `remote_user`, `inventory`, `host_key_checking`, and the `become` options. Precedence matters. Configuration settings sit below command-line options, playbook keywords, and variables, so the most specific setting wins.
## Core commands
```bash
ansible -i inventory all --list-hosts
ansible-inventory -i inventory --graph
ansible-inventory -i inventory --list
```

These commands verify which hosts Ansible sees and how groups resolve. Setting inventory in `ansible.cfg` removes the need to pass `-i` for every command.