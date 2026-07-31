# Linux Administration with Ansible: Writing Ansible Playbooks
> [!NOTE]
> Explains how to build reliable, portable Ansible playbooks using clear YAML, inventories, variables, modules, handlers, validation, idempotent execution, and reusable automation components.

Ansible playbooks describe the state or sequence of operations that managed hosts must reach. They combine YAML data, Ansible keywords, modules, roles, variables, conditions, and handlers. A well-designed playbook states its intent clearly, uses a supported module or role for each operation, protects credentials, and produces the same desired state when an operator runs it again.

Red Hat Enterprise Linux 10 includes `ansible-core` 2.16. A RHEL 10 control node can manage RHEL 9 and RHEL 10 hosts, while Red Hat Ansible Automation Platform can provide a controlled execution environment for larger operations. Managed RHEL hosts do not need Ansible installed. Most standard modules require Python on the managed host, while connection-specific modules and a small set of bootstrap actions do not.

RHEL system roles provide supported interfaces for complex operating system services. The `redhat.rhel_system_roles` collection covers areas such as firewalls, SELinux, OpenSSH, storage, sudo, systemd, and time synchronisation. These roles reduce direct edits to implementation-specific configuration files and help an organisation apply consistent settings across supported RHEL releases.
## Establishing the automation baseline
An automation project should define its control-node version, collections, roles, inventory sources, and execution dependencies. Version control should contain the playbooks, templates, variable files, and a dependency manifest. An execution environment or another isolated Python environment should hold the matching Ansible packages and collections.

| Component | RHEL 10 baseline |
| --- | --- |
| Control node | RHEL 10 with `ansible-core` 2.16, or a supported Ansible Automation Platform execution environment |
| Managed hosts | RHEL 9 or RHEL 10 hosts reachable through an approved connection |
| System roles | `redhat.rhel_system_roles` from the supported RHEL package or execution environment |
| Core modules | Fully qualified names under `ansible.builtin` |
| Additional modules | Explicitly installed, pinned, and recorded collections |
| Package manager | DNF through `ansible.builtin.dnf` for RHEL-specific playbooks |
| Service manager | systemd through `ansible.builtin.systemd_service` or the RHEL systemd role |
| Secrets | Ansible Vault, an Automation Platform credential, or an approved external secret store |

The control node should use a dedicated, non-root automation account. SSH keys should carry a passphrase and remain under the operator's or platform's credential controls. An SSH agent or Automation Platform credential can unlock a key during a run. Inventory should use DNS names or stable addresses, and SSH host-key checking should remain enabled. Disabling host-key verification weakens the connection by allowing an unverified host to impersonate a managed system.

Privilege escalation should grant only the commands required by the automation. A broad `NOPASSWD: ALL` rule gives a compromised automation account unrestricted root access. Where interactive runs use a sudo password, Ansible can prompt with `--ask-become-pass`. Automation Platform can inject the credential without placing it in source files or command history.

Disposable RHEL 10 virtual machines make suitable development targets. KVM with libvirt, approved cloud instances, or an existing lab can replace old VirtualBox and Vagrant examples. Lab creation should not enable SSH password authentication, reuse a published password, disable host-key checking, or place an unprotected private key in a repository.

Collections form part of the project interface. `ansible.builtin` ships with `ansible-core`, but modules such as `ansible.posix.authorized_key` and `community.general.archive` do not. A `collections/requirements.yml` file should pin required collection versions that work with the chosen execution environment:

```yaml
---
collections:
  - name: ansible.posix
    version: "<approved-version>"
  - name: community.general
    version: "<approved-version>"
```

An organisation should select versions from its tested and supported repository and replace each placeholder with an exact pin. The project should record role dependencies in the same way. `ansible-galaxy collection list`, `ansible-doc`, and the execution-environment build definition reveal what the runtime actually provides. The filesystem location of a module is an internal implementation detail and should not become a project dependency.
## Defining inventory and host scope
Inventory names the managed hosts and organises them into groups. It can come from a static file, an inventory directory, or a dynamic inventory plugin that queries an infrastructure source. The project should treat inventory as operational data, validate its source, and prevent an unreviewed host from entering a privileged group.

A compact YAML inventory can separate web and database tiers:

```yaml
---
all:
  children:
    webservers:
      hosts:
        web-01.example.com:
        web-02.example.com:
    databases:
      hosts:
        db-01.example.com:
```

The inventory name becomes `inventory_hostname`. Connection details such as `ansible_host`, `ansible_user`, and `ansible_port` can differ from that name, but secrets should not appear in an ordinary inventory file. A host should have one stable inventory identity so cached facts, output, and host-specific variables do not split across aliases.

Groups describe an operational characteristic rather than a temporary selection. Names such as `webservers`, `rhel10`, and `sydney_datacentre` communicate useful properties. A broad group should not imply privilege by itself. Variables that grant credentials or enable destructive behaviour need their own controls.

Group variables can hold shared data:

```yaml
---
web_package: httpd
web_service: httpd
web_document_root: /var/www/html
```

The usual project layout places these values in `group_vars/webservers.yml`. Host-specific exceptions belong in `host_vars/<inventory_hostname>.yml`, although many exceptions can indicate that the group model or role interface needs redesign. Secrets belong in an encrypted Vault file or an external credential system, not beside ordinary variables in cleartext.

Host patterns determine the play's initial scope. `hosts: webservers` targets that group, while intersections and exclusions can narrow it. An operator should inspect the resolved list before applying a high-impact change:

```text
ansible-playbook site.yml --list-hosts --limit 'webservers:&rhel10'
```

`--limit` can reduce the command-line scope but cannot add a host that the play's `hosts` pattern excluded. Quoting protects pattern punctuation from the local shell.

An inventory plugin should use a read-only infrastructure credential where possible. Its grouping rules, filters, and cache settings should live in version control. Dynamic discovery does not remove the need to review the resulting host set. A stale cache, broad cloud tag, or incorrect filter can enlarge the target unexpectedly.

Connection variables should choose the supported transport. Standard RHEL hosts normally use the SSH connection. Network devices, containers, and local actions can require different connection plugins. The playbook should not call `raw` as a substitute for selecting the correct platform connection.

The automation account needs permission on each selected host before the main playbook starts. A bootstrap workflow can create that account and install its public key, but the organisation should separate bootstrap authority from routine configuration authority. A permanent production play should not keep account-provisioning credentials that it no longer needs.

Unreachable hosts differ from failed tasks. A block's `rescue` section cannot repair a host that Ansible never reached. Inventory validation, DNS checks, SSH host-key management, credential tests, and `ansible.builtin.ping` can identify connection failures before a change window.
## Writing reliable YAML
YAML represents mappings, sequences, and scalar values. Ansible reads these structures and interprets particular keys as playbook keywords or module arguments. Spaces control YAML indentation. Tabs must not indent YAML, and a consistent two-space increment keeps the hierarchy visible.

A mapping associates keys with values:

```yaml
web_package: httpd
web_service: httpd
web_root: /var/www/html
```

A sequence stores ordered items:

```yaml
web_packages:
  - httpd
  - mod_ssl
```

A sequence of mappings can describe structured records:

```yaml
managed_accounts:
  - name: deploy
    groups:
      - wheel
    state: present
  - name: retired
    groups: []
    state: absent
```

The dash begins each sequence item. A colon followed by a space separates a mapping key from its value. Indentation places `groups` and `state` inside each account record.

Lowercase `true` and `false` provide clear boolean values. Quotation marks should protect strings that resemble other YAML types, begin with a reserved character, contain a colon followed by a space, or need to retain a leading zero. File modes should be quoted so YAML passes them as strings:

```yaml
enabled: true
release_label: "10.0"
service_banner: "Status: ready"
config_mode: "0644"
```

Jinja expressions that begin a value also require quotation marks:

```yaml
dest: "{{ web_root }}/index.html"
```

Single quotes treat most characters literally. A single quote inside a single-quoted scalar appears twice. Double quotes allow escape sequences such as `\n`, so they require more care around backslashes. Plain scalars remain useful for simple words and paths, but selective quotation prevents YAML from converting an intended string.

Block scalars preserve readable multi-line content. The literal indicator `|` retains line breaks, while the folded indicator `>` converts most line breaks to spaces. The chomping indicators `|-` and `>-` remove the final newline:

```yaml
literal_message: |-
  First line
  Second line

folded_message: >-
  This text occupies several source lines
  but becomes one logical line.
```

Comments begin with `#` outside quoted strings. Comments should explain constraints, unusual decisions, or operational risks. Task names should carry the ordinary description of intent, which avoids comments that repeat the code.

YAML anchors can reduce duplication, but they can also hide the effective value and complicate reviews. Ansible variables, defaults, roles, and task reuse usually express shared configuration more clearly. Flow-style YAML such as `{name: httpd, state: present}` saves space but makes later changes harder to review. Block style suits most playbooks.

Local tools should validate YAML that may contain credentials. Sending unpublished inventory, tokens, passwords, or configuration to an online parser can disclose sensitive data. Visual Studio Code with the Red Hat Ansible extension, or another local editor with YAML and Ansible support, can flag indentation, schema, and module errors without uploading the file.

`ansible-lint` examines more than YAML syntax. It detects many Ansible-specific defects, risky shell use, weak task names, non-idempotent patterns, and inconsistent module naming. On RHEL, the DNF package belongs to a Red Hat Ansible Automation Platform subscription. Other environments should install it in an isolated tool environment, such as `pipx` or a virtual environment, and pin a version compatible with the project's `ansible-core`. Installing Python packages into the RHEL system interpreter can conflict with RPM-managed software.
## Structuring plays and tasks
A play maps a host pattern to an ordered list of tasks. A playbook contains one or more plays and runs them from top to bottom. Tasks also run in source order for each host, subject to strategy, serialisation, conditions, failures, and delegation.

