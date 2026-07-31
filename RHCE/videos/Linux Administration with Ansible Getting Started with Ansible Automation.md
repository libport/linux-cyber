# Linux Administration with Ansible: Getting Started with Ansible Automation
> [!NOTE]
> Introduction on Ansible automation through a repeatable multi-platform lab, explaining how controllers, inventories, variables, modules, SSH, privilege escalation, and playbooks enable consistent, idempotent Linux administration.

Red Hat Enterprise Linux 10 provides `ansible-core` 2.16 through AppStream. A RHEL 10 control node can manage RHEL 9 and RHEL 10 hosts. Managed RHEL 10 hosts do not require Ansible, but modules generally require Python, and RHEL 10 supplies Python 3.12 as its default implementation. Administrators can install the control software and supported RHEL System Roles together:

```shell
sudo dnf install ansible-core rhel-system-roles
ansible --version
```

The `ansible-core` package contains the command-line tools, the `ansible.builtin` collection, and a small set of plugins. Other collections remain separate. Fully qualified collection names, such as `ansible.builtin.template` and `cisco.ios.ios_acls`, identify content unambiguously.

RHEL subscriptions support the packaged RHEL System Roles and Red Hat-generated automation within the stated support scope for `ansible-core`. Organisations that require supported automation across broader workloads can use Red Hat Ansible Automation Platform and its execution environments.
## Jinja templating
Ansible renders Jinja templates on the control node, then transfers the resulting files to managed hosts. Managed hosts therefore need neither Jinja nor the source template. The `.j2` suffix signals a template by convention, but Ansible does not require it.

`ansible.builtin.copy` transfers the same content to every target. `ansible.builtin.template` evaluates the source separately for each target, so inventory values and facts can produce a different result on every host. Ansible compares the rendered content with the destination and reports a change only when deployment alters the managed file. This idempotent behaviour allows a handler to restart a service only after its configuration changes.

Jinja separates dynamic content from literal text:

| Syntax | Purpose |
|---|---|
| `{{ expression }}` | Evaluates and inserts a value |
| `{% statement %}` | Controls flow with constructs such as `if` and `for` |
| `{# comment #}` | Adds a template comment that does not appear in the result |

Templates can read play variables, inventory variables, role defaults, registered results, facts, and magic variables. Host-specific data produces host-specific files. The `groups` variable lists inventory groups and their members. The `hostvars` variable exposes variables for another inventory host. Facts accessed through `hostvars` must already exist through gathering or a fact cache.

Variable scope should express the intended variation. A play variable normally gives every host in the play the same value. `group_vars` can describe an environment, location, or server class. `host_vars` should hold exceptional host-level data. Facts describe observed state. A template that depends on another host's fact also depends on that host participating in fact gathering or on a valid fact cache. Inventory variables often provide a stronger source for stable service addresses because they do not depend on discovery.

RHEL 10 network interfaces can use names such as `enp1s0`, so templates should not assume `eth0` or `eth1`. The default route fact usually offers a more durable source for the primary address:

```jinja2
node_ip={{ ansible_facts.default_ipv4.address }}

{% for host in groups['app_servers'] %}
server {{ host }} {{ hostvars[host].ansible_facts.default_ipv4.address }}
{% endfor %}
```

Bracket notation remains safer when a key contains punctuation or conflicts with a dictionary attribute. Clear variable names and shallow data structures also reduce template errors.

Filters transform values without altering the original variables. Common filters include `default`, `dict2items`, `items2dict`, `map`, `selectattr`, `unique`, `sort`, `to_json`, and `to_nice_yaml`. A default can handle an undefined variable:

```jinja2
character_set={{ character_set | default('utf8') }}
```

The optional second argument to `default` also replaces false values, empty strings, and empty collections. That behaviour can conceal valid input, so templates should enable it only when false-like values truly require replacement.

Conditionals select content through tests, comparisons, and logical operators. Tests use forms such as `value is defined` and `release is version('10', '>=')`. RHEL-specific logic should use gathered facts instead of obsolete CentOS assumptions:

```jinja2
{% if ansible_facts.distribution == 'RedHat'
      and ansible_facts.distribution_major_version | int >= 10 %}
platform_generation=10
{% endif %}
```

Jinja `for` loops iterate over lists, dictionaries, and inventory groups. Whitespace controls such as `{%-` and `-%}` remove adjacent whitespace, but aggressive trimming can join configuration lines and invalidate a file. Templates should keep policy and complex data manipulation in variables or tasks, leaving Jinja to format the desired configuration.

A loop can generate repeated directives from a list:

