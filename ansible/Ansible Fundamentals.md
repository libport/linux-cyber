# Ansible fundamentals
> [!NOTE]
> Ansible enables secure, repeatable, agentless infrastructure automation through inventories, playbooks, roles, variables, and disciplined testing, deployment, troubleshooting, and secret-management practices.

Ansible is an open-source automation system for configuring hosts, deploying software, and orchestrating operational tasks. It expresses infrastructure policy as readable source files, applies that policy across managed systems, and reduces the inconsistency created by repeated manual work.

Configuration management keeps systems aligned with a defined state. Without it, servers that began with identical settings can diverge as administrators install packages, alter permissions, or change services. This configuration drift complicates maintenance, security, and incident response. Ansible addresses drift by applying repeatable automation to selected hosts.

Infrastructure as code supports this approach. Administrators describe infrastructure and configuration in machine-readable files, store those files in version control, review changes, and run the same automation across environments. The result is reproducible, auditable, and easier to recover than an undocumented sequence of manual commands.

Ansible commonly automates:
- package installation and removal
- operating system and application configuration
- file and account management
- service deployment and control
- cloud, network, and security device configuration
- multi-system application workflows
## Architecture
An Ansible environment contains a control node, one or more inventory sources, and managed nodes.

The control node runs Ansible commands. Current Ansible versions support Unix-like control nodes with an appropriate Python version, including Linux, macOS, BSD, and Windows through Windows Subsystem for Linux. Native Windows without WSL is not a supported control node.

Inventory identifies managed nodes and can attach variables to hosts and groups. A managed node may be a Linux or Windows host, network device, cloud resource, virtual machine, or other system supported by an Ansible collection and connection plugin.

Ansible is agentless in the usual sense because managed nodes do not run a persistent Ansible agent. The control node normally connects to POSIX hosts through SSH and can connect to Windows through WinRM, PSRP, or supported SSH configurations. Most modules that run in a POSIX environment need Python on the managed node, but network modules and some other modules do not. Windows modules normally use PowerShell and do not require Python on the Windows host.

During a task, Ansible selects the relevant plugin and module, prepares arguments, executes the work in the required context, and returns structured results. A playbook is not converted wholesale into Python. Modules, plugins, and connection methods implement different parts of the execution process.

Ansible normally pushes work from the control node. `ansible-pull` supports the inverse pattern by retrieving automation content and running it locally on a managed node. Larger organisations can use AWX or Red Hat Ansible Automation Platform for centralised execution, scheduling, credentials, access control, inventory, and reporting. AWX is the upstream open-source project, while Red Hat Ansible Automation Platform is the supported commercial product family.
## Installation and configuration
Two related community distributions are available:
- `ansible-core` contains the automation language, runtime, command-line tools, and built-in plugins.
- `ansible` is the larger community package. It includes `ansible-core` and a curated set of collections.

The community package suits broad exploration, while `ansible-core` suits environments that install only the required collections. Installation methods and supported Python versions vary by release and operating system. A maintained operating system package, `pipx`, or an isolated Python virtual environment avoids conflicts with the system Python installation. Production systems should pin and test Ansible and collection versions instead of installing an uncontrolled latest release.

Important commands include:
- `ansible` for ad hoc tasks
- `ansible-playbook` for running playbooks
- `ansible-config` for inspecting and generating configuration
- `ansible-inventory` for viewing parsed inventory
- `ansible-doc` for module and plugin documentation
- `ansible-galaxy` for installing and managing roles and collections
- `ansible-vault` for encrypting and editing protected content

`ansible --version` confirms the installed version and reports the active configuration file. `ansible-config dump --only-changed` shows effective non-default settings and helps identify unexpected configuration.

Ansible searches for `ansible.cfg` in this order and uses the first file it finds:
1. the path in the `ANSIBLE_CONFIG` environment variable
2. `ansible.cfg` in the current directory
3. `~/.ansible.cfg` in the current user's home directory
4. `/etc/ansible/ansible.cfg`

