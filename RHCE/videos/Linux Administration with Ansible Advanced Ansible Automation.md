# Linux Administration with Ansible: Advanced Ansible Automation
> [!NOTE]
> Explains how advanced Ansible practices (including templating, roles, collections, Vault, execution controls, and network resource modules) enable safer, scalable, reusable, and data-driven automation.

Red Hat Enterprise Linux 10 includes Python 3.12 and the `ansible-core` 2.16 Application Stream. Installing `rhel-system-roles` on a RHEL 10 control node also installs `ansible-core` as a dependency. The packaged engine supports Red Hat automation content, especially RHEL System Roles. Organisations that require broader vendor support, managed execution environments, centralised credentials, or controller services should use Red Hat Ansible Automation Platform.

A RHEL 10 control node needs an inventory, SSH access to managed nodes, and an account that can use `sudo` for privileged work. Under the RHEL support scope, the RHEL 10 control node and its System Roles manage RHEL 9 and RHEL 10 nodes. Mixed estates should confirm controller, target Python, collection, and product support before selecting an execution environment.

Project-specific settings belong in a reviewed `ansible.cfg`, not in undocumented shell state. `ansible-config dump --only-changed` reveals the effective differences from defaults, while `ansible --version` reports the configuration file and collection search paths in use. Each repository should also identify its inventory source, required collections, supported operating systems, and expected Ansible release.

Automation should progress through increasingly realistic checks:
- `ansible-playbook --syntax-check` catches structural YAML and playbook errors.
- `ansible-lint` detects many style, safety, and portability problems.
- `ansible-playbook --check --diff` previews changes where modules support those modes.
- Integration tests apply the automation to disposable RHEL 10 systems.
- A limited production batch confirms application health before wider deployment.

Check mode cannot prove that a syntactically valid configuration will work, and some modules cannot simulate every action. Tests must verify the resulting service, not only a successful Ansible return code.

Ansible automation rests on six related capabilities:
- Jinja templates turn variables and facts into host-specific text files.
- Roles organise reusable tasks, handlers, defaults, files, and templates.
- Collections distribute versioned modules, plugins, roles, and documentation.
- Ansible Vault encrypts secrets stored with automation content.
- Strategy and concurrency controls govern task execution across hosts.
- Network resource modules translate structured data into device configuration.

Playbooks should use fully qualified collection names, pin external content versions, keep secrets out of output, and pass syntax, lint, check-mode, and integration tests before production use.
## Jinja templating
Jinja renders text on the control node before Ansible transfers the result to a managed host. Managed RHEL 10 hosts therefore do not need a separate Jinja installation. The `ansible.builtin.template` module renders one result for each host, compares that result with the destination, and changes the destination only when the content or managed file attributes differ.

Jinja uses three delimiter forms:
- `{{ expression }}` prints a value.
- `{% statement %}` controls an `if` block, loop, or other operation.
- `{# comment #}` records a template comment without adding it to the result.

The `.j2` suffix provides a useful convention but does not control rendering. The module identifies a template through `src`. A playbook normally keeps templates in a nearby `templates/` directory, while a role keeps them in `roles/<role_name>/templates/`.

File ownership, permissions, and SELinux context require explicit attention on RHEL 10. Quoted modes avoid YAML number conversion errors. Validation should run before a critical file replaces its destination.

```yaml
- name: Deploy a validated chrony configuration
  ansible.builtin.template:
    src: chrony.conf.j2
    dest: /etc/chrony.conf
    owner: root
    group: root
    mode: '0644'
    backup: true
    validate: /usr/sbin/chronyd -p -f %s
  notify: Restart chronyd

handlers:
  - name: Restart chronyd
    ansible.builtin.systemd_service:
      name: chronyd
      state: restarted
```

The `validate` command receives a temporary file through `%s`. Ansible invokes the command without a shell, so pipes, redirection, and shell expansion do not work in this field. Complex validation belongs in a separate task or script. `backup: true` retains the previous file, but version control and tested rollback procedures still remain necessary.

Templates can read play variables, inventory variables, role defaults, registered results, magic variables, and gathered facts. Host-scoped data creates host-specific output. General facts work better than hard-coded interface names because interface names and available devices differ across RHEL 10 installations.

```jinja2
node_name={{ inventory_hostname }}
node_address={{ ansible_facts['default_ipv4']['address'] }}
role={{ host_role | default('application') }}
```

Facts must exist before a template uses them. A play with `gather_facts: false` must obtain the required data through another task or inventory. Bracket notation handles keys safely, especially when a key contains punctuation or resembles a method name.