```jinja2
{% for source in timesync_sources %}
server {{ source.hostname }}{% if source.iburst | default(false) %} iburst{% endif %}
{% endfor %}
```

Data should remain typed until final formatting. Numeric comparisons should convert text facts with `| int`, Boolean values should remain `true` or `false`, and lists should remain lists rather than comma-separated strings. `dict2items` can turn a mapping into loopable key-value records, while `items2dict` performs the reverse transformation. Filters can be chained, with each filter receiving the previous result.

The `ansible.builtin.template` module manages ownership, permissions, backups, and atomic replacement. Its `validate` option runs a command against a temporary file before deployment. The command must contain `%s`, which Ansible replaces with the temporary path. Shell features such as pipes do not work directly because Ansible passes the command securely rather than through a shell.

```yaml
- name: Deploy sudo policy
  ansible.builtin.template:
    src: admins.j2
    dest: /etc/sudoers.d/admins
    owner: root
    group: root
    mode: "0440"
    backup: true
    validate: "/usr/sbin/visudo -cf %s"
```

Sensitive configuration may require `mode: "0600"`, an appropriate SELinux context, and `no_log: true` on tasks that could expose secrets. A backup aids recovery, but version control, syntax checks, check mode, and application-specific validation provide stronger assurance.

Modes should use quoted strings, such as `"0644"`, to avoid YAML number interpretation. A template should normally set `owner`, `group`, and `mode` explicitly. Critical configuration also needs a handler, a validation command where available, and a post-change service check. `backup: true` creates a timestamped copy on the managed host, but administrators still need retention controls and a recovery procedure.
## Roles and modular configuration
A role groups related tasks, handlers, templates, files, variables, and metadata behind a stable interface. A focused role can configure one service or capability, while a playbook composes several roles into a complete deployment. This separation supports reuse, testing, review, and version control.

Ansible recognises seven main role directories:

| Directory | Content |
|---|---|
| `tasks/` | Tasks, normally loaded from `main.yml` |
| `handlers/` | Handlers notified by changed tasks |
| `templates/` | Jinja templates rendered by template tasks |
| `files/` | Static files used by copy or script tasks |
| `defaults/` | Easily overridden role input defaults |
| `vars/` | High-precedence role variables |
| `meta/` | Dependencies, role metadata, and argument specifications |

Unused directories can remain absent. `ansible-galaxy role init role_name` creates a conventional skeleton, while a small role can use only the directories it needs. `meta/argument_specs.yml` can define and validate the role's public inputs.

Role defaults have very low variable precedence and suit settings that consumers may override. Role vars have high precedence and suit internal constants. Namespaced variables such as `web_tls_port` reduce collisions. A role should document each public variable, its type, its default, and any constraints. Secrets should not appear in defaults.

Migration from a large playbook starts by finding cohesive task groups. Package installation, configuration deployment, service management, variables, files, templates, and handlers for one capability can move into one role. Application orchestration and environment ordering should remain in the playbook. This boundary keeps a role generic enough for reuse without forcing unrelated deployment policy into it.

Every reference must move with the task group. A renamed, namespaced variable requires updates in tasks, templates, handlers, and calling playbooks. A copied file belongs under `files/`, while a rendered file belongs under `templates/`. Role-relative lookup lets tasks use a short source name. The migration is complete only after the original tasks and variables have been removed, otherwise variable precedence can mask defects.

Handlers run only after a notifying task reports a change. A handler name or `listen` topic must match the notification. A configuration task should notify a restart or reload handler instead of restarting a service on every run. Descriptive task names, fully qualified module names, and idempotent modules keep role output understandable.

Roles can enter a play in four ways:

| Method | Processing and order |
|---|---|
| `roles:` | Static processing, after `pre_tasks` and before normal tasks |
| `ansible.builtin.import_role` | Static processing at the task position |
| `ansible.builtin.include_role` | Dynamic processing when execution reaches the task |
| Role dependency | Runs before the dependent role |

Static imports expose their task structure during parsing. Conditions and tags attached to an import apply to the imported tasks. Dynamic includes support runtime conditions and loops. Task keywords on `include_role` apply to the include itself unless the role receives them through `apply`.

Ansible executes a play in a defined sequence. It runs `pre_tasks`, their notified handlers, roles listed under `roles:` in listed order, normal tasks, notified handlers, `post_tasks`, and their notified handlers. A role dependency runs before the role that declares it. Recursive dependencies can create hidden coupling, so explicit playbook composition often remains easier to understand.

