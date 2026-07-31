# Linux Administration with Ansible: Writing Ansible Playbooks
> [!NOTE]
> Explains how to build reliable, portable Ansible playbooks using clear YAML, inventories, variables, modules, handlers, validation, idempotent execution, and reusable automation components.
## Ansible's operating model
Ansible applies configuration from a control node to managed nodes. A playbook describes the required state in YAML. Each play selects hosts, applies play-level settings, and runs an ordered list of tasks. Each task calls a module or another Ansible action.

The control node reads inventory, variables, configuration, playbooks, roles, and collections. It templates task data before sending the required operation to each managed node. Most POSIX modules need a suitable Python interpreter on the managed node. The `ansible.builtin.raw` and `ansible.builtin.script` modules provide important exceptions.

Ansible encourages declarative administration. A task states that a package must be present, a file must contain approved content, or a service must be running. A well-designed module checks the current state and changes the host only when required. This behaviour supports idempotence, which means that repeated runs converge on the same required state without repeating unnecessary changes.

Idempotence belongs to the complete operation, not to Ansible as a universal property. Modules such as `ansible.builtin.package`, `ansible.builtin.copy`, and `ansible.builtin.user` usually implement state checks. Arbitrary commands and scripts usually do not. Administrators must add guards, change detection, and failure conditions when a module cannot determine the state.

Ad hoc commands suit investigation and isolated operations. Playbooks suit reviewed, repeatable administration because they preserve intent, task order, variables, conditions, handlers, and error handling in version-controlled files. Shell scripts remain useful for local algorithms or vendor tools, but they require explicit logic for distribution differences, privilege, error handling, remote execution, and state detection.
### Selecting the automation interface
The `ansible` command runs one ad hoc task against an inventory pattern. It uses the same module system as a playbook:

```shell
ansible webservers -i inventories/staging/hosts.yml \
  -m ansible.builtin.package \
  -a 'name=tree state=present' \
  --become
```

This form can confirm connectivity, inspect facts, perform a rare operation, or test a module invocation before incorporating it into maintained automation. It does not record a sequence, explain dependencies, or provide a reusable deployment unit. Repeated ad hoc commands should move into a playbook.

Calling ad hoc commands from a shell script can combine Ansible's host selection and modules with an existing workflow, but that structure splits intent across two languages. A playbook usually expresses the same sequence more clearly. If a shell wrapper remains necessary, it should select an explicit inventory, fail on a non-zero Ansible exit status, avoid passing secrets in arguments, and remain under version control.

Module documentation should come from the installed runtime:

```shell
ansible-doc ansible.builtin.package
ansible-doc -s ansible.builtin.user
ansible-doc -t lookup ansible.builtin.file
```

The installed documentation identifies parameters, requirements, return values, check-mode support, collection ownership, and examples for the exact available version. Searching Python files under `/usr/lib` does not provide a reliable module catalogue. Collections, execution environments, installation methods, action plugins, and configured search paths can place content elsewhere.

The configuration in effect also needs verification:

```shell
ansible --version
ansible-config view
ansible-config dump --only-changed
```

Ansible uses the first eligible configuration file in its search order. A project-local `ansible.cfg` can define inventory, roles paths, collection paths, callbacks, forks, and privilege behaviour for that project. Operators should not assume that a home-directory configuration applies when a project file or `ANSIBLE_CONFIG` selects another source.
## YAML for playbooks
YAML represents data through mappings, sequences, and scalar values. Ansible uses that data model for playbooks, inventories, variable files, and role content.

```yaml
---
account:
  name: julie
  shell: /bin/bash
  skills:
    - Python
    - Linux
```

`account` maps to another mapping. `skills` maps to a sequence. Indentation defines structure, so YAML forbids tab characters for indentation. Ansible projects commonly use two spaces at each level, although YAML does not impose that width. Consistent spacing prevents structural ambiguity.

Quoted and unquoted scalars can behave differently. Quoting helps when a value contains a colon followed by a space, begins with a special character, resembles a number, or could be interpreted as another data type. Boolean values should use `true` or `false`. File modes should normally use quoted strings such as `'0644'` so Ansible receives the intended octal representation.

Jinja expressions use double braces when a value needs templating:

```yaml
dest: "/etc/{{ application_name }}/application.conf"
```

Conditions such as `when`, `changed_when`, and `failed_when` already accept Jinja expressions. They should not enclose variables in double braces:

```yaml
when: ansible_facts['os_family'] == 'RedHat'
```

A literal block scalar preserves line breaks:

```yaml
content: |
  first line
  second line
```

A folded block scalar replaces most line breaks with spaces:

```yaml
content: >
  one logical line
  written across several source lines
```

The distinction is important for configuration files, scripts, certificates, and other content whose line structure carries meaning.

