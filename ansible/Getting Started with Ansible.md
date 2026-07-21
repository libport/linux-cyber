# Getting Started with Ansible
Ansible automates system configuration, software deployment, and infrastructure operations. It runs from a control node and applies a declared configuration to managed nodes. The control node does not need a persistent Ansible service, database, or agent. Managed nodes usually do not need Ansible installed because the controller transfers or invokes the required automation through connection plugins such as SSH, PowerShell remoting, local execution, or a platform API.

Ansible works best when automation describes the required end state. A task can declare that a directory must exist, a package must be installed, a setting must have a particular value, or a service must be running. A module examines the current state and changes it only when necessary. This approach reduces the conditional scripting otherwise required to handle every possible starting state.

The principal concepts form a simple progression:
- Modules perform focused operations.
- Tasks call modules with specific arguments.
- Plays apply ordered tasks to selected hosts.
- Playbooks store one or more plays in YAML.
- Inventory identifies managed hosts, groups, variables, and connection details.
- Collections distribute modules, plugins, roles, and other reusable content.
- Facts describe managed systems and support conditional configuration.

The basic workflow combines these concepts. The controller loads configuration and inventory, selects hosts through a pattern, connects to each target, gathers facts when requested, and runs the tasks in a play. Ansible reports whether each task succeeded, failed, changed the target, or was skipped.
## Installation and environment
The controller requires a supported Python version on a UNIX-like system. Linux, macOS, BSD, and Windows Subsystem for Linux can act as control nodes. Native Windows does not serve as a supported control node. Python support changes between `ansible-core` releases, so the current support matrix should determine the interpreter version rather than a fixed version copied from an older example.

Most POSIX managed nodes also require Python and an account that Ansible can use through an appropriate transport. Exceptions exist. Network modules can operate without Python on the device, and some low-level actions can bootstrap a host before Python becomes available. Each module documents its own requirements.

Two related Python packages serve different needs:
- `ansible-core` supplies the execution engine, command-line tools, and the `ansible.builtin` collection.
- `ansible` includes `ansible-core` plus a community-curated set of collections for many platforms and services.

The full package suits broad exploration. The core package suits controlled environments that install only required collections. These packages follow separate version schemes. `ansible --version` reports the associated `ansible-core` version, while `ansible-community --version` reports the full community package version.

An isolated installation avoids conflicts with operating-system Python packages. Current documentation supports `pipx` and `pip`. A `pipx` installation keeps Ansible in its own environment and exposes its commands on the shell path:

```text
pipx install --include-deps ansible
ansible --version
```

A user-level `pip` installation remains available when it fits the environment:

```text
python3 -m pip install --user ansible
ansible --version
```

The interpreter-qualified form, `python3 -m pip`, prevents a common error in systems with several Python installations. It ensures that the selected interpreter runs the matching `pip`. If the installed commands do not appear on `PATH`, the shell must include the user scripts directory. The shell profile can preserve that path across sessions.

Command completion can reduce typing and expose available options. The optional `argcomplete` package supports Bash and offers limited support for Z shell and Tcsh. A `pipx` installation can inject it into the Ansible environment:

```text
pipx inject --include-apps ansible argcomplete
activate-global-python-argcomplete --user
```
## Modules, tasks, and desired state
A module is a focused unit of automation. `ansible.builtin.file` manages paths and attributes, `ansible.builtin.copy` transfers or creates files, `ansible.builtin.apt` manages packages on Debian-family systems, and `ansible.builtin.timezone` configures time zones. The fully qualified collection name, or FQCN, identifies the exact content and avoids ambiguity.

An ad hoc command runs one module without creating a playbook. The following command tests local module execution:

```text
ansible localhost -m ansible.builtin.ping
```

The ping module does not send an ICMP echo request. It verifies that Ansible can use the selected connection and execute suitable module code on the target. A successful `pong` response confirms that basic path.

The file module can ensure that a directory exists:

```text
ansible localhost -m ansible.builtin.file \
  -a "path=test state=directory"
```

The first run creates the directory and reports a change. Later runs report success without a change while the directory already satisfies the declaration. If another process removes it, the next run creates it again. Changing the state to `absent` removes the path when present and makes no change when it is already absent.

The `touch` state needs careful interpretation. It creates a missing file, but it normally updates access and modification times on an existing file. It can therefore report a change on repeated runs unless the time settings preserve existing values. A directory declaration provides a clearer first demonstration of idempotent behaviour.

The copy module can place literal content in a file:

```text
ansible localhost -m ansible.builtin.copy \
  -a "dest=hello content='hello world'"
```

If the destination is missing, the module creates it. If its content differs, the module replaces it. If it already contains the declared content and attributes, the module leaves it unchanged. The module owns this file state, so it can handle the relevant alternatives without a script that separately tests for existence and compares content.

Not every desired state fits one module. If a file occupies a path that must become a directory, `ansible.builtin.file` reports the conflict instead of guessing whether to delete valuable data. A playbook must express the intended sequence, such as backing up or removing the file before creating the directory. Modules deliberately limit their responsibilities.

Module documentation defines parameters, return values, platform requirements, examples, and support for check or diff mode. Local documentation remains available through `ansible-doc`:

```text
ansible-doc ansible.builtin.file
ansible-doc --list
```

The online collection index provides the same information with cross-links. Documentation should match the installed collection version because parameters and behaviour can change between releases.

The command module executes a program directly without a shell:

```text
ansible all -m ansible.builtin.command -a "date"
```

Shell operators such as pipes, redirection, variable expansion, and command substitution do not work through `ansible.builtin.command`. Tasks that genuinely require those features can use `ansible.builtin.shell`, although a purpose-built module usually offers safer quoting, stronger idempotence, and clearer results.
## Check mode, diff mode, and validation
Check mode simulates supported tasks without changing managed systems. It predicts whether a module would change a target. It does not expose every internal command, and it cannot guarantee the result of a real run. Modules that do not support check mode do nothing and report no useful prediction. Later tasks can also behave differently when they depend on output that an earlier simulated task could not produce.

The following command previews a playbook:

```text
ansible-playbook site.yml --check
```

Diff mode shows before-and-after details for modules that support it. It can operate during a real run or alongside check mode:

```text
ansible-playbook site.yml --check --diff --limit web01
```

This combination helps validate file, template, user, and other configuration changes on a restricted target. Diff output can be large and can expose sensitive content, so production use should limit hosts and protect secrets. Neither mode replaces testing in a representative environment.

The `check_mode` and `diff` keywords can control behaviour at task or play scope. `check_mode: true` forces a task to simulate even during a normal run. `check_mode: false` forces it to execute even when the command includes `--check`. Such overrides require care because a supposedly non-changing validation run can otherwise alter a target.

Ansible provides several additional validation tools:
- `ansible-playbook site.yml --syntax-check` checks playbook syntax.
- `ansible-playbook site.yml --list-hosts` shows the hosts a play would target.
- `ansible-playbook site.yml --list-tasks` shows selected tasks.
- `ansible-inventory --list --yaml` displays the resolved inventory.
- `ansible-config dump --only-changed` displays active non-default configuration.
- `ansible-lint` checks playbooks against configurable quality and safety rules.

These checks answer different questions. Syntax validation cannot prove that credentials work, inventory selects the intended assets, a module supports the target platform, or the final state is correct. A safe release process combines static checks, check mode where useful, a constrained real run, and verification of the resulting service.
## Inventory, configuration, and connections
Inventory describes the hosts Ansible can manage. It may come from a command-line host list, an INI or YAML file, a directory of sources, or a dynamic inventory plugin. Inventory can also define groups, variables, aliases, and connection behaviour.

An inline list needs a trailing comma so Ansible treats it as a host list rather than a filename:

```text
ansible all -i "pi4,pi5," -m ansible.builtin.ping
```