A project-local configuration keeps behaviour close to the automation that depends on it. Ansible avoids loading a configuration file from a world-writable current directory because an untrusted user could alter execution. Administrators should also treat environment variables and command-line overrides as part of the effective configuration.

A sample configuration can be generated with:

```bash
ansible-config init --disabled > ansible.cfg
```

A minimal project might contain:

```text
ansible-project/
  ansible.cfg
  inventory/
  group_vars/
  host_vars/
  playbooks/
  roles/
  requirements.yml
  README.md
```

The exact layout can vary. Consistent names, a documented entry point, separate environment inventories, and version-controlled dependencies make the project easier to operate.
## YAML essentials
Ansible playbooks and many inventory and variable files use YAML. YAML represents mappings, sequences, and scalar values through punctuation and indentation. It is case-sensitive, and indentation defines structure. Spaces should be used instead of tabs.

Common forms include:

```yaml
package_name: nginx
enabled: true

ports:
  - 80
  - 443

service:
  name: nginx
  state: started
```

The three-hyphen document marker is conventional but not required for every YAML file. Consistent two-space indentation is common in Ansible projects, although YAML accepts other consistent indentation widths.

Jinja expressions insert variables and evaluate limited logic:

```yaml
message: "Service {{ service_name }} runs on {{ inventory_hostname }}"
```

An expression that begins a YAML value should normally be quoted. Templates and expressions should remain readable. Complex data transformation often belongs in a filter, lookup, module, or preparatory task rather than a dense inline expression.
## Inventory and host selection
Inventory defines the hosts and groups available to Ansible. `/etc/ansible/hosts` is the default inventory path, but projects can select other sources through configuration or the `-i` option. Ansible can combine multiple sources, including INI files, YAML files, directories, dynamic inventory plugins, and executable inventory scripts.

A simple INI inventory looks like this:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
```

Ansible creates two implicit groups. `all` contains every host, and `ungrouped` contains hosts outside explicitly declared groups. A host can belong to several groups, such as its application role, region, and lifecycle environment.

Inventory aliases separate a convenient Ansible name from the network address:

```ini
[webservers]
web1 ansible_host=192.0.2.10 ansible_user=automation
```

Here, `web1` is `inventory_hostname`, while `ansible_host` provides the connection address. Inventory ranges such as `web[01:20].example.com` compact repeated names. They use Ansible's range syntax, not general regular expressions.

Dynamic inventory plugins query external systems such as cloud platforms and return current hosts, groups, and variables. Plugins are generally preferable to custom scripts because they use Ansible's inventory framework and configuration conventions. Static and dynamic sources can coexist.

Inventory determines which hosts exist. Patterns select hosts for a command or play. For example, `webservers`, `all`, a single alias, or a pattern combining groups can define the target set. The `--limit` option narrows that set at run time.

Tags solve a different problem. Tags label tasks, blocks, roles, imports, or includes so an operator can select or skip parts of a playbook. Tags do not assign tasks to inventory groups. The play's `hosts` pattern and any `when` conditions determine where a task runs.

Connectivity can be checked with the Ansible ping module:

```bash
ansible webservers -m ansible.builtin.ping
```

This command tests the Ansible connection and a usable remote execution environment. It is not an ICMP network ping. A `pong` result confirms that the relevant connection and module execution succeeded.
### Ad hoc operations
The `ansible` command executes one module without creating a playbook. Ad hoc operations suit investigation, a controlled one-off change, or a quick connectivity test:

```bash
ansible webservers -m ansible.builtin.command -a "uname -r"
ansible webservers -m ansible.builtin.copy -a "src=notice.txt dest=/tmp/notice.txt"
```

The first command reads each selected host's kernel release. Because `command` cannot determine whether that read changed the host, its result may need `changed_when: false` when expressed as a playbook task. The second command copies a file only when the destination needs an update because the copy module compares source and destination state.

Ad hoc commands use the same inventory, connection, authentication, module, and privilege systems as playbooks. Options can select a user, private key, inventory source, host limit, or privilege escalation. An ad hoc change is easy to omit from review and difficult to reproduce, so recurring or operationally important work belongs in a playbook under version control.

Before any broad ad hoc command, the host pattern should be verified with `--list-hosts`:

```bash
ansible 'webservers:&production' --list-hosts
```

The intersection pattern selects hosts that belong to both groups. Quoting the pattern prevents the shell from interpreting special characters. Operators should start with a narrow target, confirm the parsed host list, and expand scope only after the result matches the intended environment.
## Variables and facts
Variables make automation reusable across hosts and environments. Names may contain letters, numbers, and underscores, but cannot begin with a number. Clear, specific names reduce collisions and make precedence easier to understand.

Variables can come from inventory, `group_vars`, `host_vars`, a play, a role, included files, registered task output, facts, prompts, or command-line extra variables. The `host_group_vars` plugin loads YAML variable files associated with inventory groups and hosts. Separate files usually scale better than long inline inventory definitions.

For example:

```text
inventory/
  production.yml
  group_vars/
    webservers.yml
  host_vars/
    web1.yml