```yaml
---
- name: Configure the RHEL 10 web tier
  hosts: webservers
  become: true
  gather_facts: true

  vars:
    web_package: httpd
    web_service: httpd

  tasks:
    - name: Install the web server
      ansible.builtin.dnf:
        name: "{{ web_package }}"
        state: present

    - name: Enable and start the web server
      ansible.builtin.systemd_service:
        name: "{{ web_service }}"
        enabled: true
        state: started
```

The play name and every task name should state a concrete action and object. Useful names improve terminal output, failure reports, and targeted starts with `--start-at-task`. A name such as `Install the web server` communicates more than `DNF task`.

`hosts` selects an inventory pattern. Operators should inspect the selection with `--list-hosts` before running a high-impact playbook. `become: true` belongs at the narrowest practical scope. A play may need root privileges throughout, while a mixed play should apply escalation only to privileged tasks or blocks.

`gather_facts: true` runs the setup module before ordinary tasks. Facts describe each host, including its distribution, interfaces, memory, and Python environment. Fact gathering has a cost, so a play that uses no facts can disable it. A play should not disable gathering and then rely on facts that happen to remain in a cache.

Fully qualified collection names make module ownership explicit:

```yaml
ansible.builtin.copy:
ansible.posix.authorized_key:
community.general.archive:
```

Short names may resolve, but two collections can publish modules with the same short name. FQCNs also connect source code to the correct documentation. A project should use `ansible-doc ansible.builtin.copy` or the corresponding collection documentation instead of inferring parameters from an old example.

Module arguments form a mapping below the module name. Task keywords such as `when`, `register`, `notify`, `become`, `loop`, and `tags` align with the module name. Misplacing a task keyword under the module's arguments produces an unsupported-parameter error or an unintended value.

Tags allow selective execution, but they do not create independent workflows automatically. A tagged subset can fail if it skips prerequisite tasks. Roles and separate playbooks provide stronger boundaries for operations that must run independently.
## Facts, variables, conditions, and results
Variables separate environment data from task logic. Inventory variables, group variables, host variables, role defaults, role variables, play variables, registered results, and extra variables have different precedence. High-precedence input can override a carefully chosen default, so a playbook should validate external values before using them.

Descriptive names such as `account_state` or `web_service_name` reduce ambiguity. Variable names should use letters, numbers, and underscores, and should not begin with a number. A project should avoid names that collide with Ansible keywords, Python methods, or magic variables.

Host facts should use the `ansible_facts` mapping:

```yaml
- name: Display the operating system family
  ansible.builtin.debug:
    msg: "{{ ansible_facts['os_family'] }}"
```

Legacy top-level aliases such as `ansible_os_family` may exist when fact injection remains enabled, but the namespaced form makes the source clear and avoids dependence on that setting.

`debug` can display a variable with `var` or a rendered message with `msg`:

```yaml
- name: Display the selected web service
  ansible.builtin.debug:
    var: web_service

- name: Display the managed host
  ansible.builtin.debug:
    msg: "Configuring {{ inventory_hostname }}"
```

Debug output can expose credentials and personal data. Sensitive tasks should use `no_log: true`, and the playbook should avoid printing secret variables. `no_log` limits Ansible's task output, but it does not protect data that a called program writes elsewhere.

A `when` clause contains a raw Jinja expression and must not use `{{ }}`:

```yaml
- name: Install the RHEL web package
  ansible.builtin.dnf:
    name: httpd
    state: present
  when:
    - ansible_facts['os_family'] == 'RedHat'
    - ansible_facts['distribution_major_version'] | int == 10
```

A list under `when` applies logical AND. A single expression can use `or`, and parentheses can clarify combinations. Filters should normalise a value before comparison when inventory or command-line input may provide a string.

Extra variables have very high precedence. A destructive operation should not treat any non-empty string as consent. An assertion can restrict the accepted states:

```yaml
- name: Validate account input
  ansible.builtin.assert:
    that:
      - account_name is match('^[a-z_][a-z0-9_-]{0,31}$')
      - account_state in ['present', 'absent']
      - account_remove_home is boolean
    fail_msg: "Account input is invalid"
```

The condition can then use the validated boolean directly:

```yaml
remove: "{{ account_remove_home }}"
```

A task can register its result for later decisions:

```yaml
- name: Read the HTTP service state
  ansible.builtin.command:
    argv:
      - systemctl
      - is-active
      - httpd
  register: httpd_state
  changed_when: false
  failed_when: httpd_state.rc not in [0, 3]
```

Registered results commonly expose `rc`, `stdout`, `stderr`, `changed`, `failed`, and `skipped`. The exact fields depend on the module. `changed_when` should describe an observed change accurately, not suppress a genuine change to produce cleaner output. `failed_when` should accept documented non-zero return codes only when the command uses them for an expected state.
## Driving tasks from structured data
Loops apply one task definition to a sequence of values. The loop input should carry enough structure to keep the task readable:

```yaml
required_packages:
  - name: httpd
    state: present
  - name: mod_ssl
    state: present
```

```yaml
- name: Manage required packages
  ansible.builtin.dnf:
    name: "{{ item.name }}"
    state: "{{ item.state }}"
  loop: "{{ required_packages }}"
  loop_control:
    label: "{{ item.name }}"
```

Many modules accept a list directly and can process it more efficiently. DNF can install the entire package list in one transaction:

```yaml
- name: Install required web packages
  ansible.builtin.dnf:
    name:
      - httpd
      - mod_ssl
    state: present
```

A loop remains useful when each item has different parameters, conditions, notifications, or result handling. `loop_control.label` limits noisy output to a meaningful field. It does not hide secrets, so a sensitive loop still needs protected inputs and `no_log: true`.

Nested data should use a distinct loop variable when an included task or role may also use `item`:

```yaml
- name: Configure application instances
  ansible.builtin.include_tasks: tasks/instance.yml
  loop: "{{ application_instances }}"
  loop_control:
    loop_var: application_instance
    label: "{{ application_instance.name }}"
```

This naming prevents an inner loop from overwriting the outer `item`. The included file can reference `application_instance.name` and other fields directly.

Templates should consume structured variables rather than assemble configuration through repeated line edits. A template can render a complete, reviewable file:

```jinja2
{% for backend in application_backends %}
server {{ backend.name }} {{ backend.address }}:{{ backend.port }}
{% endfor %}
```

The task should validate the rendered temporary file before installing it whenever the application provides a validator. Whole-file ownership also establishes a clear boundary. If another tool edits the same file, the organisation should assign one owner or use a documented drop-in directory.

Distribution-specific values can sit in variable files selected by facts:

```yaml
- name: Load operating system variables
  ansible.builtin.include_vars:
    file: "vars/{{ ansible_facts['os_family'] }}.yml"
```

The file name comes from a trusted fact rather than unchecked user input. A RHEL-focused role can simplify the interface further by supporting only RHEL 9 and RHEL 10 and asserting that range. RHEL system roles already encapsulate many release differences, which reduces local branching.

Lists and mappings should describe desired state, not an ordered imitation of terminal commands. A data model such as `managed_accounts`, `firewall`, or `storage_pools` lets the task logic remain stable while environment data changes. Assertions at the role boundary can reject missing keys, invalid choices, unsafe sizes, and unsupported combinations before a module acts.
## Idempotence and native commands
An idempotent task converges a host on the declared state. Running it again against that state should report `ok` and should not repeat a change. Package, file, user, and service modules usually support this model because they inspect current state before acting. Idempotence still depends on module behaviour, input stability, external services, and the resources being managed.

Ansible should use a purpose-built module or RHEL system role whenever one represents the desired resource. A command that installs a package cannot give Ansible the structured state, check-mode support, and error handling available through `ansible.builtin.dnf`. Direct commands remain valid for software or operations that have no suitable module.

The four native execution modules serve different purposes:

| Module | Appropriate use |
| --- | --- |
| `ansible.builtin.command` | Runs a program without shell parsing |
| `ansible.builtin.shell` | Runs syntax that requires a shell, such as a pipeline or redirection |
| `ansible.builtin.script` | Transfers a local script to the managed host and executes it |
| `ansible.builtin.raw` | Sends a command through the remote shell without the normal module subsystem |

`command` should be the default for an external executable. Its `argv` parameter passes each argument separately and avoids quoting ambiguity:

```yaml
- name: Initialise the application database once
  ansible.builtin.command:
    argv:
      - /usr/local/libexec/app-init
      - --database
      - /var/lib/example/app.db
      - --owner
      - example
    creates: /var/lib/example/app.db
```

`creates` skips the command when the named path already exists. `removes` skips it when the named path does not exist. These are module parameters, not universal task controls. They provide partial convergence for a command whose effect has a reliable filesystem marker. A marker should represent successful completion, and a failed program should not create it early.

`command` does not expand shell variables, pipes, redirections, wildcards, or command substitutions. That restriction prevents a large class of injection defects. Ansible performs Jinja rendering before the module runs, so untrusted input still requires validation even without a shell.

`shell` is appropriate when the operation genuinely requires shell grammar:

```yaml
- name: Check whether the application log contains an error
  ansible.builtin.shell:
    cmd: "set -o pipefail && journalctl -u example | grep -q 'fatal error'"
    executable: /bin/bash
  register: log_check
  changed_when: false
  failed_when: log_check.rc not in [0, 1]
```

Shell commands should not concatenate untrusted values. The `quote` filter can protect a value used as one shell argument, but strict validation and `command: argv` provide stronger controls. A task should never place a password, token, or private key in a command string because process listings, callback output, and logs can reveal it.

