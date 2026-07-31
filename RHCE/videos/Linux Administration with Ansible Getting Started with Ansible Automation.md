# Linux Administration with Ansible: Getting Started with Ansible Automation
> [!NOTE]
> Introduction on Ansible automation through a repeatable multi-platform lab, explaining how controllers, inventories, variables, modules, SSH, privilege escalation, and playbooks enable consistent, idempotent Linux administration.

Ansible automates Linux configuration from a control node. It connects to managed nodes, usually through OpenSSH, runs modules that inspect or change state, and returns structured results. Managed nodes do not run an Ansible agent, and they do not need the Ansible package. Most POSIX modules need a usable Python interpreter and an account with an interactive shell.

Red Hat Enterprise Linux 10 provides Python 3.12 as its default Python implementation and `ansible-core` 2.16 as an Application Stream. The core package supplies the command-line tools, execution engine, and `ansible.builtin` collection. Additional collections provide modules, plugins, and roles for particular platforms or products.

Red Hat supports the RHEL 10 `ansible-core` package for managing RHEL 9 and RHEL 10 nodes within the documented RHEL System Roles scope. Organisations that automate a broader estate should use content and execution environments validated for every target. Red Hat Ansible Automation Platform adds supported execution environments, centralised credentials, job control, reporting, and automation content from private automation hub.
## Why Linux estates need automation
Manual administration can work for a small number of hosts, but it scales poorly. An administrator must connect to each host, remember every command, preserve the required order, interpret failures, and record the result. Small differences accumulate as hosts age. A later administrator may not know which change created a particular state.

Shell and Python scripts improve consistency, but a general-purpose script must implement many functions itself:
- Detect the operating system and release.
- Select the correct package manager, package name, service name, and file path.
- Check existing state before making a change.
- Handle partial failure and produce useful output.
- Authenticate securely and elevate privileges.
- Run across many hosts without losing control of concurrency.

Distribution differences remain important. RHEL uses DNF, while Debian-derived systems use APT. A package may also have different names across distributions. The generic `ansible.builtin.package` module selects an available package-manager backend, but it does not translate package names. Inventory variables, facts, conditionals, or separate platform roles must supply those differences.

Ansible modules encapsulate common system operations. A package task can state that `httpd` must be present, and a service task can state that `httpd` must be enabled and running. Most state-oriented modules first inspect the target and report `ok` when it already has the requested state. They report `changed` only when they perform work.

This behaviour supports idempotence, but Ansible does not guarantee that every module, task, or playbook is idempotent. Commands with side effects, shell pipelines, unconstrained upgrades, poorly designed custom modules, and external APIs can produce a new result on each run. Administrators must test repeated runs and inspect module documentation.

Ansible separates reusable automation into several elements:

| Element | Purpose |
|---|---|
| Control node | Runs Ansible commands and playbooks |
| Managed node | Receives configuration through a connection plugin |
| Inventory | Defines hosts, groups, connection data, and inventory variables |
| Module | Performs one operation, such as managing a package, file, user, or service |
| Plugin | Extends connections, inventory, callbacks, variable sources, caching, and other engine functions |
| Play | Maps an ordered set of tasks to a host pattern |
| Playbook | Stores one or more plays in YAML |
| Collection | Packages modules, plugins, roles, documentation, and related content |
| Role | Organises reusable tasks, handlers, variables, files, and templates |

Ad hoc commands suit one-off checks and controlled interventions. Playbooks suit reviewed, repeatable configuration. Roles and collections package mature automation for reuse.
### How Ansible executes work
Ansible processes automation on the control node before it changes a managed node. It selects a configuration file, loads inventory sources, expands the host pattern, resolves variables, and chooses the requested module and connection plugin. An operator can inspect several of these decisions before execution with `ansible --version`, `ansible-config`, `ansible-inventory`, and `--list-hosts`.

For a normal POSIX task, the control node connects through SSH as the remote user. An action plugin may perform some preparation locally. Ansible then sends a temporary module payload and its arguments, invokes the managed node's Python interpreter, receives a structured result, and removes temporary data. Pipelining can reduce file transfers when the connection and privilege configuration support it.

The result normally includes the host, success or failure state, whether the task changed the host, and module-specific data. `unreachable` identifies a connection failure before useful module work. `failed` means that Ansible reached the host but the module could not complete successfully. A playbook removes a failed host from later tasks in that play unless error-handling logic changes the behaviour.