The host pattern at the end selects a subset of inventory. The automatic `all` group contains every inventory host. Patterns can select groups, intersections, unions, exclusions, and host ranges. For example, `all:!retired` selects all hosts except those in the `retired` group.

A static INI inventory can group hosts and use ranges:

```ini
[raspberry_pis]
pi[1:5]

[webservers]
web01
web02
```

Ansible reads configuration from several sources according to documented precedence rules. A project-level `ansible.cfg` can point to inventory and record other shared defaults:

```ini
[defaults]
inventory = inventory.ini
```

The command `ansible-config dump --only-changed` reveals active changes and their origins. It can expose a misspelt section name or an ignored option. `ansible-config list` documents available settings, while `ansible-config view` displays the active configuration file. Project teams should keep safe, non-secret configuration with their playbooks and avoid embedding credentials.

Remote POSIX hosts commonly use the SSH connection plugin. Ansible relies on existing SSH authentication, host-key verification, and account configuration rather than installing a resident agent. A normal SSH connection should work before Ansible troubleshooting begins:

```text
ssh pi3
ansible pi3 -m ansible.builtin.ping
```

For many Python-based modules, Ansible transfers generated code to the remote host, locates a compatible interpreter, executes the module, captures structured output, and removes temporary files. Other modules and plugins use different mechanisms. Cloud modules may call an API from the controller, network modules may use a device transport, and container plugins may invoke Docker through its CLI or API.

Ansible creates an implicit `localhost` when a task explicitly targets it and no matching inventory host exists. This host uses the local connection rather than SSH. It does not automatically join the effective inventory in the same way as an explicitly defined host, so a pattern such as `all` may not include it. Automation that must include the controller should define or target it deliberately.
## Privilege escalation and facts
Some tasks need privileges beyond those of the connection account. Package installation, system-wide configuration, and protected file changes commonly require escalation. The `become` mechanism applies privilege escalation on the machine where the task executes:

```text
ansible pi3 -m ansible.builtin.timezone \
  -a "name=Australia/Sydney" --become
```

If the escalation tool requires a password, `--ask-become-pass` prompts for it. A playbook can set `become: true` at play or task scope. It can also define the escalation method or target user when the default does not fit. Prefixing the controller command with `sudo` elevates the Ansible process on the controller, not the remote command, so it does not replace `become` for remote hosts.

Credentials should not appear in plain text in inventory, playbooks, shell history, or version control. Ansible Vault can encrypt sensitive variables and files. Central automation platforms can supply managed credentials without exposing them in a project repository.

Playbooks gather facts by default at the start of each play. The `ansible.builtin.setup` module collects operating-system, network, hardware, interpreter, and other information. An ad hoc query can filter the output:

```text
ansible all -m ansible.builtin.setup \
  -a "filter=ansible_distribution*"
```

Facts support platform-aware automation. A macOS-specific task can use a `when` expression based on `ansible_distribution`, while Linux hosts skip it. A debug task can display a variable during development:

```yaml
- name: Show the detected distribution
  ansible.builtin.debug:
    var: ansible_distribution
```

Conditionals belong to the task, not to the module argument dictionary:

```yaml
- name: Configure a macOS preference
  community.general.osx_defaults:
    domain: NSGlobalDomain
    key: AppleAccentColor
    type: int
    value: 6
  when: ansible_distribution == "MacOSX"
```

Fact collection consumes time. A play that does not use facts can set `gather_facts: false`. A play that needs only limited categories can use `gather_subset` or call the setup module with filters. Disabling facts solely for speed can break variables, conditions, and modules that rely on the information.

Interpreter discovery selects a Python executable on each managed POSIX host. A warning deserves investigation when the discovered path might change after an installation or upgrade. Pinning `ansible_python_interpreter` can provide stability when the environment guarantees that path. Silent discovery settings suppress warnings but do not correct an unsuitable interpreter.
## Playbooks as executable configuration
A playbook records repeatable automation in YAML. Each play selects hosts and contains an ordered list of tasks. Each task normally calls one module. The following play installs NGINX and replaces its default page on Debian-family hosts:

```yaml
---
- name: Deploy a web server
  hosts: webservers
  become: true
  tasks:
    - name: Install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Publish the application page
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"
```

The YAML dictionary form exposes module arguments clearly and works well with editor completion, schema validation, and linting. The older `key=value` shorthand remains useful for compact ad hoc commands, but structured YAML improves maintainability in playbooks.

The play runs with:

```text
ansible-playbook site.yml
```

The output groups results by play and task, then summarises each host. `ok` means the task succeeded without changing the target. `changed` means it altered the target. `failed`, `unreachable`, `skipped`, and `rescued` report other outcomes. Increasing verbosity with `-v`, `-vv`, or `-vvv` reveals progressively more detail, although high verbosity can expose sensitive values and generate substantial output.

Tasks compose into outcomes that no single module provides. A web deployment may install packages, create users and directories, render configuration, distribute application files, start services, and verify health. Each module handles one concern, while the playbook defines ordering and scope.

Tags allow selective execution:

```yaml
- name: Install NGINX
  ansible.builtin.apt:
    name: nginx
    state: present
  tags:
    - nginx
```

```text
ansible-playbook site.yml --tags nginx --check --diff
```

Tags reduce the work during focused testing, but tagged execution can omit prerequisite tasks. A task selected alone may depend on variables, files, packages, or facts established elsewhere. Tags therefore need deliberate design and documentation.

The `--limit` option narrows the hosts selected by a play without expanding them beyond the play's `hosts` pattern. A cautious production run can combine `--limit`, `--check`, and `--diff`, followed by a real run against the same small cohort.

Editor support can improve authoring. The Red Hat Ansible extension for Visual Studio Code provides completion, documentation, and validation. `ansible-lint` identifies problems such as risky shell use, ambiguous module names, missing task names, and non-idempotent patterns. Teams can configure rule profiles, but they should understand a rule before suppressing it. A formatter and ordinary YAML validation can also correct indentation and trailing whitespace.

The `ansible-doc -t keyword` command documents playbook keywords and their valid scope. It distinguishes Boolean values such as `gather_facts` from lists such as `gather_subset`, and it shows whether a keyword applies to a play, block, role, or task.
## Collections and fully qualified names
Collections package reusable Ansible content. A collection can contain modules, inventory plugins, connection plugins, lookup plugins, filter plugins, roles, playbooks, and supporting code. Collections let different maintainers develop and release content independently from `ansible-core`.

The `ansible` community package bundles a curated set of collections. `ansible-core` supplies only the core runtime and built-in content. A controller using `ansible-core` can install selected collections from Galaxy or another configured distribution server:

```text
ansible-galaxy collection install community.general
ansible-galaxy collection list
```

Dependencies install with the requested collection. A manually installed collection normally resides under `~/.ansible/collections`, although configuration and the `-p` option can change the search path. Such a collection does not update automatically when the `ansible` Python package updates. The `--upgrade` option updates a manually installed collection. A `requirements.yml` file should pin or constrain versions for reproducible environments:

```yaml
---
collections:
  - name: community.general
    version: ">=11.0.0,<12.0.0"
  - name: community.docker
    version: ">=5.0.0,<6.0.0"
```

```text
ansible-galaxy collection install -r requirements.yml
```

Version numbers apply independently to each collection and do not correspond to the `ansible` or `ansible-core` version. Compatibility requirements in collection documentation remain authoritative. When several copies of a collection exist, the configured collection-path order determines which copy Ansible loads. Project-adjacent collections can isolate projects and keep their dependency definitions near their playbooks.

A fully qualified name follows `namespace.collection.content`. In `community.general.timezone`, `community` is the namespace, `general` is the collection, and `timezone` is the module. Other content types use the same naming model, such as the `community.docker.docker` connection plugin. FQCNs document provenance and prevent collisions between content with the same short name.