Local validation protects confidential inventory values, credentials, and templates. Public web parsers should not receive operational playbooks or secrets. Editor integrations can provide YAML parsing, Ansible schema validation, completion, and lint feedback without disclosing project data. Vim, Nano, Visual Studio Code, and other editors can all enforce spaces and display indentation. Editor choice is less important than a shared project configuration.

An editor should display invisible whitespace, remove trailing spaces, preserve UTF-8, insert a final newline, and expand indentation tabs to spaces. A project-level EditorConfig file can apply common settings across editors:

```ini
root = true

[*.{yml,yaml}]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
```

The file does not replace parsing or linting, but it reduces accidental formatting differences. YAML syntax highlighting alone cannot determine whether a task uses a valid Ansible keyword, whether a collection is installed, or whether a variable exists at runtime.

YAML files can use `.yml` or `.yaml`. A project should choose one convention and apply it consistently. The leading `---` document marker remains common and improves recognition, although Ansible can parse many files without it. File names should describe function, such as `webservers.yml` or `accounts.yml`, rather than execution order alone.
## Project structure and dependencies
A small project can keep a playbook, inventory, variables, templates, and static files together:

```text
ansible-project/
|-- ansible.cfg
|-- requirements.yml
|-- site.yml
|-- inventories/
|   |-- production/
|   |   |-- hosts.yml
|   |   |-- group_vars/
|   |   |   |-- all.yml
|   |   |   `-- webservers.yml
|   |   `-- host_vars/
|   `-- staging/
|       |-- hosts.yml
|       `-- group_vars/
|-- files/
|-- templates/
`-- roles/
```

Larger projects should use roles to group tasks, handlers, defaults, variables, files, templates, and dependencies by function. Separate inventories reduce the risk of applying test settings to production. Version control records intent, enables review, and provides a basis for automated validation.

Ansible content now spans `ansible-core` and separately distributed collections. A short module name can become ambiguous, and some modules from older examples no longer belong to core. Fully qualified collection names identify the exact implementation:

```yaml
ansible.builtin.package:
community.general.archive:
community.general.filesystem:
ansible.posix.authorized_key:
ansible.posix.mount:
```

Projects should declare non-core collections in `requirements.yml` and install them before validation or execution:

```yaml
---
collections:
  - name: ansible.posix
  - name: community.general
```

```shell
ansible-galaxy collection install -r requirements.yml
```

Production projects should constrain dependency versions according to their support and testing policy. Execution environments can package `ansible-core`, collections, Python libraries, and system dependencies into a controlled runtime.

An `ansible.cfg` file should include only deliberate differences from defaults. Generating a large sample and leaving every setting in place obscures the effective policy. `ansible-config dump --only-changed` helps reviewers confirm the result.

Inventory, roles, and collections loaded from writable or untrusted paths can execute code on the control node. Project directories, collection sources, callback plugins, filter plugins, and configuration files therefore require code-review and filesystem controls. Running Ansible with elevated local privilege expands the impact of a malicious plugin and should remain unnecessary for ordinary remote administration.

Tests should install dependencies from `requirements.yml` into a clean environment. This step detects undeclared collections that happen to exist on one administrator's workstation. The same test environment should run linting, syntax checks, and representative Molecule or integration scenarios where the project supports them.
## Playbook anatomy
A playbook contains a sequence of plays. A play targets an inventory pattern and applies tasks to the selected hosts. A compact package play looks like this:

```yaml
---
- name: Install diagnostic utility
  hosts: linux
  become: true
  gather_facts: false

  tasks:
    - name: Ensure tree is installed
      ansible.builtin.package:
        name: tree
        state: present
```

The play name and task name should describe the intended outcome. `hosts` selects an inventory group or pattern. `become: true` requests privilege escalation for the play. `gather_facts: false` skips automatic fact collection because this play does not use host facts.

Fact gathering normally runs before ordinary tasks. It returns operating system, network, storage, hardware, and environment data under `ansible_facts`. A play should gather facts when its logic needs them:

```yaml
- name: Report the platform family
  ansible.builtin.debug:
    msg: "Platform family: {{ ansible_facts['os_family'] }}"
```

Legacy top-level variables such as `ansible_os_family` often remain available, but the `ansible_facts` dictionary makes the source explicit and avoids dependence on fact injection settings.

Playbook keywords and module parameters occupy different levels. In the next task, `when`, `notify`, and `become` control task execution. `src`, `dest`, `owner`, `group`, and `mode` belong to the module:

```yaml
- name: Install application configuration
  ansible.builtin.template:
    src: application.conf.j2
    dest: /etc/application/application.conf
    owner: root
    group: root
    mode: '0644'
  become: true
  when: application_enabled
  notify: Restart application