Filters transform values without changing the source variable. Common filters supply defaults, convert dictionaries to item lists, combine data, select values, and format YAML or JSON. A default should represent a safe operational choice. Hiding a misspelt variable behind an unsuitable default can create a valid but incorrect configuration.

```jinja2
{% for server in ntp_servers | default([]) %}
server {{ server }} iburst
{% endfor %}
```

Filters can be chained, and some accept arguments. The order changes the result because each filter receives the previous filter's output. `dict2items` converts a mapping into iterable `key` and `value` records. `combine` merges mappings. `unique`, `sort`, `select`, and `map` shape sequences. `mandatory` can force an otherwise optional variable to fail when omitted. Data type checks and explicit conversion prevent strings such as `"false"` from being treated as truthy values.

Undefined variables should usually stop execution. A `default` filter suits an intentionally optional value, while `is defined` suits a conditional branch. Broad defaults applied throughout a template can conceal inventory mistakes. Assertions placed before templating provide clearer failures:

```yaml
- name: Validate time configuration inputs
  ansible.builtin.assert:
    that:
      - ntp_servers is defined
      - ntp_servers | length >= 2
      - timezone_name is string
    fail_msg: Time configuration variables are incomplete
```

Conditions select configuration by host properties. Tests such as `defined`, `string`, and `version` express intent more accurately than improvised string comparisons. Version values require a version-aware test because lexical ordering treats values such as `10.0` and `9.5` incorrectly.

```jinja2
{% if ansible_facts['distribution'] == 'RedHat'
      and ansible_facts['distribution_major_version']
      is ansible.builtin.version('10', '>=') %}
platform_family=rhel10
{% endif %}
```

Loops can traverse ordinary lists, dictionaries converted with `dict2items`, inventory groups, and `hostvars`. A template that reads another host through `hostvars` can access only facts or variables already available for that host. Fact caching may supply such data across plays, but its security and freshness settings need deliberate control.

Inventory group iteration should use the intended group explicitly, such as `groups['application']`. The loop variable should carry a descriptive name when nested loops appear. Jinja's `loop.index`, `loop.first`, and `loop.last` values can generate separators or ordered entries without extra state. Templates should avoid implementing complex business logic. Playbooks or filter plugins can prepare a clean data model before Jinja renders it.

Whitespace control can remove unwanted blank lines, although aggressive use of `{%-` and `-%}` can join directives and produce invalid configuration. Rendered configuration should undergo application-specific validation and a test deployment.

Custom templating remains useful for applications without maintained automation content. For standard RHEL 10 configuration, RHEL System Roles usually provide a safer interface, cross-version handling, and supported defaults. Time synchronisation illustrates the distinction. RHEL 10 uses `chronyd`, reads `/etc/chrony.conf`, and supports NTP and Network Time Security. The `redhat.rhel_system_roles.timesync` role installs and configures the provider, although it replaces the provider configuration, so the playbook must declare every setting that needs to remain.

```yaml
- name: Configure RHEL 10 time synchronisation
  hosts: rhel10
  become: true
  tasks:
    - name: Apply the supported timesync role
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.timesync
      vars:
        timesync_ntp_servers:
          - hostname: time.example.com
            prefer: true
            trusted: true
            iburst: true
```

An NTS source can set `nts: true`. A configuration should not mix authenticated NTS sources with unauthenticated fallback sources. Verification should check `chronyc tracking`, `chronyc sources`, and, for NTS, `chronyc -N authdata`.
## Roles
A role groups related automation under a known directory structure. Current Ansible documentation identifies seven main standard directories:
- `tasks/` contains the task sequence.
- `handlers/` contains actions notified by changed tasks.
- `templates/` contains Jinja templates.
- `files/` contains static files and scripts.
- `defaults/` contains easily overridden variables.
- `vars/` contains high-precedence role variables.
- `meta/` contains dependencies, argument specifications, and metadata.

Roles can also carry custom plugins and modules. Test content may live beside a role or in a dedicated test framework, but `tests/` is not one of the seven main directories. Unused directories can be omitted. Each relevant directory normally exposes `main.yml`, though tasks can import or include smaller files.

`ansible-galaxy role init role_name` creates a role skeleton. A manually created role remains valid when it contains only the directories it needs. Roles stored in a collection gain a fully qualified name such as `redhat.rhel_system_roles.timesync`.

Each role should deliver one coherent capability. A time role should configure time synchronisation, while a web role should install and configure the web service. Smaller roles improve reuse, testing, review, and replacement. Variable names should include a role-specific prefix, such as `web_listen_port`, to reduce collisions.