Not every operation follows the same path. The `raw` module sends a command directly through the connection and can bootstrap a host without Python. Network modules often run code on the control node and communicate through a device API or command channel. Local actions and delegation can run a task on another host while using variables from the original target.

Privilege escalation occurs after connection. Ansible can log in as an unprivileged account and invoke `sudo` only for tasks that declare `become: true`. This separation supports better SSH policy and clearer audit records than direct root login.

Ansible can work on several hosts concurrently. Higher concurrency can shorten a run, but it can also overload package repositories, authentication services, APIs, or clustered applications. The `forks` setting controls control-node concurrency, while playbook controls such as `serial` limit the rollout batch. A new project should start with restrained values and measure the effect.

The playbook remains on the control node. Managed nodes receive the module code and data needed for each operation, not a durable copy of the complete playbook. This architecture reduces installed components on managed nodes, but it makes control-node security, project integrity, inventory accuracy, and credential protection critical.
## Build a RHEL 10 practice environment
A disposable lab should separate the control node from its managed nodes. KVM with libvirt, an approved cloud account, or another supported virtualisation platform can provide the hosts. Snapshots or rebuildable images allow safe recovery after a failed exercise.

| Host | Operating system | Function |
|---|---|---|
| `control.example.com` | RHEL 10 | Runs `ansible-core` and stores the project |
| `app01.example.com` | RHEL 10 | Tests package, service, file, and user tasks |
| `db01.example.com` | RHEL 10 | Tests groups, host variables, and multi-host execution |
| `legacy01.example.com` | RHEL 9, optional | Tests the supported cross-major-version scope |

Each host needs stable addressing and name resolution. DNS provides the cleanest solution. A small lab can use controlled `/etc/hosts` entries. The control node must resolve every inventory hostname unless the inventory assigns an `ansible_host` address.

The lab network should isolate experimental hosts from production systems. An administrator should confirm the target pattern with `--list-hosts` before a broad change. Snapshots do not replace backups for any host that holds valuable data.
### Register RHEL 10
An organisation should register each RHEL host through its approved Customer Portal, Satellite, cloud, or image workflow. Interactive registration avoids exposing a portal password in shell history or a process list:

```console
$ sudo subscription-manager register
$ sudo subscription-manager identity
$ sudo dnf repolist
```

Activation keys and organisation identifiers suit automated registration when the organisation manages them securely. A command line should never contain a personal portal password. Simple Content Access also means that older expectations about an attached subscription and a `Current` status do not describe every modern RHEL environment.
### Install Ansible on the control node
The RHEL System Roles package installs its supported collection and pulls in `ansible-core` as a dependency:

```console
$ sudo dnf install rhel-system-roles
$ ansible --version
$ ansible-galaxy collection list
```

An installation that needs only the core engine can install `ansible-core` directly:

```console
$ sudo dnf install ansible-core
```

The `ansible --version` output identifies the core version, active configuration file, module search paths, collection paths, executable location, and Python runtime. RHEL 10 should report the packaged core 2.16 stream and Python 3.12 unless the administrator deliberately uses another supported environment.

The `ansible-core` package differs from the community `ansible` package. The community package combines `ansible-core` with a changing set of community collections. A RHEL administrator should not replace the supported system package by installing an unrelated PyPI release into the system Python. A project that requires community content can use an isolated environment, `pipx`, or an Ansible Automation Platform execution environment, subject to its support and governance requirements.

Only the control node needs Ansible. A normal RHEL 10 managed node already includes Python 3.12 in most installations. A minimal image may require `python3`. The `ansible.builtin.raw` module can bootstrap Python because it does not require Python on the managed node, but an image-building or provisioning workflow usually provides a cleaner solution.
## Establish secure managed-node access
Ansible normally uses the control node's OpenSSH client. A managed node needs an SSH service, a trusted host key, a login account, and enough privilege for the intended tasks. Direct root login creates avoidable risk.

A dedicated automation identity improves auditability. An identity service can manage it centrally, or an administrator can provision a local `ansible` account during image creation:

```console
# useradd --create-home ansible
# passwd ansible
```

The control-node operator creates an Ed25519 key pair and protects the private key with a passphrase:

```console
$ ssh-keygen -t ed25519 -a 64 -f ~/.ssh/id_ansible_rhel10
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_ansible_rhel10
```