In `ansible-core` 2.16, role parameters normally remain private instead of leaking into the wider play as older releases allowed. `include_role` also keeps role defaults and vars private unless `public: true` exposes them to later tasks. Consumers should pass inputs deliberately and should not rely on accidental visibility.

Tags on a role under `roles:` or on `import_role` propagate to every task in that role. Tags on `include_role` do not automatically propagate. Selective execution with a dynamic include requires tags on both the include and the selected role tasks, or an `apply` mapping that adds tags to all included tasks.

`ansible-playbook site.yml --tags install` selects tagged work, while `--skip-tags` excludes it. Tags can bypass prerequisites and handlers, so they should support diagnostics or bounded operations rather than define the only valid execution path. Role tests should include `--syntax-check`, `ansible-lint`, idempotence checks, supported RHEL versions, and failure cases for invalid arguments.

RHEL 10 supplies supported System Roles for common administration. The `timesync` role configures `chronyd`, which RHEL 10 uses for NTP. It provides a safer starting point than a hand-built Chrony role:

```yaml
---
- name: Configure time synchronisation
  hosts: rhel10
  become: true
  tasks:
    - name: Configure trusted NTP sources
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.timesync
      vars:
        timesync_ntp_servers:
          - hostname: time.example.com
            trusted: true
            prefer: true
            iburst: true
          - hostname: 0.rhel.pool.ntp.org
            pool: true
            iburst: true
```

The role replaces the provider's time configuration, so the variable set must describe every setting that the host must retain. RHEL 10 installs the `chrony` package by default, runs the `chronyd` service, and uses `/etc/chrony.conf`. `chronyc sources` and `chronyc tracking` verify synchronisation. NTS-capable environments can configure authenticated time sources through the same role.

A custom role remains appropriate when no supported role covers the required configuration. For Chrony, such a role would install `chrony` with `ansible.builtin.package`, render `/etc/chrony.conf`, notify a handler for `chronyd`, enable and start the service, and verify its sources. The supported System Role already handles provider selection and cross-version details, which reduces local maintenance.
## Collections and content sharing
A collection packages modules, plugins, roles, playbooks, documentation, and metadata under a namespace and collection name. An FQCN combines the namespace, collection, and content name. `ansible.builtin.package` comes with `ansible-core`, while `community.general.timezone` and `cisco.ios.ios_facts` require their respective collections.

Short module names may still work through built-in lookup or compatibility redirection, but they can collide across collections and can obscure the required dependency. FQCNs remain the recommended form. Roles inside collections also use three-part names, such as `redhat.rhel_system_roles.timesync`.

Ansible Galaxy distributes community content. Red Hat Automation Hub distributes supported certified content for Ansible Automation Platform. Private automation hub can curate approved versions for an organisation. Before adoption, maintainers should review source code, licence terms, supported platforms, release activity, tests, dependencies, documentation, issue history, and change logs. Popularity alone does not establish production suitability.

The collection subcommand installs collections:

```shell
ansible-galaxy collection install cisco.ios
ansible-galaxy collection list
```

Standalone roles use the role subcommand:

```shell
ansible-galaxy role install namespace.role_name
ansible-galaxy role install namespace.role_name --roles-path ./roles
```

Projects should record direct dependencies and tested versions in `requirements.yml`:

```yaml
---
collections:
  - name: cisco.ios
    version: "11.4.2"
  - name: ansible.netcommon
    version: "8.6.0"
```

```shell
ansible-galaxy collection install -r requirements.yml
```

By default, Ansible installs collections under `~/.ansible/collections`. A project can install them under `collections/ansible_collections/` beside its playbooks, which isolates dependencies and supports repeatable builds. Ansible Automation Platform execution environments go further by packaging `ansible-core`, collections, Python packages, and system dependencies in a container image.

Different collection paths can contain different versions. Search-path order then controls which copy Ansible loads. Project-local installation or a versioned execution environment avoids accidental selection from a user-level path. Teams should test upgrades before changing pinned versions and should review transitive dependencies as carefully as direct ones.

Collections installed manually do not automatically upgrade with `ansible-core`. An upgrade can alter module arguments, return values, or platform support even when a playbook stays unchanged. A controlled workflow updates one dependency set, rebuilds the execution environment, runs automated tests, and promotes the tested image by immutable tag or digest.
## Ansible Vault
Ansible Vault encrypts files or individual YAML values at rest. It allows encrypted secrets to remain beside playbooks in source control. Vault does not provide runtime access control, protect secrets after decryption, or prevent a task from printing a value. Repository permissions, credential controls, and careful task output remain essential.