A conventional service role follows a short control flow. Its tasks install the package, validate and deploy configuration, enable the service, and notify a handler only when configuration changes. Defaults expose supported settings, while `meta/argument_specs.yml` validates the public role interface. Platform-specific variables can live in separate files selected by gathered facts, but the role should fail clearly on unsupported systems rather than apply an approximate configuration.

```yaml
- name: Load RHEL 10 variables
  ansible.builtin.include_vars:
    file: rhel10.yml
  when:
    - ansible_facts['distribution'] == 'RedHat'
    - ansible_facts['distribution_major_version'] == '10'
```

`ansible.builtin.package` provides a distribution-neutral interface when package names and behaviour align across platforms. `ansible.builtin.dnf` exposes RHEL-specific package features when the role needs them. Service tasks should use the service unit name, which can differ from the package name. For chrony on RHEL 10, the package is `chrony`, and the unit is `chronyd`.

Role defaults expose supported inputs. Role vars should hold internal values that callers should not normally replace. Inventory, play, task, role parameter, and command-line sources participate in a detailed precedence system, so high-precedence role vars should not serve as ordinary defaults. Role argument specifications can validate expected parameter names and types before tasks run.

Handlers enter the play's global handler scope. Duplicate handler names can therefore conflict across roles. A role can use a qualified notification such as `role_name : handler_name`, or handlers can share a `listen` topic. Handlers run after notified task sections and only when a notifying task reports a change.

Roles support three main reuse forms:
- `roles:` performs static reuse at play level and runs those roles before ordinary tasks.
- `ansible.builtin.import_role` performs static reuse at its task position.
- `ansible.builtin.include_role` performs dynamic reuse at run time and supports conditions on the include.

Static imports expose their tasks during playbook parsing. Conditions and tags attached to an import propagate to imported tasks. Dynamic includes evaluate at run time. A condition on `include_role` controls whether Ansible includes the role. A tag on a dynamic include applies to the include itself, not automatically to every task inside the role. Selective execution requires matching tags on the include and the target tasks, or an explicit `apply` configuration when broad inheritance is intended.

Dependencies in `meta/main.yml` run before the dependent role. Dependencies can recurse, and variables from several roles can interact, so teams should keep dependency graphs shallow and test the complete play. Version control should retain role source, requirements, tests, and change history together.

Migrating a large playbook into roles starts with cohesive task groups. Associated handlers, templates, files, and defaults move with each group. The remaining playbook coordinates the roles and application-specific tasks. The migration should preserve task order, notification behaviour, variable precedence, privilege escalation, and idempotence.

Idempotence means that a second run against the desired state reports no unnecessary changes. Command and shell tasks often break this property unless `creates`, `removes`, `changed_when`, or a purpose-built module supplies state awareness. A role test should run twice and treat unexpected changes on the second run as a defect. Handlers should not restart services on every execution.
## Collections and content sources
`ansible-core` supplies the automation language, runtime, command-line tools, and the `ansible.builtin` collection. External collections supply most vendor, cloud, database, and network content. A fully qualified collection name identifies the exact source, as in `community.general.timezone` or `cisco.ios.ios_facts`, and avoids collisions between plugins with the same short name.

RHEL 10 administrators can install `rhel-system-roles`, which places the `redhat.rhel_system_roles` collection under `/usr/share/ansible/collections/ansible_collections/`. Ansible Automation Platform users can obtain supported content from Red Hat Automation Hub and package it in an execution environment. Ansible Galaxy hosts community content. Private Automation Hub can curate approved content for an organisation.

The `ansible` community package and `ansible-core` are different distributions. `ansible-core` contains the runtime and built-in content. The larger community package selects many community collections, but it does not turn those collections into Red Hat-supported RHEL content. A project should declare each external collection it actually uses instead of relying on an incidental global installation.

Community content requires technical review. Maintainer activity, release history, supported platforms, documentation, tests, dependencies, open defects, licences, and source quality provide stronger evidence than download counts alone. A production project should pin tested versions and review breaking changes before an upgrade.

```yaml
collections:
  - name: community.general
    version: '>=10.0.0,<11.0.0'
  - name: cisco.ios
    version: '11.4.2'
```

```text
ansible-galaxy collection install -r requirements.yml
```

A single `requirements.yml` can contain both `roles:` and `collections:`. In that case, `ansible-galaxy install -r requirements.yml` installs both kinds. Project-local collection directories and execution environments improve repeatability. Signed collections should undergo signature verification when the distribution server provides signatures.