The operator verifies the managed host's SSH fingerprint through a trusted channel before accepting it. A bootstrap administrator then installs only the public key:

```console
$ ssh-copy-id -i ~/.ssh/id_ansible_rhel10.pub ansible@app01.example.com
$ ssh ansible@app01.example.com
```

The private key stays on the control node or in an approved credential store. Copying a managed node's private key to the control node, sharing one unprotected private key among users, or disabling host-key checking weakens the trust model.

Privilege escalation uses `become`, with `sudo` as the usual RHEL method. The managed account needs a reviewed sudo policy. A broad `NOPASSWD: ALL` rule gives anyone who obtains the key unrestricted root access. A stronger starting point prompts for the sudo password with `--ask-become-pass`, or `-K`. Ansible Automation Platform can retrieve the credential from its protected store.

Where unattended work requires passwordless sudo, the organisation should limit the account, source systems, allowed commands, and execution path as far as the automation permits. Logs, key rotation, account expiry, and incident revocation must form part of the design.

Ansible Vault can encrypt variables such as passwords and tokens at rest. Vault does not remove access control requirements, and a vault password should not sit beside the encrypted file. Production environments should use an approved secret manager or Automation Platform credential.
### Diagnose the first connection
A failed ping test should lead to a layered diagnosis. The operator first confirms that inventory selected the intended host and address:

```console
$ ansible-inventory -i inventory/hosts.yml --host app01.example.com
$ getent hosts app01.example.com
```

OpenSSH should then succeed independently of Ansible:

```console
$ ssh -i ~/.ssh/id_ansible_rhel10 ansible@app01.example.com
```

An SSH failure usually points to name resolution, routing, firewall policy, host-key verification, file permissions, account state, or public-key installation. The private key should normally allow read and write access only to its owner. The managed account needs an allowed shell and an unlocked authentication path.

After SSH works, the managed host needs a supported Python interpreter for most modules:

```console
$ ssh ansible@app01.example.com 'python3 --version'
```

Ansible interpreter discovery normally selects the RHEL 10 system Python correctly. A custom image with several interpreters can set `ansible_python_interpreter` in inventory, but the project should not override discovery without a specific need.

The next check isolates privilege escalation:

```console
$ ansible app01.example.com -m ansible.builtin.command -a 'id'
$ ansible app01.example.com -b -K -m ansible.builtin.command -a 'id'
```

The first command should report the automation account. The second should report root when the sudo policy and password are correct. A failure at this stage belongs to sudo policy or credential handling, not basic SSH.

Verbose output can expose the selected inventory, connection, interpreter, and escalation path:

```console
$ ansible app01.example.com -m ansible.builtin.ping -vvvv
```

Verbose logs can contain hostnames, paths, command arguments, and other sensitive operational data. The operator should collect the smallest useful trace, protect it, and remove it according to organisational policy.
## Configure Ansible
Ansible can receive settings from configuration files, environment variables, command-line options, playbook keywords, and variables. Configuration-file discovery follows this order:

| Priority | Candidate |
|---|---|
| 1 | File named by `ANSIBLE_CONFIG` |
| 2 | `ansible.cfg` in the current working directory |
| 3 | `~/.ansible.cfg` |
| 4 | `/etc/ansible/ansible.cfg` |

Ansible loads the first configuration file it finds and ignores the remaining candidates. The path `/etc/ansible/ansible.cfg` is a final candidate, not a guarantee that a populated file exists. A project-local `ansible.cfg` keeps settings with the automation that needs them and prevents one project's defaults from affecting another.

Ansible refuses to load `ansible.cfg` automatically from a world-writable current directory. Loading configuration from such a directory could execute attacker-controlled plugins or commands, including with elevated privileges. The administrator should correct ownership, permissions, or mount options instead of bypassing the protection.

The `ansible-config` command can generate a documented starting file:

```console
$ mkdir -p ~/automation/rhel10-baseline
$ cd ~/automation/rhel10-baseline
$ ansible-config init --disabled > ansible.cfg
```

A concise project configuration can contain only deliberate changes:

```ini
[defaults]
inventory = ./inventory
remote_user = ansible
private_key_file = ~/.ssh/id_ansible_rhel10
host_key_checking = True

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = True
```

The operator can inspect the effective configuration with these commands:

```console
$ ansible --version
$ ansible-config view
$ ansible-config dump --only-changed
$ ansible-config list
```

`ansible-config view` prints the active file. `dump --only-changed` shows effective values that differ from defaults and identifies their source. `list` documents every available setting. A spelling error or a setting under the wrong INI section will not produce the intended effective value, so `dump --only-changed` provides an important check.

The shell's `readonly` attribute does not turn `ANSIBLE_CONFIG` into an administrative security control. A user who controls a process can start another shell, change its environment, or invoke Ansible through another execution path. Enforcement requires operating-system permissions, a controlled service account, a reviewed pipeline, or an Automation Platform execution environment and job template.

Command-line options and variables may override configuration values under Ansible's broader precedence rules. A project should avoid defining the same behaviour in several places because a technically valid override can still surprise an operator.
### Keep each project self-contained
A project directory defines an execution boundary. Its configuration, inventory, playbooks, templates, and declared dependencies should describe one coherent automation purpose. A practical RHEL 10 baseline project can use these top-level items:

| Item | Function |
|---|---|
| `ansible.cfg` | Project defaults and plugin settings |
| `inventory/` | Static sources, plugin configurations, and inventory variables |
| `site.yml` | Main playbook entry point |
| `playbooks/` | Focused operational playbooks |
| `roles/` | Project-owned reusable roles |
| `templates/` | Jinja templates deployed by tasks |
| `files/` | Static files deployed without rendering |
| `requirements.yml` | Approved external collection and role dependencies |
| `README.md` | Scope, prerequisites, validation, execution, and recovery guidance |

Relative paths generally resolve from the active configuration file, which helps the project behave consistently from different shells. A wrapper script that silently changes directories or injects variables can obscure that behaviour, so the execution entry point should remain clear.

The repository should pin external content to reviewed versions. Installing an unpinned collection at run time can change module behaviour without a playbook revision. An execution environment can freeze the core engine, Python dependencies, system libraries, and collections as one tested image.

Separate inventories or inventory plugin configurations can represent development, test, and production. Shared variables should not erase meaningful environmental differences. The operator should select the intended source explicitly and confirm the resulting hosts. A production inventory name alone does not prove that its addresses or groups are correct.

Project documentation should identify the supported RHEL releases, required collections, authentication model, privilege assumptions, safe target patterns, expected check-mode limits, and recovery approach. These operational details allow another authorised administrator to run the automation without reconstructing hidden local knowledge.
## Design inventories and host patterns
Inventory gives Ansible an operational model of managed infrastructure. Static inventory works for a small, stable lab. Inventory plugins suit cloud platforms, virtualisation systems, identity directories, configuration databases, and other authoritative sources whose hosts change over time.

An inventory can use YAML or INI. YAML expresses structured values and data types more clearly. INI remains compact, but its parser can interpret values differently depending on where they appear. A project should use one well-understood format and validate its result.

This YAML inventory defines two RHEL 10 hosts and functional groups:

```yaml
all:
  children:
    rhel10:
      hosts:
        app01.example.com:
          ansible_host: 192.0.2.11
        db01.example.com:
          ansible_host: 192.0.2.12
      vars:
        ansible_user: ansible
    web:
      hosts:
        app01.example.com:
    database:
      hosts:
        db01.example.com:
```

Every explicit host belongs to the built-in `all` group. A host that belongs to no explicit group other than `all` also belongs to `ungrouped`. A host may belong to several functional, location, lifecycle, or platform groups.

Parent groups can contain child groups. Variables inherited from a parent apply to hosts in its children, while a more specific variable can override the parent value. A simple hierarchy reduces ambiguity. Group names should use stable identifiers that also work safely in variable and pattern contexts.

The implicit localhost exists when Ansible needs to address `localhost`, `127.0.0.1`, or `::1` without a matching inventory host. It uses a local connection and the control node's Python interpreter. It does not automatically become an ordinary member of inventory groups. Automation that relies on localhost should define it explicitly when group membership or variables need to remain visible.

A local connection belongs in inventory or on the command line:

```yaml
all:
  hosts:
    controller:
      ansible_connection: local
```

```console
$ ansible localhost -c local -m ansible.builtin.ping
```