```

Values defined for a group apply to all hosts in that group. Host-specific values can override broader group policy where precedence rules allow. A child group's inventory variables override those of its parent group.

Variable precedence is detailed and should be checked whenever the same name appears in several places. Two anchor points prevent a common error:
- role defaults in `defaults/main.yml` have very low precedence and are designed for easy override
- extra variables supplied with `-e` or `--extra-vars` have the highest variable precedence

Role variables in `vars/main.yml` have high precedence and can be difficult for callers to override. Reusable roles should place user-configurable values in `defaults/main.yml` unless a strong reason requires stricter control.

Facts are data gathered about managed hosts, such as operating system details, interfaces, addresses, and memory. Ansible gathers facts at the start of a play by default. Automation can use them in variables, templates, and conditions. Fact gathering can be disabled when it is unnecessary or too expensive.

A task can save its result with `register`:

```yaml
- name: Read service state
  ansible.builtin.command: systemctl is-active nginx
  register: nginx_state
  changed_when: false

- name: Display service state
  ansible.builtin.debug:
    var: nginx_state.stdout
```

`register` is a task keyword, not a module. The debug module can display any available variable without a preceding registered result. Registered data is useful when later tasks need a module's return values.

Jinja filters can transform variables without changing their source. The `default` filter can supply a fallback, `bool` can normalise a Boolean-like value, and `to_nice_json` can produce readable diagnostic output. Filters should not compensate for an unclear variable model. Inventories written in INI can interpret inline host values differently from group `:vars` values, so YAML inventory or explicit type conversion reduces ambiguity.

Variables should distinguish configuration from secrets. A normal file can define the name of a database, while an encrypted variable supplies its password. A non-secret variable can document the expected encrypted name without exposing the protected value. Undefined-variable failures should be resolved by establishing a clear input contract, not by applying a default that could silently direct work to the wrong host or environment.
## Playbooks, plays, tasks, and modules
A playbook is a YAML automation blueprint containing one or more plays. Each play maps a host pattern to an ordered set of tasks. Each task normally invokes a module, which performs a defined operation such as managing a package, file, user, service, cloud resource, or network setting.

```yaml
- name: Configure web servers
  hosts: webservers
  become: true
  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start and enable Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

The fully qualified collection name, such as `ansible.builtin.service`, identifies the collection and avoids ambiguity. Short names often work for built-in content, but fully qualified names improve clarity and linking to documentation.

Modules are the primary units that perform work. Plugins extend Ansible's control-side behaviour in areas such as connections, callbacks, lookups, filters, inventory, caching, and execution strategy. Many POSIX modules use Python on the managed host, while Windows modules generally use PowerShell. Module requirements and supported platforms vary, so the documentation for each module remains authoritative.