`ansible-vault create`, `edit`, `view`, `encrypt`, `decrypt`, and `rekey` manage encrypted files. `encrypt_string` produces an encrypted YAML value. A prompt avoids placing plaintext in shell history:

```shell
ansible-vault create --vault-id prod@prompt group_vars/prod/vault.yml
ansible-vault encrypt_string --vault-id prod@prompt --stdin-name db_password
ansible-playbook --vault-id prod@prompt site.yml
```

An existing file can be encrypted or assigned a new password:

```shell
ansible-vault encrypt --vault-id prod@prompt group_vars/prod/secrets.yml
ansible-vault rekey --vault-id prod@prompt --new-vault-id prod_new@prompt group_vars/prod/secrets.yml
```

Playbooks that use several identities can supply each one:

```shell
ansible-playbook \
  --vault-id dev@dev-password-client \
  --vault-id prod@prod-password-client \
  site.yml
```

Vault IDs associate encrypted content with labels such as `dev` and `prod`. Each label can obtain its password from a prompt, a protected file, or an executable password-client script. Labels guide password selection but do not enforce separation by themselves. Separate credentials and access policies must establish that boundary.

A password file must stay outside source control and use restrictive permissions. A password-client script can retrieve the secret from an external secret manager at runtime. Configuration keys such as `vault_identity_list` and `vault_password_file` can remove repeated command options, but they must not embed plaintext credentials in a committed `ansible.cfg`.

Whole-file encryption suits variable files whose contents are all sensitive. Encrypted strings suit files that need readable names and non-secret context. File rekeying changes the encryption password. Individual encrypted strings cannot use `rekey`, so administrators must encrypt them again.

Whole-file encryption can protect any file content, but automatic decryption depends on how Ansible loads or transfers that file. Encrypted variable files loaded through `vars_files` decrypt transparently after Ansible obtains the correct secret. An encrypted scalar carries the `!vault` tag and decrypts when Ansible evaluates that value. Neither form should hold a password hash when the target system expects a one-way hash instead of a recoverable secret.

Tasks that consume secrets should use `no_log: true` when their arguments or results could reveal data. Debug output can still expose secrets, and global debug mode can bypass normal redaction. Logs, registered variables, generated files, backups, and downstream commands all require review. Vault encryption complements these controls rather than replacing them.
## Parallelism and execution control
Ansible uses the `linear` strategy and five forks by default. Linear execution runs a task across the current host batch before starting the next task. Several controls alter this behaviour:

| Control | Effect |
|---|---|
| `forks` | Sets the maximum worker-process count |
| `serial` | Divides hosts into batches that complete the whole play in turn |
| `throttle` | Caps workers for one task or block |
| `strategy` | Selects how Ansible schedules tasks and hosts |
| `run_once` | Runs a task once for each serial batch |

The `free` strategy lets each host advance through the play without waiting for slower hosts. `host_pinned` also permits independent progress, but a worker stays with one host until that host completes the play. The `debug` strategy follows linear scheduling and opens an interactive debugger when a task fails.

`serial` accepts a number, percentage, or progression list, making it suitable for rolling changes. It also changes failure scope from the whole target set to the current batch. `throttle` can reduce concurrency below `forks` or `serial`, but it cannot increase either limit. Rate-limited APIs, shared databases, and CPU-heavy commands often need a low task-level throttle.

No fixed fork count suits every environment. Administrators should raise it gradually while measuring control-node CPU and memory, network capacity, SSH limits, target load, and external-service limits. A larger value can shorten independent work but can also amplify failures or exhaust dependencies.

The configuration file and command line can set the worker limit:

```ini
[defaults]
forks = 30
strategy = linear
```

```shell
ansible-playbook -f 30 site.yml
```

The effective concurrency cannot exceed the smallest applicable limit from `forks`, the current `serial` batch, and `throttle`. `throttle` limits concurrent workers, not requests per second. A service that enforces a true rate limit may also require pauses, retries with backoff, or an API-specific module.

A progressive rollout can use `serial: [1, 5, "25%"]`. The first batch contains one host, the second contains five, and subsequent batches contain 25% of the play's host set until completion. `max_fail_percentage` can stop a batch after excessive failures, while `any_errors_fatal` can stop all active hosts after a fatal task. Failure handling should reflect the application topology.

The `free` strategy suits independent host configuration such as time-client deployment. Coordinated application changes usually require `serial`, health checks, failure thresholds, and explicit load-balancer steps. `--syntax-check`, check mode, diff mode, canary batches, and idempotence tests should precede broad production runs.