Setting a lowercase shell variable such as `ansible_connection=local` before the command does not define an inventory variable. A successful test against the implicit localhost can hide that error because the implicit host already selects the local connection.
### Keep variables outside the host list
The default `host_group_vars` plugin loads YAML variable files from `group_vars` and `host_vars`. These files keep inventory structure readable and support lists and mappings.

| Path | Scope |
|---|---|
| `inventory/hosts.yml` | Hosts and groups |
| `inventory/group_vars/all.yml` | Variables for every inventory host |
| `inventory/group_vars/rhel10.yml` | Variables inherited by the `rhel10` group |
| `inventory/host_vars/app01.example.com.yml` | Overrides for one host |

A mixed-distribution project can hide package-name differences behind a common variable. The RHEL group can define `editor_package: vim-enhanced`, while a Debian group can define `editor_package: vim`. A task then passes `editor_package` to `ansible.builtin.package`. The generic module selects DNF or APT, and the project supplies the distribution-specific name.

Connection variables such as `ansible_host`, `ansible_user`, `ansible_port`, and `ansible_connection` also fit inventory. Secrets do not. Plain-text passwords, private keys, and tokens should stay in a protected credential system or encrypted vault data.
### Inspect inventory before execution
Inventory inspection reveals parsing mistakes, variable inheritance, and unintended targets:

```console
$ ansible-inventory -i inventory/hosts.yml --graph
$ ansible-inventory -i inventory/hosts.yml --list
$ ansible-inventory -i inventory/hosts.yml --host app01.example.com
$ ansible 'web' -i inventory/hosts.yml --list-hosts
```

Host patterns select groups and set operations. `web` selects a group, `web:&rhel10` selects the intersection, and `all:!database` excludes database hosts. Quoting the pattern prevents the shell from interpreting special characters.

An operator should inspect the expanded host list before a disruptive command. Inventory aliases can obscure the network address, and nested groups can broaden a target beyond its apparent name.
### Use inventory plugins for changing infrastructure
A dynamic inventory plugin queries an authoritative service and converts its records into Ansible hosts, groups, and variables. The project can combine static and dynamic sources in an inventory directory. Automation Platform can synchronise external inventory and retain job history.

An ad hoc Nmap scan piped through AWK does not provide sound dynamic inventory. It finds listening ports at one instant, cannot identify ownership or intended management scope, omits hosts hidden by firewalls, and may include systems outside the change authority. Unauthorised scanning also breaches many organisational policies. A port scan can support an authorised diagnostic exercise, but a cloud API, CMDB, Satellite, or virtualisation inventory plugin should define managed assets.

Ansible recommends inventory plugins over legacy executable inventory scripts. A project should pin the required collection, protect source credentials, test group construction, and cache results only for a justified interval.
## Run ad hoc commands
The `ansible` command runs one module against hosts selected by a pattern:

```text
ansible <pattern> -i <inventory> -m <collection.module> -a '<arguments>'
```

Fully qualified collection names identify the intended module and link directly to its documentation. Short names often work for built-in modules, but a collection can introduce a conflicting name.

The ping module tests the complete Ansible path:

```console
$ ansible all -m ansible.builtin.ping
```

`ansible.builtin.ping` is not an ICMP echo request. It authenticates through the configured connection, finds a usable Python interpreter, executes a small module, and returns `pong`. A successful result therefore proves more than basic network reachability. Windows and network devices use different modules.

The setup module gathers facts:

```console
$ ansible rhel10 -m ansible.builtin.setup
$ ansible rhel10 -m ansible.builtin.setup -a 'filter=ansible_distribution*'
```

Facts include distribution, release, architecture, interfaces, addresses, memory, processors, mounts, and Python details. Fact output can expose operational information, so logs and support bundles need suitable access controls.

Package installation requires privilege escalation:

```console
$ ansible web --become --ask-become-pass \
  -m ansible.builtin.dnf \
  -a 'name=httpd state=present'
```

The long options show intent clearly. The short forms `-b` and `-K` provide the same functions. `state=present` installs the available package when absent and leaves an installed package unchanged. `state=latest` can introduce an unreviewed update on every run, so controlled environments should use approved versions or a defined update process.

The generic package module supports heterogeneous groups:

```console
$ ansible all --become --ask-become-pass \
  -m ansible.builtin.package \
  -a 'name=tree state=present'
```

This command works only where every distribution uses the same package name and supports the common arguments. A variable should provide differing names.

Other useful ad hoc checks include:

```console
$ ansible all -m ansible.builtin.command -a 'uptime'
$ ansible web -m ansible.builtin.service_facts
$ ansible app01.example.com -m ansible.builtin.stat -a 'path=/etc/httpd/conf/httpd.conf'
$ ansible all -m ansible.builtin.user -a 'name=ansible state=present' --become -K
```

The `command` module does not process pipes, redirection, globbing, or other shell syntax. `ansible.builtin.shell` invokes a shell when such syntax is genuinely required, but dedicated modules usually provide safer state checks and clearer change reporting.

`ansible-doc` provides local module documentation:

```console
$ ansible-doc ansible.builtin.dnf
$ ansible-doc ansible.builtin.user
$ ansible-doc -l
```

Documentation identifies arguments, return values, requirements, check-mode support, and examples. The number of installed modules depends on installed collections, so a fixed module count has no lasting value.

Ansible reports each targeted host independently. A green `ok` result means that the module completed without changing the declared state. A yellow `changed` result means that the module reports a modification. A red `failed` result identifies module failure, while `unreachable` identifies a connection failure. Colour supports quick reading but scripts should use exit status and structured callback output, not terminal colour.

A successful module result confirms only the operation that the module checked. Installing a package does not prove that its service can start, accept traffic, or serve correct content. Mature automation adds explicit validation, such as a service state check, an HTTP request, a configuration test, or a port check. Validation should observe the required outcome without introducing a new change. A failure should identify enough context for diagnosis while keeping credentials and sensitive data out of output.

Ad hoc commands suit diagnosis, bootstrap, and rare operations. A command that the organisation expects to repeat should become a playbook. The playbook then preserves ordering, review history, variables, privilege settings, and verification.
## Create playbooks
A playbook is YAML data that maps hosts to ordered tasks. It is not a shell script. Each task invokes a module with arguments, and Ansible executes tasks in order across the hosts selected by the play. YAML indentation defines structure, so spaces must remain consistent and tabs must not provide indentation.

This playbook configures a simple RHEL 10 web service:

```yaml
---
- name: Configure RHEL 10 web hosts
  hosts: web
  become: true
  gather_facts: true

  vars:
    web_packages:
      - httpd
      - firewalld

  tasks:
    - name: Install web packages
      ansible.builtin.dnf:
        name: "{{ web_packages }}"
        state: present

    - name: Enable and start httpd
      ansible.builtin.service:
        name: httpd
        enabled: true
        state: started

    - name: Install a managed login notice
      ansible.builtin.copy:
        content: "Managed by Ansible\n"
        dest: /etc/motd
        owner: root
        group: root
        mode: "0644"
```

The play targets the `web` group, enables privilege escalation, gathers facts, and runs three named tasks. Passing a package list in one DNF transaction is more efficient than looping over individual packages.

The operator validates the file before applying it:

```console
$ ansible-playbook web.yml --syntax-check
$ ansible-playbook web.yml --list-hosts
$ ansible-playbook web.yml --check --diff
$ ansible-playbook web.yml
```

Syntax checking catches YAML and Ansible syntax errors, but it cannot prove that the change is safe. `--list-hosts` confirms the target expansion. Check mode asks modules to report what they would change, and diff mode displays supported content differences. Some modules cannot simulate an operation accurately, and a later task may depend on a change that check mode did not perform. A staged run on disposable or non-production hosts remains necessary.
### Facts, variables, and Jinja
Ansible gathers facts at the start of each play by default through `ansible.builtin.setup`. Current playbooks should prefer the `ansible_facts` mapping over older injected variable names:

```yaml
- name: Display the managed operating system
  ansible.builtin.debug:
    msg: >-
      {{ ansible_facts['distribution'] }}
      {{ ansible_facts['distribution_major_version'] }}
```

Jinja expressions use double braces. YAML values that begin with a Jinja expression need quotes. A folded scalar such as `>-` can improve readability for a longer expression while producing one output line.

Facts can drive controlled platform differences:

```yaml
- name: Install the RHEL editor package
  ansible.builtin.dnf:
    name: vim-enhanced
    state: present
  when:
    - ansible_facts['distribution'] == 'RedHat'
    - ansible_facts['distribution_major_version'] == '10'
```

Inventory groups often express an intended platform more clearly than runtime detection. Facts remain valuable for observed properties such as architecture, release, interfaces, and memory. A project should choose the source that represents its intent and avoid complex conditionals spread across many tasks.