Playbooks should favour purpose-built modules over shell commands. A module can validate arguments, report whether it changed a resource, and support idempotent behaviour. `ansible.builtin.command` should be used when no suitable module exists and shell processing is unnecessary. `ansible.builtin.shell` is appropriate only when the command needs shell features such as pipes, redirection, or expansion. Commands should define `changed_when`, `creates`, or `removes` where possible so results accurately represent change.

Idempotency means that repeated execution against the same starting conditions produces the same intended final state without unnecessary changes. It does not mean that Ansible resumes automatically at the failed task. After a failure, a corrected playbook normally runs from the beginning unless an operator deliberately uses options such as `--start-at-task`, tags, or step mode. Most state-oriented modules are idempotent, but not every module, command, or playbook is.

Playbooks run with:

```bash
ansible-playbook -i inventory/production.yml playbooks/site.yml
```

Useful options include:
- `--syntax-check` to parse the playbook without running it
- `--check` to predict changes where modules support check mode
- `--diff` to show supported before-and-after differences
- `--limit` to restrict hosts
- `--tags` and `--skip-tags` to select task labels
- `--step` to confirm each task interactively
- `--start-at-task` to begin at a named task
- `-v` through `-vvvv` to increase diagnostic output

Check mode is a simulation, not proof of a safe production outcome. Some modules only partly support it or do not support it. Staging tests and controlled rollouts remain necessary.
### Conditions, loops, blocks, and results
Conditions allow a task to run only when an expression evaluates to true. Facts, registered results, and input variables can drive the decision:

```yaml
- name: Install a Debian-family web package
  ansible.builtin.apt:
    name: nginx
    state: present
  when: ansible_facts['os_family'] == 'Debian'
```

A loop applies one task to several items:

```yaml
- name: Install required packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - curl
    - git
    - unzip
```

Modules that accept a list should receive the whole list in one call when possible because that approach is often faster and can resolve dependencies more consistently. Loops remain useful when each item needs distinct parameters or result handling.

Blocks group tasks so they can share `when`, `become`, or other directives. A block can also provide `rescue` tasks after failure and `always` tasks that run regardless of success. Rescue logic should restore a safe state or collect diagnostics. It should not convert an unsafe partial deployment into an apparent success.

Each module returns a result dictionary. Common fields include `changed`, `failed`, `msg`, and module-specific data. Registered results expose these fields to later tasks. `failed_when` can define domain-specific failure, and `changed_when` can correct a command's change reporting. These directives should reflect the actual state rather than hide warnings or failures.

Ansible normally uses a linear strategy. It runs a task across the hosts in the current batch before moving to the next task. The `forks` setting controls parallel worker processes, while `serial` divides hosts into batches for rolling change. `throttle` restricts concurrency for a particular task or block. A production rollout might set `serial` to a small number, verify service health after each batch, and stop if failures exceed an accepted threshold.

Task names form the operational narrative in console output and logs. Names should state the intended state, such as `Ensure Nginx is enabled and running`, rather than repeat the module name. Clear names also make `--start-at-task` and failure reports more useful.
## Privilege and connections
The remote user establishes the connection. The `become` system then uses an existing escalation mechanism, such as `sudo`, to execute a task as another user. `become: true` does not itself authenticate to the remote host, and `become_user` does not imply that escalation is enabled.

SSH key authentication supports non-interactive POSIX automation, but key handling still needs protection. Private keys should have passphrases where operationally practical, and an SSH agent or managed credential service can provide them during execution. Remote accounts should receive only the privileges their tasks require.

Connection variables can define the user, address, port, private key, connection plugin, Python interpreter, and escalation behaviour. Secrets should not appear in inventory, playbooks, shell history, logs, or repositories as plaintext.
## Roles, collections, templates, and handlers
Roles organise related tasks and assets under a recognised directory structure. They improve reuse when a playbook grows beyond a small, single-purpose file.