`run_once` executes once in every serial batch, not once across the entire play. A truly global action needs an explicit host condition based on `ansible_play_hosts_all`, or a separate play that targets the control node. Delegated tasks can still run concurrently when several hosts delegate to the same destination, so shared controller files and central APIs may need `throttle: 1`.
## Network device automation
Network modules usually run on the control node and communicate through persistent connections such as `ansible.netcommon.network_cli`, `ansible.netcommon.netconf`, or `ansible.netcommon.httpapi`. Network devices do not need Python. Platform collections provide the connection, facts, command, configuration, and resource modules.

A Cisco IOS inventory normally sets the connection and platform at group level:

```yaml
---
ios:
  hosts:
    edge01:
      ansible_host: 192.0.2.10
  vars:
    ansible_connection: ansible.netcommon.network_cli
    ansible_network_os: cisco.ios.ios
    ansible_become: true
    ansible_become_method: enable
```

Credentials should come from SSH keys, an automation-platform credential, or encrypted variables. Inventory must not contain plaintext passwords.

Network configuration supports three broad approaches. Command and config modules send platform-native commands. Jinja templates generate those commands from variables. Resource modules accept structured data for a defined resource such as ACLs, interfaces, VLANs, or routes. Resource modules reduce command parsing and provide a similar operating model across supported platforms, although each platform retains its own schema and capabilities.

Command modules suit operational queries and exceptional commands. Config modules provide change detection and platform-aware configuration modes, but the author must still know the native syntax. Templates help when many commands share a stable structure, though large multi-vendor templates can become difficult to maintain. Resource modules provide the strongest data model when the collection implements the required feature.

`cisco.ios.ios_facts` can collect both basic facts and selected resource facts:

```yaml
- name: Gather IOS resource facts
  cisco.ios.ios_facts:
    gather_subset:
      - min
    gather_network_resources:
      - interfaces
      - l3_interfaces
      - acls
```

The requested resource data appears under `ansible_facts.network_resources`. A resource module with `state: gathered` can retrieve one resource directly and returns it through the registered result's `gathered` key.

The play should set `gather_facts: false` for network devices because the normal Linux setup module cannot run there. A platform facts task then requests only the needed subsets. Limiting fact collection reduces commands and execution time. `available_network_resources: true` can report which resource facts the installed collection supports.

Common resource states have distinct scopes:

| State | Result |
|---|---|
| `merged` | Adds or updates supplied data without replacing the whole resource |
| `replaced` | Replaces supplied resource subsections |
| `overridden` | Makes the supplied data authoritative for the complete resource |
| `deleted` | Removes selected resource configuration or restores defaults |
| `gathered` | Retrieves structured data from a device |
| `rendered` | Converts structured data into native commands offline |
| `parsed` | Converts supplied native configuration into structured data offline |

Modules support states according to their platform and resource. Documentation for the installed collection version remains authoritative. `overridden` requires special care because omitted configuration can disappear, including management access. A narrow ACL change normally favours `merged`, while an intentional full-resource declaration may justify `overridden`.

`merged` does not always update an existing item in place. Cisco IOS ACL entries with an existing sequence number can require `replaced`, and missing ACE sequence numbers can prevent idempotence. `replaced` changes only the named ACLs. `overridden` changes the named ACLs and deletes other non-default ACLs. `state: gathered` needs no empty `config` argument.

Controller-side backups require explicit local execution. Without delegation, `ansible.builtin.copy` targets the inventory host:

```yaml
- name: Gather ACL configuration
  cisco.ios.ios_acls:
    state: gathered
  register: acl_state

- name: Save ACL configuration on the control node
  ansible.builtin.copy:
    content: "{{ acl_state.gathered | to_nice_yaml }}"
    dest: "{{ playbook_dir }}/{{ inventory_hostname }}-acls.yml"
    mode: "0600"
  delegate_to: localhost
  become: false
```

A safe ACL workflow gathers the current state, stores a versioned backup, constructs the desired data, renders proposed commands offline, reviews the diff, applies the narrowest suitable state, and gathers the state again for verification. Stable ACE sequence numbers support idempotence. An NTP client-request exception must permit UDP destination port 123 before a broader deny rule, and the network path and ACL direction must match the intended traffic.

The `rendered` state can generate native commands without opening a device connection. The `parsed` state can turn saved native configuration into structured data. These non-changing states support review, migration, and testing. Check mode and diff mode add useful evidence where the module supports them, but neither mode substitutes for a device backup and an independent reachability check.

Out-of-band access and a tested rollback remain essential when automation changes routing, interfaces, authentication, or management ACLs. Structured data improves consistency, but it does not remove the operational risk of an incorrect desired state.