Inline shell scripts should use a literal YAML scalar and begin with defensive shell options where appropriate. Shell behaviour differs across interpreters, so a task that depends on Bash should set `executable: /bin/bash`. A pipeline needs `pipefail` if failure in an earlier command must fail the task.

`script` copies a file from the control node to the managed host and then executes it. It does not stream and execute each source line separately. It can operate before Python is available on the managed host, but the script still needs its own interpreter and dependencies. `creates` and `removes` can give the action partial check-mode support:

```yaml
- name: Run the vendor bootstrap script once
  ansible.builtin.script:
    cmd: files/vendor-bootstrap.sh --non-interactive
    creates: /var/lib/vendor/bootstrap.complete
```

A maintained module, role, or package should replace a script when the automation becomes long-lived. Scripts often hide state detection, error handling, and sensitive output from Ansible.

`raw` bypasses the module subsystem and passes a command through the configured remote shell. It is suitable for a narrow bootstrap operation, such as installing Python on a host that lacks it. It does not provide normal change reporting or check-mode support. Network automation should use the platform's connection plugin and network modules rather than treating every device as a Python-free Linux shell.

Ad hoc commands can appear in a controlled shell script for a short operational task, but a playbook provides better structure, review, and error reporting. If a script invokes Ansible, it should use an inventory, a restricted host pattern, explicit module names, checked exit status, and quoted arguments. Repeated administration should move into a playbook or role.
## Packages, files, services, and handlers
Package installation, configuration deployment, and service management form a common sequence. On RHEL 10, `ansible.builtin.dnf` expresses package state directly:

```yaml
- name: Install the Apache HTTP Server
  ansible.builtin.dnf:
    name:
      - httpd
      - mod_ssl
    state: present
```

`state: present` accepts the repository's selected version, while `state: latest` requests an upgrade whenever a newer candidate exists. Production playbooks should use an explicit update policy. An unreviewed `latest` operation can introduce unrelated changes during a configuration run.

`ansible.builtin.package` selects a package backend for the managed host, but it does not translate package names. Cross-distribution automation still needs variables or role logic for names such as `httpd` and `apache2`. A RHEL-only playbook should use `dnf` when DNF-specific behaviour or clarity is useful.

`copy` transfers a static source or writes static content. `template` renders a Jinja template. Both can manage ownership, group, mode, SELinux-related attributes, backups, and validation where supported. Configuration files should declare permissions explicitly:

```yaml
- name: Deploy the web home page
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: "0644"
```

`content` suits a short, non-templated value. A template file suits larger content and variable interpolation. Sensitive templates should use `no_log: true` and `diff: false`, but the destination permissions and application logs still require review.

A handler performs a deferred action after a notifying task reports a change:

```yaml
- name: Deploy the Apache configuration
  ansible.builtin.template:
    src: templates/example.conf.j2
    dest: /etc/httpd/conf.d/example.conf
    owner: root
    group: root
    mode: "0644"
  notify: Restart Apache

handlers:
  - name: Restart Apache
    ansible.builtin.systemd_service:
      name: httpd
      state: restarted
```

When a module uses `validate`, the command must accept a temporary file path in the position represented by `%s`. An application-specific validator can prevent deployment of syntactically invalid configuration. A validator for an Apache fragment must also load the surrounding configuration tree, so the project should test the complete candidate configuration in its deployment workflow.

Notified handlers normally run after the relevant play section. Multiple notifications execute the same handler once for each host at that point. This coalescing prevents repeated restarts when several tasks update related files. If a later task requires the changed service immediately, `ansible.builtin.meta: flush_handlers` can run pending handlers earlier. Early flushing should remain exceptional because it adds another service transition.

A handler runs only when its notifying task reports `changed`. A forced `changed_when: true` restarts a service on every run and defeats convergence. A configuration task should rely on the module's comparison or define an accurate change condition.

A RHEL 10 web-service play can combine package, content, service, and firewall state:

```yaml
---
- name: Configure the RHEL 10 web service
  hosts: webservers
  become: true
  gather_facts: true

  pre_tasks:
    - name: Confirm the RHEL major version
      ansible.builtin.assert:
        that:
          - ansible_facts['distribution'] == 'RedHat'
          - ansible_facts['distribution_major_version'] | int == 10
        fail_msg: "This play requires RHEL 10"

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Deploy the home page
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"

    - name: Deploy the virtual host
      ansible.builtin.template:
        src: templates/example.conf.j2
        dest: /etc/httpd/conf.d/example.conf
        owner: root
        group: root
        mode: "0644"
      notify: Restart Apache

    - name: Enable and start Apache
      ansible.builtin.systemd_service:
        name: httpd
        enabled: true
        state: started

    - name: Permit HTTP through firewalld
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.firewall
      vars:
        firewall:
          - service: http
            state: enabled
            runtime: true
            permanent: true

  handlers:
    - name: Restart Apache
      ansible.builtin.systemd_service:
        name: httpd
        state: restarted
```