A role can contain:

```text
roles/webserver/
  defaults/main.yml
  files/
  handlers/main.yml
  meta/main.yml
  tasks/main.yml
  templates/
  vars/main.yml
```

Only the directories a role needs must exist. `tasks/main.yml` contains the main task list. `handlers/main.yml` defines handlers. `defaults/main.yml` supplies easily overridden defaults. `vars/main.yml` supplies high-precedence role variables. `files` contains static assets, and `templates` contains Jinja templates. `meta/main.yml` holds role metadata and can describe dependencies and argument specifications. It is not a general documentation substitute.

`ansible-galaxy role init webserver` scaffolds a standalone role. A play can apply roles through the `roles` keyword. `import_role` adds a role statically during playbook parsing, while `include_role` adds it dynamically during execution. `include_role` is a playbook task and cannot be invoked as an ad hoc module.

Collections package related Ansible content. A collection can include playbooks, roles, modules, and plugins, but it does not need to contain roles. Collections provide namespaces such as `community.general` and distribute content through Galaxy or another compatible server. Projects should record required collection and role versions in `requirements.yml` and review third-party content before use. Readable source lowers some inspection barriers, but it does not remove supply-chain risk.

Templates use Jinja to generate host-specific files from variables and facts. The template module renders content on the control node and transfers the result to the managed host. A template might set an application's bind address, port, environment name, or document root. Validation options should check generated configuration before it replaces a working file.

Handlers run operations in response to change. A task uses `notify` when it changes, and Ansible normally runs the notified handler once near the end of the relevant play section. Restarting a service after a configuration update is the standard example. A failed task can prevent pending handlers from running unless the play deliberately forces them, so automation should account for failure behaviour.
### Reusable web service example
A reusable web role can define overridable inputs in `defaults/main.yml`:

```yaml
web_package: nginx
web_service: nginx
web_port: 80
```

Its task file can install the package, render configuration, and start the service:

```yaml
- name: Ensure the web package is installed
  ansible.builtin.package:
    name: "{{ web_package }}"
    state: present

- name: Render the web configuration
  ansible.builtin.template:
    src: web.conf.j2
    dest: /etc/web/web.conf
    owner: root
    group: root
    mode: "0644"
    validate: /usr/sbin/webserver -t -c %s
  notify: Restart web service

- name: Ensure the web service is enabled and running
  ansible.builtin.service:
    name: "{{ web_service }}"
    state: started
    enabled: true
```

The handler can restart the service only after a valid configuration change:

```yaml
- name: Restart web service
  ansible.builtin.service:
    name: "{{ web_service }}"
    state: restarted
```

Validation protects the destination from a syntactically invalid rendered file when the module and application support it. The exact command and path depend on the application. File modes should be quoted so YAML and Ansible interpret them as intended.

A short playbook can apply the role:

```yaml
- name: Configure production web servers
  hosts: webservers
  become: true
  roles:
    - role: organisation.web.webserver
      web_port: 8080
```

The fully qualified role name shows that the role belongs to the `organisation.web` collection. The play overrides a low-precedence default without changing the role. This boundary lets staging and production use the same tested implementation with different inputs.

Roles should expose a small, documented interface. Defaults need descriptions, supported platforms need testing, and handlers need unique names or qualified references to avoid collisions. A role should not conceal unrelated operational workflows simply to reduce the number of playbook files.
## Security and secret handling
Source control should contain playbooks, roles, inventories where appropriate, dependency manifests, tests, and documentation. It should not contain plaintext passwords, tokens, private keys, vault passwords, or sensitive command output.

Ansible Vault encrypts individual variables or entire files. It protects data at rest, but Ansible must decrypt that data during execution. Vault does not automatically conceal a secret from task arguments, remote processes, debug output, or logs. Tasks that handle secrets should use `no_log: true` where appropriate, while recognising that `no_log` can also hide useful diagnostics.

