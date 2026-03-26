# Linux Administration with Ansible: Advanced Ansible Automation
Advanced Ansible automation relies on a small set of disciplined practices. Jinja templates render configuration from data instead of copying static files. Roles split large playbooks into small, reusable units. Collections package modules, plugins, and roles in a form that can move independently of `ansible-core`. Vault protects secrets without removing them from the repository that uses them. Execution controls such as `forks`, `serial`, `strategy`, and `throttle` keep large runs predictable. Network resource modules apply the same design principle to routers and switches by treating configuration as structured data instead of long command strings.

The result is not just faster automation. It is safer change, lower drift, clearer review, and easier reuse across environments.
## Templating with Jinja
Jinja templating lets Ansible generate host-specific text files from one source file. The template can emit an Apache virtual host, an SSH daemon configuration, a database config, or any other text document. Jinja only evaluates variables, filters, and control structures. It passes ordinary text through unchanged.

Three delimiters define most template behaviour.

- `{{ ... }}` prints a value
- `{% ... %}` runs logic such as `if` and `for`
- `{# ... #}` adds a template comment that does not appear in the rendered file

The `ansible.builtin.template` module renders templates on the controller, then copies the rendered file to the managed host. Managed hosts do not need Jinja installed for this workflow. That detail matters because it keeps the remote host simple and leaves the rendering logic on the control node.

```yaml
- name: Render a Chrony configuration
  ansible.builtin.template:
    src: chrony.conf.j2
    dest: /etc/chrony.conf
    owner: root
    group: root
    mode: "0644"
    backup: true
    validate: "chronyd -p -f %s"
  notify: restart chronyd
```

A strong template keeps static content readable and moves only the changing parts into variables or logic. Inventory variables, group variables, host variables, and gathered facts all support that pattern. Host-scoped data gives the highest level of customisation because each host can render a different result from the same template.

```jinja
# chrony.conf.j2
{{ section_header }}
role={{ host_role }}
node_ip={{ ansible_facts.eth1.ipv4.address }}

{% for server in ntp_servers %}
server {{ server }} iburst
{% endfor %}
```

Filters reshape data inside template expressions. They do not change the original variable. They only transform output at render time. That makes them useful for formatting strings, converting data structures, choosing defaults, or preparing data for loops.

```jinja
{{ customer_name | capitalize }}
{{ employee1 | dict2items }}
{{ character_set | default('utf8') }}
```

Conditionals decide whether a block appears in the rendered file. Loops repeat a block for every item in a sequence. Together they allow one template to handle different platforms, roles, versions, or environments without duplication.

```jinja
{% if ansible_facts.distribution_version is version('8', '>=') %}
supported_release=true
{% elif ansible_facts.distribution_version is version('7', '>=') %}
legacy_release=true
{% else %}
unsupported_release=true
{% endif %}
```

```jinja
{% for node in groups['test'] %}
server {{ hostvars[node].ansible_facts.eth1.ipv4.address }}
{% endfor %}
```

Ansible exposes several magic variables that make this practical. `groups` returns inventory groups, which allows a template to iterate over members of a cluster. `hostvars` exposes variables and facts for every host in inventory, which supports templates for load balancers, clustered applications, and service discovery files. Facts usually provide richer host detail than inventory alone because they include interfaces, addresses, operating system data, and other runtime properties.

Template quality depends on restraint. A template should not turn into a second programming language hidden inside a config file. A good design keeps complex decision making in variables, group structure, or task logic, then lets the template focus on formatting the final text. That approach preserves readability and makes troubleshooting easier when a rendered file differs from expectation.

Good templates follow a few rules.

- Keep business logic light. Put complex decisions in variables or task files where possible.
- Use facts and inventory variables for host-specific output.
- Use `default()` to prevent unnecessary failures when a value can safely fall back.
- Use `validate` for critical configuration files such as `sudoers` or service configs.
- Use comments in the template for maintainers, not for the rendered file.
- Prefer clear variable names over clever filters.

The Chrony example shows how templating scales beyond simple substitution. A team can define one list of NTP servers for front-end hosts and another for back-end hosts, then render each host's file from the same template. That approach removes manual editing, reduces drift, and keeps the final configuration aligned with inventory data. The same pattern works for firewall rules, web server back ends, storage definitions, or application settings that vary by environment or role.
## Roles and reusable structure
A playbook can deploy applications, configure services, and orchestrate infrastructure, but a long single-file playbook becomes hard to read and harder to reuse. Roles solve that problem by grouping tasks, handlers, defaults, vars, templates, files, and metadata into a standard structure.

