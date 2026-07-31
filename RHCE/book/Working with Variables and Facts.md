# Variables, Facts, Vault, and Registered Results on RHEL 10
Red Hat Enterprise Linux 10 provides `ansible-core` 2.16 for supported management of RHEL 9 and RHEL 10 nodes. Variables separate reusable automation from host-specific data. Facts describe managed hosts, magic variables expose Ansible's internal context, Vault protects sensitive values at rest, and registered results carry task outcomes to later tasks.
## Defining and using variables
Ansible variables use YAML data and Jinja expressions. A valid name contains letters, numbers, and underscores, cannot begin with a number, and cannot use a Python or playbook keyword. An initial underscore is valid but provides no privacy.

Jinja encloses an expression in double braces. YAML requires quotes when a value begins with an expression:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars:
    web_packages:
      - httpd
      - firewalld
    service_account:
      name: webops
      shell: /bin/bash
  tasks:
    - name: Install web packages
      ansible.builtin.dnf:
        name: "{{ web_packages }}"
        state: present

    - name: Create the service account
      ansible.builtin.user:
        name: "{{ service_account['name'] }}"
        shell: "{{ service_account['shell'] }}"
        state: present
```

A list stores ordered items and uses zero-based indexes such as `web_packages[0]`. A dictionary maps keys to values. Both `service_account.name` and `service_account['name']` can access a dictionary value, but bracket notation avoids collisions with dictionary attributes.

Variables can reside in a play's `vars` block, files named under `vars_files`, role defaults and variables, inventory, `group_vars`, `host_vars`, task-level definitions, registered results, or extra variables. Inventory variables remain supported and are not deprecated. Separate `group_vars/GROUP` and `host_vars/HOST` files usually organise structured data more clearly than inline INI inventory values.

| Location | Typical use |
|---|---|
| Role defaults | Values that callers can readily override |
| Inventory and `group_vars` | Environment, location, or host-group policy |
| `host_vars` | A host-specific exception or connection value |
| Play `vars` and `vars_files` | Data scoped to one play |
| Task variables and role parameters | Data scoped to specific work |
| Registered variables and `set_fact` | Values produced during execution |
| Extra variables | An explicit run-time override |

`vars_files` loads named YAML or JSON data into a play:

```yaml
vars_files:
  - vars/common.yml
  - vars/rhel10.yml
```

The enabled `host_group_vars` plugin automatically loads matching files relative to the inventory source or playbook. These implicit locations improve reuse but require a coherent project layout so administrators can trace each value.

Precedence does not follow one universal "most specific value" rule. Configuration settings, command-line options, playbook keywords, variables, and direct plugin assignments occupy different precedence categories. Within variable sources, role defaults have low precedence, while `--extra-vars` or `-e` overrides every other variable. A project should define each value in one intentional location and consult the formal precedence order when duplication cannot be avoided.
## Gathering and using facts
Ansible runs fact gathering at the beginning of each play by default. The `ansible.builtin.setup` module discovers operating system, network, hardware, storage, date, and other host properties. An ad hoc command can inspect selected facts:

```console
ansible all -i inventory -m ansible.builtin.setup -a 'filter=ansible_distribution*'
```

Ordinary variables hold values supplied by inventory, playbooks, roles, or operators. Facts hold values discovered from a host or returned by a module. Magic variables hold Ansible's execution state. All three can participate in Jinja expressions and conditions, but Ansible controls the names and values of facts and magic variables.

Playbooks store gathered facts in the `ansible_facts` dictionary. Common RHEL checks include:

```yaml
ansible_facts['distribution']
ansible_facts['distribution_major_version']
ansible_facts['os_family']
ansible_facts['default_ipv4']['address']
```

Fact injection also creates top-level names such as `ansible_distribution` when `inject_facts_as_vars` remains enabled. That compatibility setting defaults to enabled in current Ansible, but projects can disable it. References through `ansible_facts` avoid dependence on the injected namespace.

The `ansible.builtin.debug` module can display a variable without Jinja delimiters:

```yaml
- name: Display the RHEL major version
  ansible.builtin.debug:
    var: ansible_facts['distribution_major_version']
