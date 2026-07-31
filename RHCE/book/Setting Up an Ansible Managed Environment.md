# Setting Up an Ansible Managed Environment
## Project organisation
RHEL 10 includes `ansible-core` 2.16 for supported RHEL automation content. A project directory keeps related automation together and gives each workload or environment its own defaults. Teams should protect the directory with appropriate permissions and manage its non-secret content in version control.

| Component | Purpose |
| --- | --- |
| `ansible.cfg` | Project-level defaults |
| Inventory sources | Managed hosts, groups, and connection data |
| `group_vars/` and `host_vars/` | Group-specific and host-specific variables |
| Playbooks and roles | Automation logic |
| Requirements files | Collection and role dependencies |

This layout supports review, testing, reuse, and delegation. An organisation can separate production, test, regional, or application inventories while sharing roles and collections through controlled dependencies.
## Static inventory
Ansible builds an in-memory inventory from one or more sources. The built-in INI and YAML inventory plug-ins accept hostnames, fully qualified domain names, IP addresses, aliases, groups, ranges, and parent-child group relationships. A host can belong to several functional, regional, and lifecycle groups.

Ansible always creates the `all` and `ungrouped` groups. `all` contains every explicit inventory host, while `ungrouped` contains hosts outside other groups. When Ansible parses no inventory, it warns that only an implicit localhost is available. Group patterns cannot target that implicit host.

Host and group variables in inventory remain supported. Separate files under `host_vars/` and `group_vars/` usually provide clearer policy, richer YAML structures, and easier maintenance. Teams must not store unencrypted passwords, tokens, or private keys in inventory or source control.

Ansible selects inventory through the `-i` option, the `inventory` configuration setting, or the default `/etc/ansible/hosts` path. It does not automatically select a file named `inventory` from the current directory. These commands inspect an explicit source:

```shell
ansible -i inventory all --list-hosts
ansible-inventory -i inventory --graph
ansible-inventory -i inventory --list
```

Repeated `-i` options combine sources. An inventory directory can also contain static files, executable scripts, plug-in configuration files, `group_vars/`, and `host_vars/`, subject to configured ignore rules. Ansible loads sources in the supplied order, and the last definition wins when the same variable appears more than once.
## Dynamic inventory
Dynamic inventory discovers hosts from cloud platforms, virtualisation systems, directories, and other external services. Inventory plug-ins generally suit dynamic inventory better than scripts because they integrate configuration, caching, variable composition, and grouping with `ansible-core`. Collections distribute provider-specific plug-ins, which often use YAML configuration sources and require additional libraries.

Legacy inventory scripts still work. A script can use any language, not only Python, but it must run as an executable, accept `--list` and `--host <hostname>`, and emit valid JSON. Including host variables under the `_meta` key avoids a separate script call for every host. Teams should use maintained plug-ins instead of unreviewed scripts found through general web searches.
## Configuration and precedence
Ansible searches for configuration in this order:

```text
ANSIBLE_CONFIG
./ansible.cfg
~/.ansible.cfg
/etc/ansible/ansible.cfg
```

It loads the first file found and ignores the others. A home-level configuration file therefore uses the hidden name `~/.ansible.cfg`. Ansible refuses to load `./ansible.cfg` automatically from a world-writable directory because another user could inject unsafe settings.

Common settings include `inventory`, `remote_user`, connection options, privilege-escalation behaviour, and plug-in paths. `remote_user` identifies the login account, while `become_user` identifies the account used after escalation. Without `remote_user`, SSH normally uses the current control-node account.

SSH host-key checking remains enabled by default and protects against spoofing and man-in-the-middle attacks. Administrators should manage trusted entries in `known_hosts` instead of setting `host_key_checking = false`. Secrets belong in Ansible Vault or managed platform credentials, not `ansible.cfg`.

From lowest to highest, the main precedence categories are configuration settings, command-line options, playbook keywords, variables, and direct assignment. Environment variables override corresponding `ansible.cfg` entries, and extra variables have high variable precedence. Ansible does not apply a universal "most specific setting wins" rule.

The following commands generate and inspect configuration:

```shell
ansible-config init --disabled > ansible.cfg
ansible-config dump --only-changed
```