A role usually owns one clear responsibility such as NTP, Apache, MySQL, or SSH. That scope matters. A narrowly defined role remains easier to test, document, override, and share. A role that tries to build an entire platform usually becomes rigid and fragile.

```text
roles/
  ntp/
    tasks/main.yml
    handlers/main.yml
    defaults/main.yml
    vars/main.yml
    templates/
    files/
    meta/main.yml
    tests/
```

The `tasks/main.yml` file acts like the role's task entry point. `handlers/main.yml` receives notifications from tasks. `defaults/main.yml` stores low-precedence variables that callers can override easily. `vars/main.yml` stores high-precedence role variables and should be used sparingly. `templates/` and `files/` hold content that tasks deploy. `meta/main.yml` stores metadata, dependencies, and optional collection search settings. `tests/` can hold simple inventory and test playbooks for role validation.

```yaml
# roles/ntp/tasks/main.yml
- name: Install Chrony
  ansible.builtin.package:
    name: chrony
    state: present

- name: Deploy Chrony configuration
  ansible.builtin.template:
    src: chrony.conf.j2
    dest: /etc/chrony.conf
    owner: root
    group: root
    mode: "0644"
  notify: restart chronyd

- name: Set system time zone
  community.general.timezone:
    name: Europe/Istanbul
```

```yaml
# roles/ntp/handlers/main.yml
- name: restart chronyd
  ansible.builtin.service:
    name: chronyd
    state: restarted
```

Role defaults make a role flexible. They let the consuming playbook or inventory override values without editing the role itself.

```yaml
# roles/ntp/defaults/main.yml
ntp_pool_frontend:
- 0.pool.ntp.org
- 1.pool.ntp.org

ntp_pool_backend:
- 2.pool.ntp.org
- 3.pool.ntp.org
```

A playbook then stays small because it only assigns hosts, privilege escalation, and the role list.

```yaml
- name: Deploy NTP
  hosts: test
  become: true
  roles:
  - ntp
```

Role defaults and role vars differ in precedence. Defaults sit near the bottom of Ansible's precedence ladder, so callers can replace them through inventory, play vars, extra vars, or other higher-precedence sources. Vars override most other sources and can make a role difficult to customise. That is why defaults fit user-tunable settings such as package names, ports, or service lists, while vars should remain rare and deliberate.

Role dependencies declared in `meta/main.yml` load automatically before the dependent role. That can simplify shared prerequisites, but it also widens variable scope and can obscure execution order if overused. Clear namespacing prevents collisions. Variables such as `mysql_db_name` and `apache_server_name` communicate intent better than generic names such as `db_name` or `server_name`.

Role migration usually follows a simple path. A team starts with a working playbook, identifies cohesive groups of tasks, then moves each group with its handlers, templates, files, and variables into a role. The remaining playbook becomes an orchestration layer rather than a monolith. That pattern improves maintainability without changing the final behaviour. It also encourages selective testing because each role can be exercised on its own.

Three mechanisms reuse roles.

- `roles:` applies roles at play level before normal tasks
- `ansible.builtin.import_role` reuses a role statically and places its tasks at a specific point in task order
- `ansible.builtin.include_role` reuses a role dynamically at runtime

`import_role` and `include_role` differ in how they handle conditionals and tags. `import_role` applies most keywords to imported tasks. `include_role` applies them to the include itself unless the role also carries matching task-level tags or the playbook passes keywords through `apply`. That distinction matters when a run should execute only part of a role.

```yaml
- name: Run only tagged install tasks from a role
  ansible.builtin.include_role:
    name: ntp
  tags:
  - install
```

```yaml
# roles/ntp/tasks/main.yml
- name: Install Chrony
  ansible.builtin.package:
    name: chrony
    state: present
  tags:
  - install
```

Tagging works best when roles reserve a few high-value tags for install, config, service, verify, or cleanup phases. Excessive tagging usually hides poor role design. A clearer role with smaller task files often beats a large role covered in dozens of tags.
## Galaxy, collections, and fully qualified names
Ansible Galaxy distributes reusable community content. Historically it focused on roles. Modern Ansible also distributes collections, which package roles, modules, plugins, playbooks, and documentation together. Collections decouple content from `ansible-core`, so new modules and plugins can ship independently of the core release cycle.

A collection lives under a namespace and a collection name. That structure produces the fully qualified collection name, or FQCN. For example, the timezone module from the community general collection uses `community.general.timezone`.

FQCNs remove ambiguity and future-proof playbooks. They avoid collisions when different collections expose modules with the same short name. They also make documentation and code review easier because the source of a plugin is explicit.

```yaml
- name: Set time zone explicitly
  community.general.timezone:
    name: Australia/Sydney
```