Vault passwords require their own secure lifecycle. Interactive prompts work for manual runs. Automated systems can obtain vault passwords through protected files, executable password clients, or external secret managers. Vault IDs help distinguish passwords for environments or teams.

Example commands include:

```bash
ansible-vault create --vault-id production@prompt group_vars/production/vault.yml
ansible-vault edit --vault-id production@prompt group_vars/production/vault.yml
ansible-playbook --vault-id production@prompt playbooks/site.yml
```

Ansible Vault is distinct from HashiCorp Vault. Integrating HashiCorp Vault requires an appropriate lookup plugin or module from a collection, plus secure authentication to that service.

Security also depends on dependency review, least privilege, protected control nodes, verified host keys, restricted repository access, safe temporary files, log handling, and tested recovery. Community roles and collections should be pinned, inspected, and updated through a controlled process.
## Troubleshooting and operational practice
Troubleshooting should move from environment and scope to syntax, connectivity, task input, and remote state.

1. Run `ansible --version` to confirm the executable, Python version, module paths, and configuration file.
2. Run `ansible-config dump --only-changed` to identify active overrides.
3. Run `ansible-inventory -i <source> --graph` and `--host <name>` to inspect parsed hosts and variables.
4. Test a narrow host pattern with `ansible.builtin.ping`.
5. Run `ansible-playbook --syntax-check` before execution.
6. Use `--check --diff` where supported, first against a staging host or a restricted `--limit` target.
7. Increase verbosity gradually. `-vvvv` can expose connection details and sensitive data, so diagnostic output needs careful handling.
8. Read the failed module's structured result, including `msg`, `stdout`, `stderr`, and return code where present.
9. Use `ansible-doc` to confirm parameters, requirements, return values, check-mode support, and examples.
10. Add temporary debug tasks only for non-sensitive values, or protect output with `no_log`.

`No route to host` indicates a network routing, firewall, address, or availability problem rather than an inventory syntax error by itself. `Permission denied` points towards authentication, account, key, or authorisation. An undefined variable often reflects naming, scope, load order, or precedence. A task that always reports `changed` may need a more suitable module or explicit change conditions.

Removing an unreachable host from inventory hides the symptom but does not diagnose the cause. A deliberate inventory change is reasonable when the host should no longer be managed. Otherwise, network and connection evidence should drive the repair.

Reliable Ansible projects use version control, peer review, meaningful task names, consistent formatting, and automated validation. They separate production, staging, and development inventories while sharing reusable roles. They pin dependencies, test upgrades, lint YAML and Ansible content, and roll changes out to small host batches before broad deployment.

A practical delivery path moves the same revision through validation and environments:
1. format and lint YAML, playbooks, roles, and collections
2. run syntax checks and automated role or collection tests
3. execute check mode where it provides useful coverage
4. deploy to an isolated test target
5. deploy to staging with production-like variables and dependencies
6. deploy to a limited production batch
7. verify application health and configuration state
8. expand the rollout and retain the execution record

Rollback requires explicit design. Version control can restore earlier automation, but an earlier playbook does not always reverse a database migration, package upgrade, or external API operation. Risky tasks need backups, compatibility checks, restore procedures, and clear stop conditions. Idempotency supports safe repetition of the defined state, not automatic reversal of every change.

Tasks should remain focused and observable. Comments should explain intent or constraints rather than restate syntax. Variables should expose real configuration choices. Roles should have a clear purpose and documented inputs. Playbooks should use handlers for change-driven operations, explicit privilege escalation, and modules that accurately report state.

Centralised automation does not replace these practices. AWX and Red Hat Ansible Automation Platform add governance and operational controls, but sound inventories, secure credentials, deterministic dependencies, readable automation, and tested recovery still determine reliability.