```

Correct indentation preserves that distinction.
## Validation and controlled execution
Playbook development requires several complementary checks. No single command proves that a change is safe.

```shell
yamllint .
ansible-lint
ansible-playbook -i inventories/staging/hosts.yml site.yml --syntax-check
ansible-playbook -i inventories/staging/hosts.yml site.yml --list-hosts
ansible-playbook -i inventories/staging/hosts.yml site.yml --list-tasks
ansible-playbook -i inventories/staging/hosts.yml site.yml --check --diff
```

`yamllint` checks YAML style and syntax. `ansible-lint` checks Ansible syntax and maintainability rules, including module naming and change reporting. `--syntax-check` loads and parses the playbook but does not test every runtime path. `--list-hosts` confirms the target set. `--list-tasks` exposes the expanded task order where static imports permit it.

Check mode simulates changes only for modules that support it. A module with no check-mode support may skip its operation and return little information. Later tasks that depend on a registered result from a skipped task may behave differently from a real run. Diff mode can display file content, so operators must avoid exposing secrets in terminals, logs, and continuous integration output.

A safe rollout begins in an equivalent test environment. Production execution should narrow both scope and concurrency:

```shell
ansible-playbook -i inventories/production/hosts.yml site.yml \
  --limit web-canary-01
```

After canary verification, `serial` can process hosts in batches:

```yaml
- name: Update web servers in batches
  hosts: webservers
  serial: 2
```

Verbose output supports diagnosis, but higher verbosity can expose arguments and connection details. Sensitive tasks should use `no_log: true`, and log access should follow the same controls as other administrative records.

Validation should follow the same route as execution. The selected inventory, variable files, vault identities, collections, and configuration all affect parsing and runtime behaviour. A syntax check against an empty placeholder inventory can pass while the production inventory exposes an undefined variable or an unsupported platform.

Task results separate several states:
- `ok` means the task completed without reporting a change.
- `changed` means the module reports that it altered the host or would alter it in check mode.
- `failed` means the task returned a failure under its current failure conditions.
- `unreachable` means Ansible could not establish the required connection.
- `skipped` means a condition, tag, check-mode rule, or host selection prevented execution.
- `rescued` means a failure entered a block's rescue path.
- `ignored` means execution continued after a failure that policy explicitly ignored.

Green output does not by itself prove semantic correctness. A task can report `ok` while checking the wrong file, or `changed` on every run because its change logic is incomplete. A second real run in a safe environment provides a practical idempotence test. The second recap should report zero changes unless the playbook intentionally performs a non-idempotent action.

Runtime verification should test the service outcome as well as task status. An HTTP deployment can use `ansible.builtin.uri` to request a health endpoint. An account deployment can query the account database and inspect authorised keys without displaying secret material. A storage deployment can verify the mounted source, filesystem type, capacity, and persistence. Assertions can convert those observations into explicit failures.

Tags and `--start-at-task` help development and recovery, but partial execution can bypass prerequisites. A tagged configuration task may run without the package task that installs its validation command. Playbooks should remain correct when run normally, and documentation should identify any supported partial paths.
## Inventory, variables, and facts
Inventory describes managed hosts and groups. Groups should express function, location, lifecycle, or platform characteristics that genuinely affect policy. A YAML inventory can place hosts in child groups:

```yaml
---
all:
  children:
    linux:
      children:
        webservers:
        time_servers:
    webservers:
      hosts:
        web01.example.net:
        web02.example.net:
    time_servers:
      hosts:
        time01.example.net:
```

`ansible-inventory --graph` and `ansible-inventory --host HOSTNAME` provide clearer inventory inspection than printing the internal `groups` variable from every host.

Variables separate policy data from task logic. A cross-distribution Apache play can use platform-specific values:

```yaml
# inventories/production/group_vars/redhat.yml
apache_package: httpd
apache_service: httpd
apache_document_root: /var/www/html
```

```yaml
# inventories/production/group_vars/debian.yml
apache_package: apache2
apache_service: apache2
apache_document_root: /var/www/html
```

Inventory group names do not automatically follow `ansible_facts['os_family']`. Administrators must create and maintain the corresponding groups, or select a platform variable file from gathered facts. Explicit functional inventory often produces clearer control than inferring all policy from platform facts.

Variables have scopes and precedence. Role defaults provide easy-to-override values. Inventory variables describe host and group policy. Play variables apply within a play. Extra variables supplied with `--extra-vars` have very high precedence and can override safeguards. Projects should validate externally supplied values before using them:

```yaml
- name: Validate requested account state
  ansible.builtin.assert:
    that:
      - account_state in ['present', 'absent']
    fail_msg: "account_state must be present or absent"
```

A typed state value is clearer than paired string flags such as `user_create=yes` and `user_create=no`. One task can then apply the selected state without creating contradictory create and delete paths.

Facts describe observed host properties. Variables describe intended policy. Mixing the two can grant a compromised managed node too much influence over sensitive decisions. Privilege, credentials, repository sources, and trust anchors should come from controlled inventory, role variables, or a secret manager rather than untrusted host facts.

Registered variables capture a task result for later tasks on the same host:

```yaml
- name: Read application version
  ansible.builtin.command:
    argv:
      - /usr/local/bin/application
      - --version
  register: application_version
  changed_when: false