The firewall role manages the runtime and permanent firewalld state. A public service may also require TLS, a reviewed SELinux policy, application-specific file contexts, and monitoring. SELinux should remain enforcing. Disabling it to make a service start hides a configuration defect and removes a security control.

Time synchronisation should use the RHEL timesync role instead of deleting comments from `/etc/chrony.conf` or replacing fragments without understanding the complete configuration:

```yaml
- name: Configure RHEL time synchronisation
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

The timesync role replaces the configuration of the selected provider. The variable set must therefore describe every required source and option. On RHEL 10, the role normally configures `chronyd`. Network Time Security can authenticate time sources, but a deployment should follow the role's rules for NTS-only source sets.
## Managing users and secure access
Account automation should separate the desired account state from credentials and authorisation. A structured variable can define ordinary attributes:

```yaml
managed_accounts:
  - name: deploy
    comment: Deployment service account
    groups:
      - webops
    shell: /bin/bash
    state: present
```

A loop can apply the records:

```yaml
- name: Manage local accounts
  ansible.builtin.user:
    name: "{{ item.name }}"
    comment: "{{ item.comment | default(omit) }}"
    groups: "{{ item.groups | default(omit) }}"
    append: true
    shell: "{{ item.shell | default('/bin/bash') }}"
    state: "{{ item.state }}"
    password_lock: true
    create_home: true
  loop: "{{ managed_accounts }}"
  loop_control:
    label: "{{ item.name }}"
```

`append: true` preserves supplementary groups that the task does not list. Without it, the module can replace the account's supplementary group membership. `password_lock: true` supports a key-only service account, but the administrator should confirm that no local password workflow requires the account.

Deleting an account can remove data. `state: absent` with `remove: true` may delete the home directory and mail spool. A production playbook should validate the account name, require an explicit deletion state, back up required data to a separate system, and apply an approval process. It should never construct the target name from unchecked external input.

On Linux, the `user` module expects an encrypted password hash, not a cleartext password. It writes the supplied value to the account database without validating the hash. Playbooks should not generate password hashes from cleartext literals, commit hashes to an unprotected variable file, or prescribe an obsolete algorithm. An approved identity process should generate the hash according to current RHEL policy. Vault or a credential system should protect it, and the task should set `no_log: true`.

SSH keys should be generated on the control side or by an approved credential system, not repeatedly on managed hosts as part of ordinary account configuration. RHEL 10 generates Ed25519 keys by default outside FIPS mode and RSA keys in FIPS mode. The private key should have a passphrase, remain outside the playbook repository, and enter an automation run through an SSH agent or platform credential.

The public key belongs in the user's `authorized_keys` file. The `ansible.posix.authorized_key` module resides in the `ansible.posix` collection, not in `ansible-core`:

```yaml
- name: Install the deployment public key
  ansible.posix.authorized_key:
    user: deploy
    state: present
    key: "{{ lookup('ansible.builtin.file', 'public_keys/deploy.pub') }}"
    manage_dir: true
```

The default file name is `authorized_keys`, with a final `s`. A repository may contain public keys, although an organisation should still track their owner, purpose, rotation date, and revocation. The module's `exclusive: true` option removes unspecified keys. It is not loop-aware, so an exclusive policy must pass the complete approved key set in one operation.

Key restrictions can limit port forwarding, agent forwarding, source addresses, or the permitted command. Those restrictions should match the automation's actual needs. A compromised unrestricted key can provide capabilities beyond the intended deployment task.

OpenSSH server settings should use the RHEL sshd role. On RHEL 10, a non-exclusive configuration can use a drop-in under `/etc/ssh/sshd_config.d`:

```yaml
- name: Apply the automation SSH policy
  ansible.builtin.include_role:
    name: redhat.rhel_system_roles.sshd
  vars:
    sshd_config_file: /etc/ssh/sshd_config.d/42-automation.conf
    sshd_config:
      PermitRootLogin: false
      PasswordAuthentication: false
      PubkeyAuthentication: true
```

The operator must install and test a working key before disabling password authentication. A staged change should keep an existing administrative session open, validate the generated configuration, test a second connection, and define a console recovery path. The role manages configuration and service handling more safely than an unvalidated line edit.

Sudo rules should grant named commands rather than `ALL` wherever practical. The RHEL sudo role provides a structured interface:

```yaml
- name: Grant the deployment command
  ansible.builtin.include_role:
    name: redhat.rhel_system_roles.sudo
  vars:
    sudo_sudoers_files:
      - path: /etc/sudoers.d/deploy
        user_specifications:
          - users:
              - deploy
            hosts:
              - ALL
            commands:
              - /usr/bin/systemctl restart httpd
```

The exact command form, arguments, executable path, and service implications need security review. Some permitted programs can launch a shell or write arbitrary files and therefore provide effective root access even when the sudo rule names a single binary.

A project that writes a sudoers fragment directly must set mode `"0440"` and validate it before replacement:

```yaml
- name: Install the reviewed sudoers fragment
  ansible.builtin.copy:
    src: files/deploy-sudoers
    dest: /etc/sudoers.d/deploy
    owner: root
    group: root
    mode: "0440"
    validate: /usr/sbin/visudo -cf %s