Only one version of a given collection can occupy one installation path at a time. Separate projects or execution environments can carry different tested versions. Configuration should use the current `collections_path` setting or the corresponding search-path controls for the installed Ansible release.

Collection dependencies in `MANIFEST.json` can install transitively, but a project-level requirements file should still record direct dependencies. Builds should preserve the resolved versions in an execution environment image or another repeatable artefact. Pulling an unpinned latest release during a production job makes testing and rollback unreliable.
## Ansible Vault
Ansible Vault protects data at rest with symmetric encryption. It can encrypt a complete file or an individual string embedded in YAML. It does not protect a secret after Ansible decrypts it, sends it to a module, writes it to a managed host, or exposes it in output.

Complete files suit variable sets and structured data:

```text
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-vault view secrets.yml
ansible-vault rekey secrets.yml
```

`ansible-vault encrypt existing.yml` encrypts an existing plaintext file in place. `ansible-vault decrypt` restores plaintext and should be used only when the resulting file can be protected. Backups, editor swap files, terminal capture, and repository history can retain earlier plaintext copies.

`encrypt_string` keeps variable names visible while encrypting selected values. A prompt or standard input avoids recording the secret in shell history.

```text
ansible-vault encrypt_string --vault-id dev@prompt --stdin-name db_password
```

Vault IDs associate encrypted content with labels such as `dev`, `prod`, or `network`. A source can be a prompt, protected password file, or executable client script. Labels guide secret selection but do not enforce one password per label by default. Multiple `--vault-id` options support content encrypted with different passwords.

The correct configuration keys under `[defaults]` are `vault_password_file` and `vault_identity_list`. Their environment equivalents include `ANSIBLE_VAULT_PASSWORD_FILE` and `ANSIBLE_VAULT_IDENTITY_LIST`. Command-line forms include `--vault-password-file` and repeated `--vault-id` options. Password files must stay outside source control, use restrictive permissions, and receive protection equivalent to the secrets they unlock. A client script can retrieve a password from an external secret service at run time.

Tasks that handle secret values should set `no_log: true` when their arguments or results could expose those values. That setting reduces normal output exposure but does not protect debug-level internals in every circumstance. Playbooks should avoid debug tasks that print secrets, broad logging of task arguments, and diffs of secret-bearing files.

Vaulted templates and files decrypt during use. A file supplied to modules such as `copy` or `template` reaches the managed host as plaintext when the target service needs plaintext. File permissions, ownership, SELinux labels, backups, remote logs, and application diagnostics must protect that deployed copy. Vault encryption alone does not secure the destination.

Whole-file encryption hides variable names and can hinder review. A common layout keeps public variable names in one file and sensitive values in a separate vaulted file with a `vault_` prefix. The public value refers to its encrypted counterpart. This pattern keeps the role interface visible without exposing the secret.

Vault works well for encrypted data stored with a project. A dedicated secret manager offers stronger access control, audit, rotation, and short-lived credential workflows. Whichever method an organisation selects, it should separate duties, restrict decryption access, rotate compromised credentials, and test recovery from lost Vault passwords.
## Parallelism and execution control
Ansible uses the `linear` strategy and five forks by default. Linear execution runs a task across the current host set before starting the next task. The effective concurrency never exceeds the smallest applicable limit from the host batch, available forks, and task throttle.

`forks` sets the maximum number of worker processes. A larger value can shorten execution, but no universal host-to-memory formula determines a safe value. Module payloads, fact gathering, connection plugins, network latency, controller CPU, controller memory, and target capacity all influence performance. Teams should benchmark representative jobs, increase gradually, and observe controller and target health.

The principal strategies serve different needs:
- `linear` preserves task-by-task coordination across hosts.
- `free` lets each host advance through tasks without waiting for slower hosts.
- `host_pinned` keeps a worker with one host until that host finishes, while limiting active hosts through forks and batches.
- `debug` follows linear execution and opens an interactive debugging session on failure.

Independent configuration tasks can benefit from `free`. Cross-host orchestration, clustered changes, and tasks with shared dependencies often require `linear`, explicit delegation, or other coordination. Interactive debugging does not suit unattended production jobs.

`serial` divides a play into host batches. It accepts a number, percentage, or sequence of batch sizes and supports rolling changes. Failure thresholds apply to each batch, so production playbooks should define suitable error handling and health checks.

`throttle` limits workers for one task or block. It can reduce concurrency below `forks` or `serial`, but it cannot increase concurrency. It does not implement a timed request rate. An API with a quota expressed as requests per second may require retries, delay logic, or purpose-built rate handling in addition to `throttle`.

