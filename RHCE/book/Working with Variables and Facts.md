# Working with Variables and Facts
## Ansible variables and facts
Ansible uses variables to separate changing data from task logic. Facts, user-defined variables, and magic variables all provide values at runtime, but they serve different purposes.

| Type | Purpose |
| --- | --- |
| Facts | Describe managed hosts |
| User-defined variables | Hold values chosen by the operator |
| Magic variables | Expose Ansible's internal state |

A simple play can define a variable in `vars` and then interpolate it in a task:

```yaml
---
- name: Create a user
  hosts: ansible1
  vars:
    username: lisa
  tasks:
    - name: Create the account
      ansible.builtin.user:
        name: "{{ username }}"
```

Quoting a value remains safest when the value starts with a variable expression. Variable expansion also works in task names, which helps produce readable play output.
## Working with facts
Ansible gathers facts at the start of each play unless fact gathering is disabled. It stores the result in `ansible_facts`, a dictionary of host data such as the hostname, distribution, network interfaces, addresses, devices, and date information.

Square-bracket notation is the preferred way to access nested fact data:

```yaml
{{ ansible_facts['hostname'] }}
{{ ansible_facts['distribution'] }}
{{ ansible_facts['default_ipv4']['address'] }}
{{ ansible_facts['devices']['sda']['partitions']['sda1']['size'] }}
```

Dot notation also works in many cases, but square brackets avoid ambiguity with keys that collide with Python methods or contain punctuation.

Older playbooks often use injected fact variables such as `ansible_hostname` or `ansible_default_ipv4.address`. Ansible still exposes these in some contexts, but `ansible_facts[...]` is the clearer and more future-safe form.

The `debug` module can print the entire fact set or a specific value:

```yaml
- name: Show all gathered facts
  ansible.builtin.debug:
    var: ansible_facts
```

Commonly used facts include the short hostname, Linux distribution, distribution version, primary IPv4 address, interfaces, and attached storage devices.
## Managing fact gathering
Fact gathering improves portability because tasks and conditionals can branch on the actual state of each host. It also adds overhead. A play can skip the initial collection step with `gather_facts: false` and then call the `setup` module later when facts become necessary.

Fact caching requires a configured cache plugin such as `jsonfile` or `redis`. Caching can improve performance, but cached values can become stale. Tasks that depend on current disk, package, or service state should prefer fresh facts rather than cached data.
## Custom facts
Custom facts let a managed host publish site-specific values. Static custom facts live in `/etc/ansible/facts.d` on the managed host, use the `.fact` extension, and use INI or JSON format. Executable `.fact` files can return JSON dynamically.

Example static fact file:

```ini
[packages]
web_package = httpd
ftp_package = vsftpd

[services]
web_service = httpd
ftp_service = vsftpd
```

After Ansible gathers facts, these values appear under `ansible_facts['ansible_local']`. A play can use them directly:

```yaml
{{ ansible_facts['ansible_local']['custom']['software']['package'] }}
```

When a play installs a new custom fact file and needs that data immediately, it should run `ansible.builtin.setup` again in the same play. Otherwise, the new fact becomes available in the next play that gathers facts.
## Defining and organising variables
A playbook can define variables in a `vars` block, load them from `vars_files`, or place them in `host_vars` and `group_vars`. Centralising shared values in files usually improves consistency and reduces duplication.

Variable names should start with a letter and should contain only letters, numbers, and underscores. They are case-sensitive.

Example variable file usage:

```yaml
---
- name: Install a package from a variable file
  hosts: ansible1
  vars_files:
    - vars/common.yml
  tasks:
    - name: Install the package
      ansible.builtin.yum:
        name: "{{ my_package }}"
        state: latest
```

Host-specific files in `host_vars/hostname` and group-specific files in `group_vars/groupname` load automatically when the inventory matches. This structure keeps inventory data separate from play logic and works well for larger projects.
## Multivalued variables
Ansible commonly uses lists and dictionaries.

A list stores multiple items in order:

```yaml
users:
  - linda
  - lisa
  - anna
```

A dictionary stores named key-value pairs:

```yaml
users:
  linda:
    homedir: /home/linda
    shell: /bin/bash
```

A task can address dictionary members with square brackets:

```yaml
{{ users['linda']['homedir'] }}
```

These structures become especially useful in loops and conditional logic, where one task can act on many users, packages, or services.
## Magic variables and precedence
Magic variables reflect Ansible's runtime state. Common examples include `hostvars`, `groups`, `group_names`, `inventory_hostname`, and `inventory_file`. Because Ansible reserves these names, a playbook should not reuse them for unrelated data.

`hostvars` deserves special attention because it exposes the variables associated with every host in inventory. That makes cross-host lookups possible when a play needs information gathered from another machine.

When the same variable name appears in multiple places, the most specific definition wins. Extra variables passed on the command line have the highest practical precedence. Play-level variables override inventory variables. Inventory host variables override broader group-level values. The safest design keeps names unique enough that precedence rarely becomes a problem.
## Protecting sensitive data with Vault
Sensitive values such as passwords, tokens, and keys should live in separate variable files encrypted with Ansible Vault.

Common Vault commands include:
- `ansible-vault create`
- `ansible-vault encrypt`
- `ansible-vault decrypt`
- `ansible-vault rekey`
- `ansible-vault view`
- `ansible-vault edit`

A playbook can load an encrypted file through `vars_files`. Runtime decryption can prompt for a password with `--ask-vault-pass` or `--vault-id @prompt`. A password file can automate access, but that file needs strict file permissions and careful handling.

A common layout stores plain variables in `group_vars/<group>/vars` and encrypted values in `group_vars/<group>/vault`, or the equivalent host-specific directories. This separation keeps ordinary configuration readable while isolating secrets.
## Capturing command output with register
A task can save its result with `register`. The registered variable stores structured data rather than only raw text.

| Key | Meaning |
| --- | --- |
| `cmd` | Command executed |
| `rc` | Return code |
| `stdout` | Standard output as one string |
| `stdout_lines` | Standard output split into lines |
| `stderr` | Standard error as one string |
| `stderr_lines` | Standard error split into lines |

Example:

```yaml
- name: Read passwd
  ansible.builtin.shell: cat /etc/passwd
  register: passwd_contents

- name: Show command output
  ansible.builtin.debug:
    var: passwd_contents['stdout_lines']
```

This pattern supports later tasks, especially when a conditional needs to test a return code, inspect command output, or branch on whether a previous task changed the host.
## Key points
Ansible relies on variables to keep playbooks flexible and reusable. Facts describe hosts, custom facts add local site data, and magic variables expose Ansible's own context. Structured variables, sensible variable placement, clear precedence rules, Vault-encrypted secrets, and registered task results all help produce maintainable automation.