```

Blocks apply shared directives and can provide error handling:

```yaml
- name: Configure the deployment account
  become: true
  block:
    - name: Create the account
      ansible.builtin.user:
        name: deploy
        state: present
        password_lock: true

    - name: Install the public key
      ansible.posix.authorized_key:
        user: deploy
        state: present
        key: "{{ lookup('ansible.builtin.file', 'public_keys/deploy.pub') }}"

  rescue:
    - name: Report the failed account task
      ansible.builtin.debug:
        msg: "Account configuration failed at {{ ansible_failed_task.name }}"

  always:
    - name: Record completion in the controller log
      ansible.builtin.debug:
        msg: "Account configuration attempt completed"
```

Tasks in a block inherit directives such as `become`, `when`, and `tags`. A `rescue` section runs after a task returns a failed state, but it does not catch invalid task definitions or unreachable hosts. For hosts that enter block error handling and remain active, an `always` section runs after the block and any rescue tasks. Error handling should not conceal a failed security control. Recovery tasks should restore a safe state or add useful evidence, and the run should fail when the desired security state remains absent.
## Reusing tasks, roles, and playbooks
Small playbooks can keep related tasks together. As an automation project grows, roles provide standard directories for tasks, handlers, defaults, variables, templates, files, and metadata. A role exposes a coherent interface and reduces long collections of one-task files.

Ansible supports dynamic includes and static imports:

| Mechanism | Processing model | Common use |
| --- | --- | --- |
| `ansible.builtin.include_tasks` | Dynamic at run time | File name or execution depends on earlier results, conditions, or a loop |
| `ansible.builtin.import_tasks` | Static during playbook parsing | Task structure is known before execution |
| `ansible.builtin.include_role` | Dynamic at run time | Role invocation depends on run-time state or a loop |
| `ansible.builtin.import_role` | Static during parsing | Role structure should appear in the parsed task graph |
| `ansible.builtin.import_playbook` | Static and top-level | Combines complete playbooks |

`include_tasks` is plural. The old generic `include` keyword is obsolete. Dynamic content may not appear in `--list-tasks` exactly as imported content does because Ansible has not evaluated its run-time decision. Static imports cannot use a file name derived from a result that exists only after an earlier task runs.

A task file contains a list of tasks without a play header:

```yaml
---
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Enable and start Apache
  ansible.builtin.systemd_service:
    name: httpd
    enabled: true
    state: started
```

The calling play can import it:

```yaml
- name: Configure the web service
  hosts: webservers
  become: true

  tasks:
    - name: Apply the Apache tasks
      ansible.builtin.import_tasks: tasks/apache.yml
```

A dynamic include can pass variables and use a loop:

```yaml
- name: Configure selected application instances
  ansible.builtin.include_tasks: tasks/instance.yml
  loop: "{{ application_instances }}"
  loop_control:
    loop_var: application_instance
```

Static and dynamic reuse have different inheritance and evaluation rules. A project should choose deliberately and test conditions, tags, handlers, and variable scope. Mixing both styles throughout one workflow can make the effective task graph hard to inspect.

`import_playbook` can appear only at the top level:

```yaml
---
- name: Import the web-tier playbook
  ansible.builtin.import_playbook: web.yml

- name: Import the database-tier playbook
  ansible.builtin.import_playbook: database.yml
```

An imported playbook contains complete plays with their own `hosts` and task sections. It does not belong inside a play's `tasks` list.

Role defaults should provide low-precedence, user-overridable values. Role variables should remain rare because their high precedence makes overrides difficult. Role arguments can use an argument specification to validate types, required fields, and choices before tasks make changes.
## Scheduled work and backups
`ansible.builtin.cron` can manage traditional cron entries:

```yaml
- name: Schedule the application health check
  ansible.builtin.cron:
    name: application health check
    user: root
    minute: "*/15"
    job: /usr/local/sbin/example-health-check
```

The command should use an absolute path, produce controlled output, and avoid secrets on its command line. A systemd timer may suit RHEL 10 services that need dependency ordering, resource controls, persistent catch-up, structured logs, or stronger unit-level security. The RHEL systemd role can manage systemd units and drop-ins.

An archive is not automatically a backup. `community.general.archive` creates an archive on the managed host and does not copy it to the control node. An archive under `/tmp` or on the same disk shares the host's failure domain and may disappear during cleanup. A valid backup design sends protected data to separate storage, defines retention, verifies restoration, and monitors failures.

When a local archive forms one stage of a backup process, the module name and collection dependency should remain explicit:

```yaml
- name: Create the staged application archive
  community.general.archive:
    path: /srv/example/data
    dest: /var/tmp/example-data.tar.gz
    format: gz
    mode: "0600"