- name: Require an approved major version
  ansible.builtin.assert:
    that:
      - application_version.stdout is match('^4\\.')
```

The registered result can contain `stdout`, `stderr`, `rc`, `changed`, `failed`, and module-specific fields. Code should inspect documented fields rather than parse coloured terminal output. Debug tasks should print only the value needed for diagnosis, especially when results can carry commands, tokens, or file content.

Fact gathering has a cost across large inventories. A play that uses no facts can disable it. A play that needs a narrow subset can call `ansible.builtin.setup` with a filter or `gather_subset`. Fact caching can reduce repeated discovery, but stale cached facts can drive incorrect decisions. Policy should define cache lifetime and invalidate cached data after material platform changes.
## Idempotence and native commands
Modules should replace native commands whenever a supported module expresses the required state. A package task can detect whether installation is required. A copy task can compare content. A user task can inspect account attributes. A command generally reports a change whenever it runs because Ansible cannot infer the command's effect.

Four modules cover common native execution cases:

| Module | Execution model | Appropriate use |
|---|---|---|
| `ansible.builtin.command` | Runs a command without a remote shell | Commands that do not need shell syntax |
| `ansible.builtin.shell` | Runs a command through `/bin/sh` by default | Pipelines, redirection, shell expansion, or compound shell logic |
| `ansible.builtin.raw` | Bypasses the normal module subsystem | Bootstrapping Python or managing a device without a usable Python interpreter |
| `ansible.builtin.script` | Transfers a local script and runs it remotely | Existing scripts that must execute on managed nodes |

The script module does not execute a local script line by line over the connection. It transfers the script to a temporary remote location, invokes it, and removes the temporary copy according to Ansible's execution process.

The command module should take an argument list when values can contain spaces or untrusted data:

```yaml
- name: Query an account
  ansible.builtin.command:
    argv:
      - /usr/bin/getent
      - passwd
      - "{{ account_name }}"
  register: account_query
  changed_when: false
```

`argv` prevents shell parsing and avoids fragile quoting. The task also reports no change because it only reads state.

The shell module enables metacharacters such as `|`, `>`, `<`, `&&`, and variable expansion. That power expands the injection surface. Untrusted values must not be concatenated into shell text. The command module remains the safer choice when shell behaviour is unnecessary.

`creates` and `removes` can guard command, shell, and script operations:

```yaml
- name: Initialise application data
  ansible.builtin.command:
    cmd: /usr/local/sbin/app-init --root /srv/application
    creates: /srv/application/.initialised
```

`creates` skips the command when the named path exists. `removes` skips it when the named path does not exist. These guards only test path existence. They do not prove that every effect of a complex script remains correct. A stronger task may register output and define `changed_when` and `failed_when` from documented exit codes.

The raw module can install a Python interpreter during bootstrap, but normal modules should take over after that point. Raw commands lack the richer argument handling, state checks, and return structure of standard modules. Network platforms may also provide purpose-built connection and network modules that are safer than generic raw commands.

A bootstrap play can gather no facts, test interpreter availability, and install Python only when required. Distribution-specific bootstrap commands may still be necessary because ordinary package facts are unavailable before Python starts. Inventory should group hosts by their known bootstrap image rather than guess the package manager with an `else` branch. After installation, `ansible.builtin.setup` can gather facts and normal roles can proceed.

Scripts require the same security and state controls as direct commands:

```yaml
- name: Run vendor initialisation script once
  ansible.builtin.script:
    cmd: files/vendor-init.sh --root /opt/vendor
    creates: /opt/vendor/.initialised
  become: true
```

The local script should use a valid shebang, strict error handling, quoted variables, absolute paths, and validated arguments. Ansible transfers it, but the remote host still needs the interpreter named by the shebang. The `executable` parameter can select another interpreter when required.

A timezone should not be managed by forcing `/etc/localtime` to a hand-built symbolic link unless platform policy explicitly requires that implementation. A supported timezone module or operating system role can handle the platform's state and associated files. Similar reasoning applies to package repositories, firewalls, SELinux, and storage. Direct file manipulation can bypass validation and auxiliary state that a dedicated module manages.

Native commands should use absolute paths where the remote environment could vary. They should quote internal shell values, set an explicit working directory, constrain input, and avoid depending on interactive profiles. A dedicated module or role becomes worthwhile when the same imperative logic appears repeatedly.
## Provisioning disposable lab systems
Vagrant can invoke Ansible as a provisioner for local virtual machines. The remote `ansible` provisioner runs `ansible-playbook` on the Vagrant host and uses generated connection details. The `ansible_local` provisioner runs Ansible inside the guest. The former keeps the control runtime on the workstation, while the latter needs Ansible and project content inside the guest.

```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
  end