```

A play that requires no host facts can set `gather_facts: false`. A later `ansible.builtin.setup` task can gather all facts or a filtered subset. Persistent cache plugins can reduce collection time across runs, but administrators must choose an appropriate expiry and avoid decisions based on stale disk, network, package, or security data.

Specialised modules can add information that default gathering omits. For example, `ansible.builtin.package_facts` returns installed-package data. Module documentation defines each returned structure, and supporting utilities on the managed host can affect which facts are available.
## Custom facts and magic variables
Linux local facts normally reside in `/etc/ansible/facts.d` on each managed host. Every filename ends in `.fact`. A static file uses INI or JSON and must not be executable. A dynamic `.fact` program must be executable and return valid JSON.

Ansible places local facts under `ansible_facts['ansible_local']`. For `/etc/ansible/facts.d/site.fact` containing an INI section named `software`, the package value appears as:

```yaml
ansible_facts['ansible_local']['site']['software']['package']
```

Fact gathering occurs before normal tasks. A play that copies a new `.fact` file must run `ansible.builtin.setup` again before later tasks in that play can use the new values. A subsequent play gathers them normally.

```yaml
- name: Create the local fact directory
  ansible.builtin.file:
    path: /etc/ansible/facts.d
    state: directory
    owner: root
    group: root
    mode: "0755"

- name: Install a static local fact
  ansible.builtin.copy:
    src: site.fact
    dest: /etc/ansible/facts.d/site.fact
    owner: root
    group: root
    mode: "0644"

- name: Refresh local facts
  ansible.builtin.setup:
    filter:
      - ansible_local
```

These file tasks require privilege escalation for the system directory.

Magic variables describe execution and inventory context. `inventory_hostname` identifies the current inventory host, `group_names` lists its groups, `groups` maps group names to members, and `hostvars` maps hosts to their variables. Facts for another host appear through `hostvars` only after Ansible gathers or caches them. Ansible reserves magic-variable names, so project variables must not reuse them.
## Protecting sensitive values with Vault
Ansible Vault encrypts complete files or individual YAML values. It protects stored content but decrypts values during execution, so task output, logs, templates, and external systems can still disclose a secret. Sensitive tasks should use `no_log: true` where appropriate, and operators should secure editors, temporary files, output, backups, and password sources.

Vault supports `create`, `encrypt`, `encrypt_string`, `edit`, `view`, `decrypt`, and `rekey`. A labelled Vault ID clarifies which password source belongs to encrypted content:

```console
ansible-vault create --vault-id prod@prompt group_vars/webservers/vault.yml
ansible-playbook -i inventory site.yml --vault-id prod@prompt
```

| Operation | Effect |
|---|---|
| `create` | Creates and edits a new encrypted file |
| `encrypt` | Encrypts an existing file |
| `encrypt_string` | Produces an encrypted YAML value |
| `view` or `edit` | Displays or updates content while retaining encrypted storage |
| `rekey` | Changes a file's password or Vault ID |
| `decrypt` | Writes a file back to plaintext |

Multiple passwords require repeated options:

```shell
ansible-playbook -i inventory site.yml --vault-id dev@prompt --vault-id prod@prompt
```

Password files and client scripts remove interactive prompts but become high-value secrets. They require restrictive permissions, controlled storage, and exclusion from version control. A managed secret service or an approved credential facility often provides stronger operational control than a plaintext password file.

Public variable names and encrypted values can remain separate. For example, an unencrypted variables file can map `db_password` to `vault_db_password`, while a Vault-encrypted file defines `vault_db_password`. This structure keeps the expected interface visible without exposing the value.

Vault encryption does not create an operating system password hash. On RHEL 10, `ansible.builtin.user` requires its `password` argument to contain an encrypted Linux password hash. Automation should generate an approved hash securely, protect that hash with Vault, and avoid passing a plaintext login password to the module.
## Registering task results
The `register` keyword stores a task result as a per-host variable for the current playbook run. Every registered result contains task status, while each module documents its additional return keys. Command modules commonly return `rc`, `stdout`, `stdout_lines`, `stderr`, and `stderr_lines`.

```yaml
- name: Query the httpd package
  ansible.builtin.command:
    argv:
      - rpm
      - -q
      - httpd
  register: httpd_query
  changed_when: false
  failed_when: httpd_query.rc not in [0, 1]

- name: Report an installed package
  ansible.builtin.debug:
    msg: "{{ httpd_query.stdout }}"
  when: httpd_query.rc == 0
```

The read-only command sets `changed_when: false` because the command module otherwise reports execution as a change. The package query uses return codes 0 and 1 as expected outcomes and treats other codes as failures. A purpose-built fact or information module remains preferable when one supplies the required state directly.

Registered variables exist in memory, apply separately to each host, and do not persist into a later playbook run. A failed or skipped task can still create a registered result with the corresponding status. Looped tasks place per-item results in a `results` list.