```

A later task or backup system must transfer the archive to durable storage and remove the staging file under an approved retention policy. Removing the source during archival is not a backup operation and can cause data loss.
## Managing LVM-VDO storage on RHEL 10
RHEL 10 integrates Virtual Data Optimizer with LVM. Old playbooks that create a standalone VDO service or call a legacy VDO module do not represent the RHEL 10 storage model. The `redhat.rhel_system_roles.storage` role can create an LVM-VDO volume with compression and deduplication.

Storage automation can destroy data. A playbook should target stable device identifiers, confirm that each device is disposable or approved, restrict the host set, record capacity assumptions, and test recovery before production use. A path such as `/dev/sdb` may identify a different device after hardware or discovery changes. `/dev/disk/by-id` names provide a safer basis where the environment supports them.

The RHEL 10 storage role can create a VDO-backed logical volume:

```yaml
---
- name: Configure LVM-VDO application storage
  hosts: storage_nodes
  become: true

  tasks:
    - name: Create the LVM-VDO volume
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.storage
      vars:
        storage_pools:
          - name: app_vg
            disks:
              - /dev/disk/by-id/example-approved-device
            volumes:
              - name: app_lv
                compression: true
                deduplication: true
                vdo_pool_size: 100 GiB
                size: 500 GiB
                mount_point: /srv/example
```

`vdo_pool_size` defines the physical space consumed by the VDO pool. `size` defines the virtual size presented by the VDO volume. The virtual size may exceed physical capacity because compression and deduplication can reduce stored data, but the saving depends on the workload. Capacity monitoring must track physical data usage and VDO health. Exhausting the physical pool can disrupt writes and damage service availability.

The storage role permits one volume per LVM-VDO pool. Administrators should consult the installed role README for every supported variable because the packaged role defines the interface available on the control node. A syntax check confirms YAML and task structure, but it cannot confirm that a selected disk is safe to erase.
## Validating and operating playbooks
Validation should progress from inexpensive static checks to controlled execution:
1. The editor checks YAML structure and Ansible schemas.
2. `ansible-playbook --syntax-check` parses the playbook in its execution environment.
3. `ansible-lint` examines Ansible-specific quality and safety rules.
4. `--list-hosts`, `--list-tasks`, and `--list-tags` expose the planned scope and task graph.
5. Check mode and diff mode exercise supported simulations on a limited host.
6. A disposable RHEL 10 host receives a normal run.
7. A second normal run checks convergence.
8. Service, security, and application tests verify the resulting state.
9. A canary subset receives the production change before the wider fleet.

Representative commands include:

```text
ansible-playbook --syntax-check site.yml
ansible-lint
ansible-playbook site.yml --list-hosts
ansible-playbook site.yml --check --diff --limit web-canary-01
ansible-playbook site.yml --limit web-canary-01
ansible-playbook site.yml --limit web-canary-01
```

Check mode is a simulation, not proof of a safe run. Modules that do not support it may skip their actions. A later task can also depend on a change that check mode did not make, which changes the simulated path. Command, shell, script, and raw actions have limited or no simulation unless parameters such as `creates` or `removes` provide enough state information.

Diff mode can reveal file content, including credentials. Sensitive tasks should set `diff: false` and `no_log: true`, and the automation platform should restrict job-output access and retention. These controls do not replace secure variable storage.

A second normal run should report no changes for stable configuration. Unexpected repeat changes often reveal a timestamp embedded in a template, an unstable command, a generated file with changing order, a module parameter that resets state, or external software that rewrites managed content. The team should investigate the cause instead of masking the result.

Verification should test outcomes rather than task completion. A web-service play should confirm that systemd reports the unit active, the expected socket listens, firewalld permits only the intended service, SELinux remains enforcing, and an HTTP request returns the required response. A storage play should verify mounts, capacity, ownership, and restoration procedures. A user play should test key authentication, rejected password authentication where required, sudo scope, and revocation.

Production runs should use a reviewed commit, an immutable execution environment, a recorded inventory source, and an auditable credential. `serial` can limit the number of hosts changed at once:

```yaml
- name: Update the web tier in batches
  hosts: webservers
  serial: 2
  max_fail_percentage: 0
```

Batching reduces the failure radius but does not replace load-balancer coordination, health checks, rollback, or application-aware deployment logic. A handler that restarts two hosts at the end of a batch can still affect capacity.

Source-control checks should reject private keys, Vault passwords, cleartext credentials, generated archives, and execution output. Reviews should examine both task logic and variable changes because a one-line inventory or group-variable edit can enlarge scope or enable a destructive state. Protected branches, signed release artefacts, and separate deployment approval can connect the tested commit to the production run. The job record should retain the commit identifier, inventory revision, execution-environment image, limit, initiator, and final result.

Every operational playbook should define ownership, review requirements, supported RHEL releases, required collections, expected privileges, validation commands, rollback or recovery, and post-change tests. Clear task names and small, coherent roles help an operator connect a failed action to the resource that needs attention.