end
```

Provisioning should call the same roles and declared dependencies used by other test environments where practical. A separate lab inventory can supply host-specific values. Vagrant-generated inventory may override connection variables, so operators should inspect the invoked command and inventory when behaviour differs from a direct playbook run.

Lab convenience must not weaken the production baseline. Enabling SSH password authentication, installing a known password, or granting a broad password-free sudo rule can suit an isolated disposable exercise only when the network and credentials remain contained. Reusable roles should default to secure settings, and lab overrides should live in the lab inventory rather than in the main task logic.
## Packages, files, and services
Packages, configuration files, and services form a common configuration sequence:
1. Install the required packages.
2. Deploy approved configuration and content.
3. Enable and start the service.
4. Restart or reload the service only when configuration changes.

The generic `ansible.builtin.package` module acts as a proxy for the detected package manager. It does not translate package names between distributions. The generic `ansible.builtin.service` module similarly acts as a proxy for the detected service manager, but service names can still differ.

```yaml
---
- name: Deploy Apache
  hosts: webservers
  become: true
  gather_facts: true

  vars:
    apache_platforms:
      RedHat:
        package: httpd
        service: httpd
      Debian:
        package: apache2
        service: apache2

  pre_tasks:
    - name: Validate supported operating system family
      ansible.builtin.assert:
        that:
          - ansible_facts['os_family'] in apache_platforms
        fail_msg: "The Apache role does not support this operating system family"

    - name: Select Apache platform values
      ansible.builtin.set_fact:
        apache_package: "{{ apache_platforms[ansible_facts['os_family']]['package'] }}"
        apache_service: "{{ apache_platforms[ansible_facts['os_family']]['service'] }}"

  tasks:
    - name: Ensure Apache is installed
      ansible.builtin.package:
        name: "{{ apache_package }}"
        state: present

    - name: Install the home page
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Ensure Apache is enabled and running
      ansible.builtin.service:
        name: "{{ apache_service }}"
        enabled: true
        state: started
```

Group variables can replace the platform mapping when inventory already carries reliable platform policy. A role can place these values in `vars/RedHat.yml` and `vars/Debian.yml`, then select the correct file after gathering facts.

The copy module handles static files and short literal content. Its `content` parameter writes a complete file and should specify a file path as `dest`. The `src` parameter reads from the control node unless `remote_src: true` changes that behaviour. A trailing slash on a source directory copies its contents, while a source directory without the slash copies the directory itself. Ownership and mode should be explicit.

Directory deployment through `copy` suits small static trees. Large trees can make the module perform expensive recursive checks. Packages, synchronisation tools, artefact repositories, or application deployment modules may scale better. Deleting a source file does not automatically remove an old remote file from a copied directory, so exact mirroring needs an explicit design.

Inline `content` replaces the destination file with the supplied string. It should not manage one setting inside a complex vendor file. `lineinfile` can enforce a well-defined line, `blockinfile` can own a marked block, and `template` can own the whole configuration. Whichever method applies, one authority should control each region of the file. Overlapping tasks can undo one another and report changes on every run.

Templates suit files whose content depends on variables. A Jinja template provides cleaner structure than embedding a long configuration under `content`. It also supports validation before replacement:

```yaml
- name: Install sudo policy
  ansible.builtin.template:
    src: ansible-operator.j2
    dest: /etc/sudoers.d/ansible-operator
    owner: root
    group: root
    mode: '0440'
    validate: /usr/sbin/visudo -cf %s
```

Validation prevents an invalid candidate file from replacing the active file. Similar validation should protect web server, SSH, and time-service configuration.
### Handlers and change-driven service control
A handler runs after a notifying task reports `changed`. Ansible normally runs notified handlers at defined synchronization points, including the end of a play's main task section. Repeated notifications of the same handler lead to one run at that point.

```yaml
- name: Manage Chrony
  hosts: time_servers
  become: true
  gather_facts: true

  vars:
    chrony_platforms:
      RedHat:
        package: chrony
        service: chronyd
        config: /etc/chrony.conf
      Debian:
        package: chrony
        service: chrony
        config: /etc/chrony/chrony.conf

  tasks:
    - name: Select Chrony platform values
      ansible.builtin.set_fact:
        chrony_data: "{{ chrony_platforms[ansible_facts['os_family']] }}"

    - name: Ensure Chrony is installed
      ansible.builtin.package:
        name: "{{ chrony_data['package'] }}"
        state: present

    - name: Install Chrony configuration
      ansible.builtin.template:
        src: chrony.conf.j2
        dest: "{{ chrony_data['config'] }}"
        owner: root
        group: root
        mode: '0644'
      notify: Restart Chrony

    - name: Ensure Chrony is enabled and running
      ansible.builtin.service:
        name: "{{ chrony_data['service'] }}"
        enabled: true
        state: started

  handlers:
    - name: Restart Chrony
      ansible.builtin.service:
        name: "{{ chrony_data['service'] }}"
        state: restarted