`run_once`, `order`, delegation, asynchronous tasks, and failure controls further shape execution. `run_once` operates once per current batch when `serial` applies, which can surprise a playbook that expects one execution for the entire inventory. Production automation should make host selection and shared-state operations explicit.

A rolling service change can combine a conservative batch with a task-specific limit:

```yaml
- name: Update application servers
  hosts: application
  serial: 10%
  max_fail_percentage: 20
  tasks:
    - name: Apply the CPU-intensive migration
      ansible.builtin.command: /usr/local/sbin/migrate-data
      throttle: 2
```

The batch protects overall service capacity, while the throttle protects the shared dependency used by the migration. A load balancer drain, health probe, and rejoin step should surround the update when traffic must stay available. `any_errors_fatal` can halt all active hosts after a critical failure, while `max_fail_percentage` defines the tolerated failure rate within the current batch.
## Network resource modules
Ansible can manage network devices through native configuration modules, rendered command templates, or resource modules. Native modules expose device commands directly. Jinja templates generate those commands from variables. Resource modules accept structured data for a defined resource such as interfaces, VLANs, routes, or access control lists.

The `cisco.ios` collection does not ship in `ansible-core`. A project must install and pin it before using `cisco.ios.ios_facts` or `cisco.ios.ios_acls`. Network facts can gather selected resources as structured data under `ansible_facts['network_resources']`. Resource module output can also return `before`, `after`, and generated native commands.

Network plays normally disable ordinary server fact gathering and call the platform facts module directly. The connection and network operating system must match the device and collection documentation.

```yaml
- name: Gather selected Cisco IOS resources
  hosts: edge_routers
  gather_facts: false
  tasks:
    - name: Read interfaces and ACLs
      cisco.ios.ios_facts:
        gather_subset:
          - min
        gather_network_resources:
          - interfaces
          - l3_interfaces
          - acls
```

The resulting structured facts can support compliance checks, backups, migration, and input to a resource module. An observed configuration is not automatically the intended configuration. A controller should compare observed facts with reviewed source data and then calculate a controlled change.

Common resource states include:
- `merged` adds supplied configuration without removing unspecified entries.
- `replaced` replaces the specified resource subsection.
- `overridden` makes the supplied resource configuration authoritative across the managed scope.
- `deleted` removes selected configuration.
- `gathered` reads on-device configuration as structured data.
- `rendered` converts structured data into native commands without changing a device.
- `parsed` converts supplied native configuration into structured data without changing a device.

Supported states vary by module and collection version. Module documentation and `ansible-doc` provide the authoritative argument model.

`rendered` and `parsed` support offline workflows. Rendering shows the native commands that structured data would produce. Parsing converts supported native configuration into the resource model. Neither operation proves that a live device will accept the configuration. Platform version differences, unsupported commands, and surrounding configuration still require device-specific tests.

An ACL change that permits NTP traffic should add the narrowest required rule and preserve every management path. `merged` normally carries less risk than overriding all ACLs when a single new entry is required.

```yaml
- name: Permit NTP from the RHEL server segment
  cisco.ios.ios_acls:
    config:
      - afi: ipv4
        acls:
          - name: WAN_IN
            acl_type: extended
            aces:
              - sequence: 15
                grant: permit
                protocol: udp
                source:
                  address: 10.10.6.0
                  wildcard_bits: 0.0.0.255
                destination:
                  any: true
                  port_protocol:
                    eq: ntp
    state: merged
```

ACL behaviour depends on attachment direction, return traffic, existing entries, and the device platform. A safe change gathers the current state, records an independent backup, renders or checks the proposed commands, reviews the diff, tests from an out-of-band management path, applies to a limited device set, and verifies both time synchronisation and administrative access. Saving live configuration directly into auto-loaded `host_vars` can blur observed state and intended state. Separate audit snapshots from authoritative inventory data.

Resource modules improve consistency and idempotence, but they do not remove the need for network design knowledge. Teams must understand the module's scope, the device's command semantics, and the consequences of each state before applying a change.
## Operational baseline
Reliable RHEL 10 automation uses a repeatable control environment, supported RHEL System Roles where available, explicit collection versions, role argument validation, and fully qualified names. Templates and resource modules should validate generated configuration before deployment. Vault or an external secret service should protect stored credentials, while `no_log` and controlled logging should protect run-time output. Concurrency should follow measured capacity and service limits. Check mode, diff review, staged rollout, application health checks, and verified rollback should precede broad production changes.