Useful discovery commands include:

```text
ansible-galaxy collection list
ansible-doc --list
ansible-doc -t inventory --list
ansible-doc community.general.timezone
```

Galaxy and the official collection index support broader searches by technology, collection, and plugin type. Popularity can help identify established content, but selection should also consider maintainership, release activity, documentation, compatibility, security, dependencies, and licence terms.

`ansible-galaxy collection download` can fetch a collection and its requirements for offline use. Collection removal still relies on removing the intended installation directory. Before any removal, the operator should identify the active collection path and avoid deleting a broader namespace or collection root by mistake.
## Static and dynamic inventory
Static inventory suits stable hosts. Dynamic infrastructure changes as virtual machines, containers, and cloud resources appear or disappear. Inventory plugins query an authoritative source at run time and convert its current resources into Ansible hosts, groups, and variables. This prevents a hand-maintained list from drifting away from the platform.

Docker provides a compact laboratory for this model. The `community.docker.docker_container` module can declare containers on the controller:

```yaml
---
- name: Create the lab containers
  hosts: localhost
  gather_facts: false
  vars:
    container_count: 10
  tasks:
    - name: Ensure lab containers are running
      community.docker.docker_container:
        name: "c{{ item }}"
        image: python:3.13-slim
        state: started
        command: sleep infinity
      loop: "{{ range(1, container_count + 1) | list }}"
```

The current module requires access to a suitable Docker API and controller-side Python dependencies documented by the installed collection. Older guidance that always requires the Docker SDK for Python no longer describes every current `community.docker` module.

A static inventory can connect to a known container through the Docker CLI connection plugin:

```ini
[containers]
c[1:10] ansible_connection=community.docker.docker
```

The Docker inventory plugin removes the hard-coded list. Its YAML source must end with `docker.yml` or `docker.yaml`:

```yaml
---
plugin: community.docker.docker_containers
connection_type: docker-cli
```

The plugin reads the Docker API and produces hosts for the current containers. `connection_type: docker-cli` selects the Docker CLI connection. The current default is `docker-api`, which selects the Docker API connection. Requirements and limitations differ, so the configuration should state the intended type when reproducibility is important.

The inventory can be inspected before use:

```text
ansible-inventory -i lab.docker.yml --list --yaml
ansible all -i lab.docker.yml -m ansible.builtin.ping
```

Dynamic inventory should filter resources so automation reaches only intended assets. Docker labels, names, states, or composed groups can separate a laboratory from unrelated local containers. Cloud inventory should similarly constrain accounts, regions, projects, tags, and lifecycle states.

Desired-state automation repairs simple laboratory drift. If containers stop, the container playbook can start them. If a declared container disappears, the playbook can recreate it. This recovery demonstrates convergence, but production replacement may involve persistent data, identity, health checks, traffic draining, and dependency ordering that a small laboratory does not model.
## Parallelism, batches, and rolling changes
Ansible's default linear strategy runs each task across selected hosts before it advances to the next task. The `forks` setting limits the number of hosts that can execute work concurrently. Raising it can shorten a run when the controller, network, managed systems, and external services can sustain the additional load. Lowering it can protect constrained infrastructure.

Forks control concurrency within a task, not rolling-update safety. With ten web servers and five forks, Ansible can update five hosts at once, then the other five, before it begins the next task. If the first task temporarily leaves every host unable to serve traffic until a later task completes, all hosts can pass through that unsafe intermediate state.

The `serial` keyword divides hosts into batches and completes the whole play for one batch before starting the next:

```yaml
---
- name: Roll out the web application
  hosts: webservers
  serial: 3
  tasks:
    - name: Copy application files
      ansible.builtin.copy:
        src: app/
        dest: /srv/app/

    - name: Render web configuration
      ansible.builtin.template:
        src: server.conf.j2
        dest: /etc/nginx/conf.d/app.conf
        mode: "0644"
```