```

The play should validate the operating system family before indexing the mapping. Configuration syntax should also be validated when the service provides a suitable check command. A forced restart is not idempotent by itself, but the handler restricts it to configuration changes.

`ansible.builtin.meta: flush_handlers` can run notified handlers before the normal synchronization point when later tasks require the new service state. Early flushing should remain deliberate because it changes task order.
## Account management and access
The `ansible.builtin.user` module creates, modifies, and removes local accounts. A single state variable can control the lifecycle:

```yaml
---
- name: Manage local automation account
  hosts: linux
  become: true
  gather_facts: false

  vars:
    account_name: ansible-operator
    account_state: present

  pre_tasks:
    - name: Validate account state
      ansible.builtin.assert:
        that:
          - account_state in ['present', 'absent']

  tasks:
    - name: Apply account state
      ansible.builtin.user:
        name: "{{ account_name }}"
        state: "{{ account_state }}"
        shell: /bin/bash
        create_home: true
        remove: "{{ account_state == 'absent' }}"
```

`remove: true` asks the platform's account tool to remove associated directories when the account becomes absent. It requires careful review because account removal can destroy a home directory and mail spool. Production workflows should separate approval for creation, modification, suspension, and deletion.

Supplementary group management also needs care. Supplying `groups` without `append: true` removes memberships not listed in the task. That behaviour can enforce an exact policy, but an accidental incomplete list can remove required access.
### Passwords and secrets
Linux expects the user module's `password` value to contain a password hash, not clear text. A literal clear-text password inside a playbook remains exposed even when a Jinja filter hashes it during execution. Source control, process arguments, logs, editor recovery files, and job output can all disclose it.

A safer account task receives an approved hash from a protected variable:

```yaml
- name: Create account with managed password hash
  ansible.builtin.user:
    name: "{{ account_name }}"
    password: "{{ account_password_hash }}"
    update_password: on_create
  no_log: true
```

The variable can come from Ansible Vault or an external secret manager. Vault protects encrypted data at rest. Once Ansible decrypts the value for use, task authors must still prevent disclosure with access controls, careful logging, and `no_log`.

`update_password: on_create` sets the supplied password only for a newly created account. `always` updates it when the supplied hash differs. The correct choice follows the organisation's ownership model for password rotation. A fixed hash should not stand in for a centrally managed identity system.

Extra variables placed directly on a command line can appear in shell history and process inspection. A vaulted variable file, a protected secret integration, or a private prompt provides a safer path. Diff mode and debugging must not reveal secret-backed files.
### SSH public-key authentication
SSH authentication proves possession of a private key against a public key installed for the remote account. The private key should remain under the control of its owner or an approved credential service. Ansible normally distributes only public keys.

`ansible.posix.authorized_key` belongs to the `ansible.posix` collection:

```yaml
- name: Authorise the automation public key
  ansible.posix.authorized_key:
    user: "{{ account_name }}"
    key: "{{ lookup('ansible.builtin.file', 'files/ansible-operator.pub') }}"
    state: present
    manage_dir: true
```

The lookup reads the public key on the control node. The module manages the target account's `authorized_keys` file and can create the `.ssh` directory with appropriate ownership and permissions.

`generate_ssh_key: true` under `ansible.builtin.user` generates a key pair for the managed user on the host where that task runs. It does not inherently generate a controller key or copy a public key to other hosts. A task that must create controller-side credentials should target or delegate to the controller explicitly, protect the private key, avoid overwriting an existing key, and follow organisational key-generation policy.

No key algorithm is universally best for every compatibility and compliance requirement. Modern OpenSSH environments often support Ed25519, while some regulated or legacy environments require an approved RSA configuration. Policy, client support, cryptographic libraries, and lifecycle management should determine the selection.

Public-key access does not by itself grant privilege escalation. An automation account may need `become`, but a broad `NOPASSWD: ALL` rule grants extensive power. The account should have tightly controlled credentials, host access, network reachability, audit logging, and job permissions. Where command-specific sudo rules conflict with Ansible's temporary module execution, an automation platform can mediate access instead of distributing unrestricted interactive credentials.

Account provisioning should follow a complete lifecycle:
- Create a named account or bind a centrally managed identity.
- Apply the intended primary group, supplementary groups, shell, home, and expiry.
- Install approved public keys and remove retired keys.
- Grant only the required escalation path.
- Verify login and escalation through the automation channel.
- Suspend access before destructive deletion when investigation or handover may be required.
- Remove credentials, sudo policy, scheduled jobs, and owned service access during offboarding.
- Retain or archive data according to policy before requesting home-directory removal.

The `exclusive: true` option of `ansible.posix.authorized_key` can remove keys that the task does not supply. That option supports exact key ownership, but loops can remove keys installed by earlier loop iterations unless the complete key set is supplied together. Shared access files need one declared owner and a reviewed source of truth.

Changing `sshd_config` to enable password authentication weakens a key-only environment and should not serve as a routine provisioning step. Any SSH change should use a template or specialised role, validate the candidate configuration with `sshd -t`, notify a reload or restart handler, and preserve an established recovery path.
## Conditions, blocks, and errors
Conditions control tasks from typed data:

```yaml
- name: Install a Red Hat family package
  ansible.builtin.dnf:
    name: httpd
    state: present
  when: ansible_facts['os_family'] == 'RedHat'