A playbook can define a `collections` search list for short names, but roles do not inherit that list from the playbook. A role must declare its own collection search order in `meta/main.yml`, or it should use FQCNs directly.

```yaml
# roles/ntp/meta/main.yml
collections:
- ansible.builtin
- community.general
```

Galaxy content still needs review before use. Useful signs include clear documentation, recent maintenance, passing tests, sane variable names, sensible defaults, and readable code. Download counts and community scores help, but they do not replace code review. A widely used role may still miss a required feature, and an excellent role may still need local customisation. Maintenance recency matters because Ansible deprecations and platform changes can break older content.

A quick role review should inspect more than the README. The task files should show idempotent modules instead of shell commands where practical. Defaults should expose the variables that operators are likely to override. Handlers should restart services only when configuration actually changes. Templates should stay readable and should not bury critical logic in deeply nested Jinja. A role that passes those checks usually offers a safer starting point than one that only looks popular on Galaxy.

Collections have their own installation and search paths. Ansible uses `collections_path` in `ansible.cfg` and `ANSIBLE_COLLECTIONS_PATH` in the environment. Roles use `roles_path` and `ANSIBLE_ROLES_PATH`. Search order decides which installed copy wins when multiple versions exist on the controller, so explicit per-project configuration reduces surprises.

A requirements file keeps those dependencies reproducible.

```yaml
# requirements.yml
roles:
- name: geerlingguy.ntp

collections:
- name: community.general
- name: cisco.ios
```

The current CLI separates role and collection installation more clearly than older material often suggests.

```bash
ansible-galaxy role install geerlingguy.ntp
ansible-galaxy collection install community.general
ansible-galaxy collection install -r requirements.yml
```

A combined `ansible-galaxy install -r requirements.yml` can install mixed requirements, but the explicit subcommands communicate intent more clearly and avoid confusion during maintenance.
## Protecting secrets with Vault
Ansible Vault encrypts sensitive content at rest while keeping it inside the same repository as the automation that uses it. That suits passwords, API tokens, certificates, and other secrets that should not appear in plain text.

Vault supports two main approaches.

- Encrypt a whole file such as `group_vars`, `host_vars`, role defaults, or standalone variable files
- Encrypt individual strings and embed them in otherwise readable YAML

Whole-file encryption works well when most of the file is sensitive. That keeps the secret boundary simple and avoids scattering encrypted fragments through many files.

```bash
ansible-vault create group_vars/prod/database.yml
ansible-vault edit group_vars/prod/database.yml
ansible-vault view group_vars/prod/database.yml
ansible-vault encrypt roles/mysql/defaults/main.yml
ansible-vault rekey group_vars/prod/database.yml
```

Playbooks load vaulted files in the same way as plain variable files. Ansible decrypts them in memory during execution.

```yaml
- name: Read vaulted variables
  hosts: db
  vars_files:
  - vaultfile1.yml
  tasks:
  - name: Show database admin user
    ansible.builtin.debug:
      var: db_admin_user
```

Ansible can request a vault password interactively with `--ask-vault-pass`, read it from a password file with `--vault-password-file`, or use configured vault identities. Password files simplify non-interactive runs, but they need strict file permissions and careful exclusion from version control.

Vault IDs label different secrets such as `dev` and `prod`. The label does not enforce a specific password, but it helps Ansible select the right secret source for decryption and encryption.

```ini
[defaults]
vault_identity_list = dev@.devpass, prod@.prodpass
```

```bash
ansible-vault create --encrypt-vault-id dev vars/dev.yml
ansible-vault create --encrypt-vault-id prod vars/prod.yml
```

Encrypting individual variables keeps the rest of the YAML readable.

```bash
ansible-vault encrypt_string --encrypt-vault-id dev 'foobar' --name 'app_token'
```

```yaml
app_token: !vault |
  vault-encrypted-value-for-dev
```

Whole-file encryption and `encrypt_string` solve different problems. Whole-file encryption hides everything and supports rekeying cleanly. Encrypted strings preserve context around one value, which can help when only a few fields are sensitive. The trade-off is maintenance overhead because many small encrypted values can become hard to rotate and review. A team should pick one pattern per file where possible.

Vault should protect data, not excuse weak operational practice. Strong passwords, regular rotation, restrictive permissions on password files, and controlled access to secret sources still matter. Where possible, external secret managers offer a cleaner long-term design, but Vault remains a practical built-in option for many teams.
## Controlling execution and parallelism
Ansible does not execute every task on every host in one giant burst. It applies a strategy, a fork limit, and optional task-level controls to decide how work flows. Those settings matter once an estate grows past a handful of hosts.