When a play does not use facts, `gather_facts: false` reduces connection work. The generic package module may still gather the package-manager fact when it needs to select a backend.

Variables can come from inventory, playbooks, roles, included files, command-line extra variables, facts, and other sources. Their precedence can be complex. A variable should have one natural owner, and sensitive values should remain encrypted or external.
### Use idempotent tasks
State-oriented tasks should describe the required result:
- `state: present` ensures that a package, user, or file exists.
- `state: absent` ensures that an object does not exist.
- `enabled: true` ensures that a service starts during boot.
- `state: started` ensures that a service currently runs.

Repeated execution should produce `ok` after the required state exists. An operator should investigate a task that reports `changed` on every run without a legitimate changing input.

The `command` and `shell` modules need explicit guards when a dedicated module cannot replace them. Parameters such as `creates` or `removes` can prevent repeated execution for simple cases. A command that edits a file with `sed` usually belongs in `lineinfile`, `blockinfile`, `copy`, or `template`.

Templates use Jinja to render host-specific files from reviewed source. A handler can restart a service only when a task notifies it after a configuration change. This pattern avoids unnecessary service disruption and connects the action to the change that requires it.
### Reuse RHEL System Roles
RHEL System Roles provide supported automation for common RHEL services and settings. The `rhel-system-roles` RPM installs the `redhat.rhel_system_roles` collection in a system collection path. Available roles cover areas such as time synchronisation, networking, storage, firewall configuration, SELinux, SSH, sudo, logging, kernel settings, certificates, and systemd.

A role call uses its fully qualified name:

```yaml
---
- name: Configure time synchronisation
  hosts: rhel10
  become: true

  vars:
    timesync_ntp_servers:
      - hostname: time.example.com
        iburst: true

  roles:
    - role: redhat.rhel_system_roles.timesync
```

Role variables form a public interface. The administrator should use the variable names documented for the installed RHEL 10 role version. Examples under `/usr/share/doc/rhel-system-roles/` and the Red Hat documentation match the packaged content more closely than an arbitrary internet example.

Collections also prevent a project from treating every module as part of one global namespace. `ansible.builtin.dnf` belongs to core, while another vendor's module may require an additional collection. A `requirements.yml` file can declare and pin approved non-system collections for an isolated execution environment.
## Operate automation responsibly
An automation project should keep `ansible.cfg`, inventory structure, playbooks, roles, templates, and collection requirements in version control. Peer review can then examine the target pattern, privilege use, module choice, handlers, variable changes, and rollback plan.

Secrets, generated private keys, vault passwords, retry files, caches, and temporary output do not belong in the repository. File permissions and repository access should match the sensitivity of inventory and configuration data.

A controlled change path includes:
- Validate YAML and Ansible syntax.
- Inspect inventory and expanded host patterns.
- Run static analysis where available.
- Test repeated execution in a disposable environment.
- Use check and diff modes where modules support them.
- Apply the change to a small canary group.
- Review results before wider rollout.
- Record the playbook revision, inventory source, operator, and outcome.

Automation accelerates both correct and incorrect operations. Broad patterns such as `all`, elevated privileges, and high concurrency deserve extra care. Production playbooks can use `serial` to limit each batch and preserve service capacity during a rollout.
## Command reference
| Goal | Command |
|---|---|
| Show the installed core and active configuration | `ansible --version` |
| Show changed configuration values | `ansible-config dump --only-changed` |
| Generate a sample configuration | `ansible-config init --disabled > ansible.cfg` |
| Display the inventory hierarchy | `ansible-inventory -i inventory/hosts.yml --graph` |
| Inspect one host and its variables | `ansible-inventory -i inventory/hosts.yml --host app01.example.com` |
| Preview a pattern | `ansible 'web:&rhel10' --list-hosts` |
| Test login and Python execution | `ansible all -m ansible.builtin.ping` |
| Gather facts | `ansible all -m ansible.builtin.setup` |
| Read module documentation | `ansible-doc ansible.builtin.dnf` |
| Check playbook syntax | `ansible-playbook site.yml --syntax-check` |
| Preview supported changes | `ansible-playbook site.yml --check --diff` |
| Run a playbook and request the sudo password | `ansible-playbook site.yml --ask-become-pass` |