```

String comparisons such as `user_create == 'yes'` work, but booleans or state values reduce ambiguity. Conditions should also handle undefined data explicitly when a variable is optional.

Blocks group tasks and apply common directives such as `when`, `become`, `tags`, or environment settings:

```yaml
- name: Configure privileged application files
  become: true
  when: application_enabled
  block:
    - name: Create configuration directory
      ansible.builtin.file:
        path: /etc/application
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Install configuration
      ansible.builtin.template:
        src: application.conf.j2
        dest: /etc/application/application.conf
        owner: root
        group: root
        mode: '0644'
```

`rescue` runs after an ordinary task in the associated block returns a failed state. `always` runs after the block and any rescue tasks. Invalid task definitions and unreachable hosts do not enter `rescue` in the same way as an ordinary failed task.

```yaml
- name: Apply configuration with recovery
  block:
    - name: Install candidate configuration
      ansible.builtin.template:
        src: service.conf.j2
        dest: /etc/service/service.conf
        backup: true
      register: service_config

    - name: Validate live service
      ansible.builtin.command:
        argv:
          - /usr/sbin/service-check
          - /etc/service/service.conf
      changed_when: false

  rescue:
    - name: Stop rollout after validation failure
      ansible.builtin.fail:
        msg: "Service configuration validation failed"

  always:
    - name: Record completion
      ansible.builtin.debug:
        msg: "Configuration attempt completed on {{ inventory_hostname }}"
```

Recovery tasks must perform a real recovery when rollback is required. A backup file alone does not restore the previous configuration. Transactional services, snapshots, load balancer delegation, and serial batches may provide safer deployment patterns.
## Reusing tasks, roles, and playbooks
Task files contain a sequence of tasks without play-level keys such as `hosts`. They can be reused through static imports or dynamic includes.

| Mechanism | Processing | Main consequence |
|---|---|---|
| `ansible.builtin.import_tasks` | Static, during playbook preprocessing | Ansible expands tasks before execution |
| `ansible.builtin.include_tasks` | Dynamic, when execution reaches the include | Runtime conditions, loops, and earlier results can select content |
| `ansible.builtin.import_playbook` | Static, at the top level | A control playbook combines complete playbooks |

An imported task file can use variables available during preprocessing and at task execution, but the import statement itself cannot use every runtime feature that an include supports. A dynamic include can use a loop, runtime condition, or host-specific result to choose tasks.

```yaml
tasks:
  - name: Load platform tasks
    ansible.builtin.include_tasks: "tasks/{{ ansible_facts['os_family'] }}.yml"
```

`import_playbook` can appear only at the top level:

```yaml
---
- ansible.builtin.import_playbook: webservers.yml
- ansible.builtin.import_playbook: time-servers.yml
- ansible.builtin.import_playbook: storage.yml
```

Roles provide stronger structure than a growing collection of loose task files. They package defaults, variables, tasks, handlers, templates, files, metadata, and tests behind a named interface. Repeated cross-project functions such as web server deployment, time synchronisation, account policy, and storage configuration usually belong in roles or supported collections.
## Archives and scheduled work
`community.general.archive` creates an archive on the managed node. It does not transfer that archive to the control node or to durable backup storage:

```yaml
- name: Create configuration archive
  community.general.archive:
    path: /etc
    dest: "/var/backups/etc-{{ inventory_hostname }}.tar.gz"
    format: gz
    owner: root
    group: root
    mode: '0600'
```

An archive stored on the same host and disk is not a complete backup. A backup design also requires remote storage, retention, integrity checks, encryption, access control, monitoring, and tested restoration. Archiving `/etc` can capture secrets, so the destination requires strict protection.

`ansible.builtin.cron` manages named crontab entries and files under `/etc/cron.d`:

```yaml
- name: Schedule configuration backup
  ansible.builtin.cron:
    name: Configuration backup
    weekday: '5'
    hour: '2'
    minute: '0'
    user: root
    job: /usr/local/sbin/configuration-backup
    cron_file: ansible_configuration_backup