`forks` controls how many worker processes the controller spawns. The default is conservative. A stronger controller can often handle more, but the right value depends on CPU, memory, storage, network conditions, and the cost of individual tasks. Raising `forks` blindly can swamp the controller or the target systems, so tuning should follow measurement rather than guesswork.

`serial` limits how many hosts Ansible processes in each batch for a play. That setting suits rolling updates because it lets only part of the estate change at once. It also provides a natural pause point for validation between batches.

```yaml
- name: Roll out Chrony in batches
  hosts: production
  serial: 20
  tasks:
  - name: Apply NTP role
    ansible.builtin.import_role:
      name: ntp
```

Ansible offers four built-in strategy plugins.

- `linear` runs each task across the current host batch before moving to the next task
- `free` lets each host progress through tasks independently
- `host_pinned` keeps one worker attached to one host until that host finishes the play
- `debug` opens an interactive debugging session when a task fails

`linear` remains the default because it is predictable. `free` usually improves throughput where hosts do not depend on one another. `host_pinned` helps when each host should progress independently but the controller still needs a hard worker cap. `debug` is a development aid rather than a throughput tool, but it can save time while writing or fixing a playbook because it allows inspection at failure time.

```yaml
- name: Deploy NTP quickly to independent hosts
  hosts: production
  strategy: free
  roles:
  - ntp
```

`throttle` limits concurrency for one task or block without forcing the whole playbook to run slowly. That suits API calls, expensive database operations, or other tasks that fail under heavy parallel load.

```yaml
- name: Update a rate-limited service
  hosts: app
  tasks:
  - name: Register nodes with external API
    ansible.builtin.uri:
      url: https://api.example.invalid/register
      method: POST
    throttle: 2
```

These controls combine. The smallest effective limit wins. For example, if a play uses `forks = 50`, `serial: 10`, and `throttle: 2` on one task, that task runs with at most two concurrent workers inside batches of ten hosts. That interaction explains why a play can still appear slow even after `forks` has been raised.

A practical NTP rollout illustrates the point. A production fleet of 200 Linux servers does not need a serial window if hosts sync against public or redundant time sources independently. In that case, `strategy: free` and a higher fork count can reduce run time sharply. A clustered database upgrade would require the opposite approach because the hosts depend on staged changes and health checks between batches.
## Network automation with resource modules
Network automation often starts with raw device commands or Jinja-generated command blocks. Those methods work, but they push too much vendor-specific syntax into playbooks and templates. Resource modules improve that model by converting structured data into device-native configuration and by converting running configuration back into structured data.

Each resource module manages one network function such as ACLs, interfaces, or VLANs. The module name usually combines the platform and the resource, such as `cisco.ios.ios_acls`. Network facts modules and resource modules align around the same data model, which makes it easier to gather, store, modify, and reapply configuration.

```yaml
- name: Gather ACL, interface, and layer 3 interface facts
  hosts: ios1
  gather_facts: false
  tasks:
  - name: Collect network resources
    cisco.ios.ios_facts:
      gather_subset:
      - min
      gather_network_resources:
      - interfaces
      - l3_interfaces
      - acls
```

Resource modules use structured input and a `state` value. Action states change device configuration. Non-action states transform or retrieve data without changing the device.

- `merged` adds the provided configuration into the existing resource configuration
- `replaced` replaces a subsection
- `overridden` replaces the resource configuration under management
- `deleted` removes the managed configuration
- `gathered` returns structured data from the device
- `rendered` converts structured data to vendor CLI without contacting the device
- `parsed` converts device-native configuration into structured data

A clean ACL workflow follows three steps. First, gather the current ACL as structured data. Second, store it in version control or `host_vars`. Third, change the structured data and push it back with the desired state.

```yaml
- name: Gather ACLs from a router
  hosts: ios1
  gather_facts: false
  tasks:
  - name: Read ACL configuration
    cisco.ios.ios_acls:
      config: []
      state: gathered
    register: acl_config

  - name: Save gathered ACLs
    ansible.builtin.copy:
      content: "{{ acl_config.gathered | to_nice_yaml }}"
      dest: host_vars/ios1/acls.yml
```

```yaml
- name: Override ACLs with updated structured data
  hosts: ios1
  gather_facts: false
  tasks:
  - name: Push ACL changes
    cisco.ios.ios_acls:
      config: "{{ acls_on_device }}"
      state: overridden
```

This model gives two benefits. It preserves idempotency because Ansible compares intended and running state before changing the device. It also keeps automation readable because the playbook manipulates structured YAML instead of long platform command strings. That pays off quickly in mixed-vendor environments, especially when configuration must be gathered, reviewed, and reapplied at scale.