With ten hosts, `serial: 3` creates batches of three, three, three, and one. Each batch completes all tasks before the next begins. A safe rolling deployment usually adds load-balancer removal, health validation, failure thresholds, and re-entry steps. Batch size should preserve enough healthy capacity for expected traffic and failures.

Execution order and completion order differ. Ansible may start tasks in a predictable inventory order, but varying host performance changes the order of results. Logs should use host names and task names rather than line position as identity. Callback plugins can change output density and structure, while automation platforms can retain structured event data for later analysis.
## Templates, variables, and cleanup
The copy module distributes static content. The template module renders Jinja expressions on the controller and transfers the resulting file to each target. A template can combine facts, play variables, inventory variables, and special variables:

```jinja2
server_name {{ ansible_hostname }}
listen {{ web_port }}
deployment_batch {{ ansible_play_batch | join(',') }}
```

```yaml
- name: Configure web servers
  hosts: webservers
  serial: 3
  vars:
    web_port: 8080
  tasks:
    - name: Render the server configuration
      ansible.builtin.template:
        src: server.conf.j2
        dest: /etc/example/server.conf
        mode: "0644"
```

`ansible_hostname` varies by host. `web_port` remains constant for the play unless a more specific variable overrides it. `ansible_play_batch` contains the active serial batch, so the rendered value changes between batches. Template logic should remain limited. Complex transformations belong in well-named variables, filters, or supporting data so configuration files stay readable.

Removing task definitions does not reverse changes already applied to hosts. Ansible manages declarations that it executes, not the historical ownership of every resource. A container continues to exist after its creation task disappears from a playbook. Explicit cleanup must declare `state: absent` for the intended containers. `force_kill: true` can speed disposal by skipping graceful termination, but production systems should prefer an orderly shutdown unless rapid force is explicitly required.

Repeated arguments can move to `module_defaults` at play scope:

```yaml
- name: Remove lab containers
  hosts: localhost
  gather_facts: false
  module_defaults:
    community.docker.docker_container:
      state: absent
  tasks:
    - name: Remove the numbered containers
      community.docker.docker_container:
        name: "c{{ item }}"
        force_kill: true
      loop: "{{ range(1, 11) | list }}"
```

Module defaults reduce repetition, but broad defaults can hide important behaviour. Their scope and group syntax depend on the collection and module, so the installed documentation should guide their use.

An effective Ansible project keeps playbooks, inventory definitions, dependency constraints, templates, and non-secret configuration under version control. It validates changes, limits early runs, records dependencies, protects credentials, and tests recovery as well as deployment. The resulting automation converts system intent into repeatable, reviewable, and scalable operations.
## Operational practice and troubleshooting
### Convergence and idempotence
Convergence describes the movement from the current state to the declared state. Idempotence describes the ability to repeat an operation without introducing another change after the target has converged. Most configuration modules aim for idempotence, but the outcome depends on arguments and external behaviour. A command that appends a line on every run remains non-idempotent even when Ansible launches it. A dedicated file module can instead test whether the line already exists.

An unexpected `changed` result deserves investigation because a task that changes on every run can restart services, rewrite timestamps, consume API quotas, or hide genuine drift. An unexpected `ok` result also needs attention when the required effect did not occur. The module's documented comparison rules, target permissions, check-mode support, and return data help distinguish a correct result from an incomplete declaration.

Modules do not infer broad intent beyond their arguments. Removing a package task from a playbook does not uninstall the package. Changing a desired path from a file to a directory does not authorise automatic data deletion. Declarations need explicit states and transitions, especially for destructive or irreversible operations.

Module return data supports diagnosis and later tasks. A task can register its result, then inspect fields such as `stdout`, `stderr`, `rc`, `changed`, or module-specific values. Registered data varies by module and can contain secrets, so debugging should display only necessary fields. A failed module often returns a direct explanation, the attempted operation, and relevant target details. Higher verbosity adds transport and interpreter information when the normal message does not reveal the cause.