```

A `cron_file` name should contain portable filename characters, and the task must specify `user`. The managed command should reside in a reviewed script rather than a dense shell expression. The script should use absolute paths, lock against overlapping runs, report failures, and send archives to protected remote storage.

Systemd timers can provide stronger dependency, logging, and missed-run behaviour on systemd hosts. The scheduler should match the operational platform rather than defaulting automatically to cron.
## Storage automation
Storage tasks can destroy data, so they require explicit device selection, preflight assertions, serial execution, backups, and a tested recovery procedure. Fixed device names such as `/dev/sdb` can identify different disks after hardware or topology changes. Stable identifiers, inventory policy, and discovery checks reduce that risk.

The modules used by common storage examples do not all belong to `ansible-core`:
- `community.general.filesystem` creates or manages filesystem signatures.
- `ansible.posix.mount` manages active mounts and `/etc/fstab`.
- Red Hat system roles can manage supported RHEL storage configurations at a higher level.

```yaml
- name: Mount an existing XFS filesystem
  ansible.posix.mount:
    path: /srv/data
    src: UUID=11111111-2222-3333-4444-555555555555
    fstype: xfs
    opts: defaults
    state: mounted
```

`state: mounted` both mounts the filesystem and maintains the persistent entry. It can create the mount-point directory. A separate file task remains useful when policy requires explicit ownership and mode.

A command-driven loop-device example based on a guessed `/dev/loop100` does not safely reserve that device. Another process could already own it, and a `creates` check on the device node does not prove the association. Storage automation should use supported storage abstractions or inspect and register the actual allocation.

VDO provides block-level deduplication, compression, and thin provisioning. Older RHEL examples manage a standalone VDO service and device. Current RHEL uses VDO as an LVM logical-volume type. The supported RHEL storage system role can create an LVM-VDO pool, filesystem, and mount from declared storage data. Capacity planning must distinguish physical size, logical size, index memory, workload deduplication, and failure behaviour. Thin provisioning does not remove the need to monitor physical space.

Storage playbooks should fail before mutation unless the target device, expected size, current signatures, host identity, and intended state all pass validation. Destructive options such as filesystem force flags must never compensate for uncertain discovery.

Storage changes should also distinguish creation from adoption. Creating a new filesystem on an empty device and mounting an existing filesystem require different evidence. An adoption task should verify the expected UUID, filesystem type, label, mount path, and existing data before changing persistent configuration. A creation task should require an explicit empty-device assertion and approval for irreversible formatting.

Check mode cannot guarantee storage safety. Module support varies, discovery can change between review and execution, and a command-based task may not simulate its effect. Maintenance windows should freeze competing storage operations, process one host or failure domain at a time, and verify application access after each change.
## Testing playbook behaviour
A useful test sequence separates defects that require different evidence:
1. Static tests parse YAML, run `ansible-lint`, and confirm declared dependencies.
2. Inventory tests list selected hosts, resolved groups, and representative variables.
3. Planning tests run syntax, check, and diff modes against a disposable environment.
4. Integration tests apply the playbook to clean hosts and verify service outcomes.
5. Convergence tests apply it again and expect no unintended changes.
6. Change tests alter relevant input, apply the playbook, and confirm the required transition.
7. Failure tests supply unsupported values, break a dependency, or trigger validation, then confirm a safe stop or recovery.
8. Rollout tests use a canary and production-sized batches before wider execution.

Clean-host tests detect missing prerequisites that an administrator's long-lived lab may already contain. Convergence tests detect commands, templates, permissions, generated hashes, and handlers that report changes on every run. Change tests detect the opposite defect, where a task reaches `ok` despite a required update.

Assertions should examine externally visible state. Package installation alone does not prove that a service can accept requests. A running service does not prove that it loaded the new configuration. A mounted filesystem does not prove that the application can read and write the intended path with the intended identity.

Test data should include every supported operating system family and significant version. A generic package task can hide differences in package names, repository availability, default configuration, service units, security policy, and filesystem paths. Unsupported platforms should fail with a clear assertion rather than reach a later module error.

Continuous integration should use the same `requirements.yml`, lint configuration, and execution environment as release automation. It should avoid production inventory and live credentials. Where a test needs secrets, the job should receive short-lived credentials through a protected secret facility, restrict output, and destroy the test environment afterwards.

Production verification completes the test cycle. Monitoring, service health, audit events, and user-facing checks can expose failures that task results cannot. A successful recap records Ansible's execution result, while operational acceptance confirms the required system state.
## Durable automation practices
Reliable playbooks share several properties:
- Each play, task, handler, and block has an outcome-focused name.
- Fully qualified collection names identify every module and plugin.
- Supported modules replace shell commands where possible.
- Variables separate policy from reusable task logic.
- Assertions reject unsupported platforms and invalid external input.
- Secrets remain outside ordinary source files and logs.
- File ownership, permissions, and validation commands are explicit.
- Handlers connect configuration changes to reloads or restarts.
- Check mode supports review but never substitutes for staging and canary execution.
- Serial batches constrain production impact.
- Roles and declared dependencies make reuse testable.
- Idempotence tests run each playbook twice and expect no changes on the second run.
- Version control, peer review, linting, and continuous integration gate deployment.

Ansible automates the state that a playbook actually declares. Clear state, constrained inputs, supported modules, and verified rollouts turn that mechanism into dependable Linux administration.