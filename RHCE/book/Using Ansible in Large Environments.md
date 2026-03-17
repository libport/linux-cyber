# Using Ansible in Large Environments
## Advanced inventory
Large Ansible estates rely on precise inventory targeting. Inventory can contain hostnames, IP addresses, and groups. If automation targets an IP address, that IP address needs to exist in inventory rather than relying on DNS alone. Ansible also provides implicit groups named `all` and `ungrouped`.

Quoted host patterns let Ansible expand wildcards instead of the shell. Patterns can match names, addresses, and group names, so operators need to choose them carefully. A pattern such as `web*` may match hosts like `web1` as well as a group such as `webservers`.

```bash
ansible -m ping 'ansible*'
ansible -m ping 'web,&file'
ansible -m ping 'web,!webserver1'
ansible -m ping ansible1,192.168.4.202
```

Commas are clearer than colons when combining targets because IPv6 addresses already contain colons. Host execution order is not guaranteed, so playbooks should not depend on one host running before another unless batching is configured explicitly.
## Dynamic and combined inventory
Static inventory suits stable environments. Fast-changing environments benefit from dynamic inventory, which now usually means inventory plugins rather than older executable scripts. Legacy scripts still work if they return JSON for `--list` and `--host` and have execute permission.

A minimal inventory source can discover hosts through `getent hosts` and expose them under `all`. The key implementation detail is `universal_newlines=True`, not `universal_newline=True`.

```python
#!/usr/bin/python
from subprocess import Popen, PIPE
import json

pipe = Popen(['getent', 'hosts'], stdout=PIPE, universal_newlines=True)
hosts = []
for line in pipe.stdout:
    hosts.extend(line.split())

print(json.dumps({"all": {"hosts": hosts, "vars": {}}}))
```

Dynamic inventory becomes most useful when the source of truth already exists elsewhere, such as FreeIPA, Active Directory, Satellite, VMware, or public cloud platforms. These sources often need separate configuration, credentials, and cache controls. `ansible-inventory` makes inventory easier to inspect and troubleshoot in machine-readable or graph form.

```bash
ansible-inventory -i inventory_dir --graph
ansible-inventory -i inventory -i dynamic_source --list
```

An inventory directory can combine static files and executable dynamic sources in one place. Multiple `-i` options can merge several inventory sources for a single run, which makes it practical to overlay local hosts, lab systems, and cloud-discovered nodes.
## Parallel and serial execution
Ansible uses the linear strategy by default. It runs each task across the targeted hosts before moving to the next task and uses 5 forks unless configured otherwise. Raising `forks` can speed up work when managed nodes do most of the processing, but the control node still sets the practical limit. A higher value is useful only when CPU, memory, and network capacity can support it.

Serial execution protects clustered or customer-facing services during change. Instead of updating every node at once, `serial` completes the full play on one batch before moving on.

```yaml
- hosts: web
  serial: 2
  tasks:
    - name: Install Apache
      ansible.builtin.package:
        name: httpd
        state: present
```

This pattern reduces the risk of taking an entire service offline during package upgrades or restarts. It also gives operators a clear point to validate the first batch before continuing.
## Reusing playbooks and tasks
Large playbooks stay maintainable when common logic moves into separate files. `import_playbook` works at the top level only and loads another playbook statically during parsing. `import_tasks` also loads tasks statically, which makes them visible to `--list-tasks` and suitable for predictable task graphs. `include_tasks` loads tasks dynamically at runtime, which is usually the better choice when conditions, loops, or filenames depend on values discovered during execution. Dynamically included tasks do not appear in `--list-tasks`, so static imports remain easier to preview.

Variables keep shared task files generic. A reusable task file should refer to package names, services, and firewall rules through variables rather than hard-coded values.

```yaml
- hosts: ansible2
  tasks:
    - name: Configure service tasks dynamically
      include_tasks: tasks/service.yml
      vars:
        package: httpd
        service: httpd

    - name: Configure firewall tasks statically
      import_tasks: tasks/firewall.yml
      vars:
        firewall_package: firewalld
        firewall_service: firewalld
```

This structure separates site-specific values from reusable automation and keeps large projects easier to test, delegate, and extend. Roles remain the better option when a whole feature set needs its own standard directory structure.