Several frequent failures have distinct causes:
- `unreachable` usually indicates name resolution, routing, authentication, host-key, transport, or connection-plugin trouble.
- A missing interpreter error indicates that a selected module cannot find a compatible runtime on the managed host.
- A permission error often indicates insufficient access or a missing `become` setting.
- A module-not-found error often indicates an absent collection, an incorrect FQCN, or a collection-path problem.
- An argument error indicates that the task does not match the module interface for the installed version.

Diagnosis should start at the failed layer. A direct SSH test cannot validate an API-based cloud module, and reinstalling Ansible cannot fix a remote privilege policy. `ansible --version`, `ansible-config dump --only-changed`, `ansible-inventory --host HOST`, and the relevant `ansible-doc` page provide a compact view of the execution context.
### Inventory and connection checks
Configuration discovery requires care. Environment variables, command-line options, playbook keywords, variables, and direct assignment can override settings at different precedence levels. A value shown in `ansible.cfg` may therefore lose to a later source. Diagnosis should examine the effective configuration, resolved inventory variables, and task context rather than the contents of one file alone.

Inventory variables can describe connection details without changing a host's inventory name. An alias can map `app01` to an IP address through `ansible_host`, select a login through `ansible_user`, and choose an interpreter through `ansible_python_interpreter`. Group variables apply shared values to related hosts. Host variables handle exceptions. Excessive inventory logic becomes difficult to audit, so behavioural variables should remain deliberate and documented.

Static inventory can combine functional, geographical, and lifecycle groups. A host might belong to `webservers`, `sydney`, and `production` at the same time. Plays can target a useful intersection rather than duplicate host lists. Group membership also supports bulk variables, although ambiguous inheritance across many groups can make the final value difficult to predict.

`ansible-inventory --graph` shows groups and hosts. `ansible-inventory --host app01` displays variables resolved for one host. `ansible all --list-hosts` checks a pattern without connecting. These commands reduce the risk of discovering a targeting error only after a task begins.

Host-key verification protects SSH connections from impersonation. Disabling it can make a disposable laboratory easier to start, but it weakens trust and should not become a routine production workaround. A stable environment should manage known hosts, SSH certificates, or another suitable trust mechanism.
### Playbook structure and recovery
YAML indentation defines structure. Module arguments sit below the module name, while task keywords such as `name`, `when`, `tags`, `become`, `check_mode`, and `register` align with the module name. A condition indented under the module arguments becomes an invalid module option instead of a task condition.

Task names should describe the intended outcome rather than repeat the module name. `Ensure the application package is installed` provides more useful logs than `Run apt`. Clear names help operators locate a failure across many hosts and allow `--start-at-task` to identify a recovery point. Names also improve reviews by expressing why a task exists.

Variables separate reusable intent from environment-specific values. Inventory can hold host and group data, a play can define local defaults, and extra variables can supply a high-precedence override. Secrets should use Vault or a credential service. Variable precedence is powerful, but many hidden overrides can make a play difficult to reason about. Projects should prefer a small number of predictable variable locations.

Version control turns playbooks into an auditable configuration history. A commit can connect a state change with its author, rationale, review, and reversal. A useful review examines more than YAML syntax. It asks which hosts a pattern selects, which privileges a task receives, whether an operation is idempotent, what check mode can predict, how secrets flow, and how a failed partial run recovers.

Playbooks execute tasks in order, but a later failure does not automatically reverse earlier changes. A package installed by task one remains installed when task two fails. Modules should therefore leave useful intermediate states, and deployment designs should include validation, rescue logic, or explicit rollback where the risk warrants it.
### Collection architecture
Ansible originally distributed most content in one package. The collections model separated the execution engine from independently maintained automation content. This architecture lets a cloud collection release fixes without waiting for an `ansible-core` release and lets a controller install only required integrations. The separation also places responsibility on projects to manage collection versions and compatibility.

The broad `ansible` package provides convenience, not a guarantee that every integration is present or current enough. Some collections leave the package, change support policies, or require separate Python libraries. A project should declare the collections it uses even when a developer's global installation already supplies them. An execution environment can package `ansible-core`, collections, Python dependencies, and system libraries into a reproducible container image.

Downloading a collection reveals its structure. Namespace and collection directories contain metadata and content-type directories such as `plugins/modules`, `plugins/inventory`, and `plugins/connection`. This structure explains why `community.general.timezone` and `community.docker.docker_containers` resolve to different files and plugin types even though both use dot-separated names.
### A layered container laboratory
The Docker laboratory contains two automation layers. The first playbook manages containers as resources through the Docker daemon. The second layer treats running containers as managed hosts and configures their internal state. Keeping the layers separate clarifies which connection and inventory apply. Container creation targets `localhost`, while configuration targets hosts returned by the Docker inventory plugin.

The connection plugin determines how tasks enter a container. `community.docker.docker` uses the Docker CLI, while `community.docker.docker_api` uses the API. Neither requires an SSH daemon inside the container. Ansible launches processes through Docker, then runs compatible module code in the container. A command such as `hostname` can confirm that execution occurs inside the intended container by returning its container hostname.

A minimal Python image provides a convenient target because many POSIX modules need Python. Image tags should be pinned when interpreter paths and package behaviour must remain stable. A floating tag can change beneath the laboratory, introduce a new Python version, and invalidate a hard-coded interpreter path. Interpreter discovery can absorb some change, but reproducible training and testing benefit from explicit image versions.

Loops scale declared containers without copying tasks. `range(1, 11)` produces numbers one through ten because the upper bound is exclusive. The task combines each number with a prefix to form `c1` through `c10`. The loop result records one outcome per item, which helps locate a failed container declaration.

The environment can test connectivity, facts, templates, conditional tasks, concurrency, and batch behaviour without changing production hosts. It should remain isolated from valuable local containers because cleanup tasks can remove every resource selected by their names or labels.
### Concurrency and safe batches
The default fork count is often five, but configuration or command-line options can change it. `--forks 2` permits up to two host workers for a task. Actual concurrency may remain lower when a play selects fewer hosts, a task uses throttling, or the strategy imposes another limit. External APIs can also enforce their own rate limits.

Serial values can use numbers, percentages, or a list that changes batch size over the run. A small first batch acts as a canary, followed by larger batches after successful validation. Failure settings such as `max_fail_percentage` can stop a rollout when a batch exceeds an acceptable threshold. These controls limit impact, but health checks must still detect whether a changed host can serve real traffic.

The `free` strategy allows each host to advance independently instead of waiting for every host to finish the current task. It can improve throughput for independent work, but it changes ordering assumptions and does not create a safe rolling deployment by itself. Strategy selection should follow task dependencies and service availability requirements.
### Practical progression
A controlled learning sequence builds confidence without obscuring cause and effect:
1. A local ping confirms installation and module execution.
2. File and directory tasks demonstrate desired state and repeated runs.
3. Check and diff modes show prediction limits and before-and-after output.
4. Static inventory introduces host patterns, groups, variables, and SSH.
5. Privilege escalation enables protected changes without elevating the controller unnecessarily.
6. Playbooks combine modules into repeatable configurations.
7. Facts and conditions adapt one play to different operating systems.
8. Collections add integrations beyond built-in content.
9. Dynamic inventory tracks changing infrastructure.
10. A container laboratory demonstrates drift repair, templates, concurrency, and rolling batches.

Each stage should include a second run. The first run shows the changes needed to converge. The second run tests whether the declaration remains stable. A manual change between runs creates controlled drift and reveals whether the playbook restores the intended state. Check mode can preview supported tasks, but a real isolated target remains the decisive test.

The final operational habit is to inspect both selection and effect. Inventory tools confirm which hosts enter a run. Syntax and lint checks inspect the automation. Check and diff modes provide limited predictions. A constrained real run proves transport, permissions, dependencies, and module behaviour. Service-level validation confirms that the configured system works.