# Linux Administration with Ansible: Writing Ansible Playbooks

> [!NOTE]
> A practical guide to writing readable, idempotent Ansible playbooks for mixed Linux environments, covering YAML structure, inventories, variables, handlers, safe use of native commands, common administration tasks, reusable task composition, and validation practices for reliable automation.

Ansible playbooks define and maintain system state in YAML. They replace repetitive, host-specific administration with repeatable, readable automation. A playbook can install software, manage files, start services, create users, schedule jobs, and combine smaller units of automation into larger workflows.

The material centres on a small Linux estate with Red Hat Enterprise Linux, CentOS Stream, and Ubuntu hosts. The examples stay practical and focus on the patterns that make Ansible reliable across distributions.
## Core ideas
- YAML expresses structured data with indentation, mappings, and lists.
- A playbook is a YAML document that contains one or more plays.
- A play targets hosts and contains tasks.
- A task usually calls an Ansible module.
- Most Ansible modules aim for idempotent behaviour, which means repeated runs converge on the same desired state.
- Variables, facts, handlers, and reusable task files reduce duplication and support mixed environments.
- Native commands remain useful, but they need tighter control than standard modules.
## Inventory and host targeting
Good automation starts with a clear inventory. Groups make host targeting predictable and reduce duplication in later playbooks. A small mixed estate can separate individual hosts, platform families, and broader operating system categories.

```ini
[RHEL]
192.168.0.11

[Stream]
192.168.0.12

[Ubuntu]
192.168.0.13

[RedHat:children]
RHEL
Stream

[Linux:children]
RedHat
Ubuntu
```

This structure supports several levels of targeting.

- `RHEL` targets one Red Hat Enterprise Linux host
- `RedHat` targets both Red Hat-based hosts
- `Linux` targets the full Linux estate
- `all` targets every host in the inventory

A playbook stays simpler when the inventory already models the estate properly. Instead of embedding host-specific logic in tasks, the play can target the right group from the start.

Inventory design also supports variable scope. A variable in `group_vars/RedHat` applies to both Red Hat-based hosts. A variable in `group_vars/Ubuntu` applies only to Ubuntu. This keeps platform-specific values close to the platform itself.

The inventory can also help with diagnostics. Ad hoc commands can display group names and group membership during testing.

```bash
ansible all -m ansible.builtin.debug -a "var=groups.keys()"
ansible all -m ansible.builtin.debug -a "var=groups"
```

These checks confirm that hosts see the same inventory structure and that group membership matches the intended design.
## Playbook anatomy
A useful playbook follows a recognisable shape. Most plays include some combination of these fields:
- `name`
- `hosts`
- `become`
- `gather_facts`
- `vars`
- `tasks`
- `handlers`

A fuller skeleton looks like this.

```yaml
- name: Standard playbook structure
  hosts: all
  become: true
  gather_facts: false
  vars:
    example_pkg: tree

  tasks:
  - name: Ensure the package is installed
    ansible.builtin.package:
      name: "{{ example_pkg }}"
      state: present

  handlers:
  - name: restart_example
    ansible.builtin.service:
      name: example
      state: restarted
```

This structure makes intent easy to scan. The play declares the hosts and privilege model first. Variables sit near the top where they are easy to change. Tasks occupy the main body. Handlers stay at the end and run only when notified.

Ansible also benefits from a tidy directory layout. A practical layout separates each automation topic into its own project directory, then keeps related playbooks, task files, templates, and static files together. That layout reduces path confusion and keeps each project small enough to reason about.
## YAML and playbook structure
YAML is a human-readable data serialisation format. It is not a markup language. In Ansible, YAML expresses keys, values, lists, and nested mappings through indentation.

A simple mapping uses key-value pairs.

```yaml
author: Andrew
company: Pluralsight
```

A list under a key uses a consistent indent and a leading hyphen for each item.

```yaml
departments:
  - hr
  - sales
  - it
```

A nested mapping groups related keys beneath one parent key.

```yaml
user0:
  name: Joe
  age: 27
  department: IT
```

The structure depends on indentation, not visual alignment alone. Tabs can look correct in an editor while breaking YAML because YAML expects spaces. Two spaces per level remain a common and readable convention.

A playbook wraps that structure in Ansible terms. Each play declares the target hosts, privilege escalation when required, and a list of tasks.

```yaml
- name: Install a package with a simple playbook
  hosts: all
  become: true
  tasks:
  - name: Ensure tree is installed
    ansible.builtin.package:
      name: tree
      state: present
```

This pattern matters more than file naming. A file may use `.yml` or `.yaml`, but the meaningful structure is the play containing tasks and module calls.

YAML uses indentation to show relationships. A child value must line up consistently under its parent key. An extra space or a missing space can change the structure completely.

This matters most in three common cases:
- a list under a key
- a nested mapping under a key
- multiple tasks under `tasks`

A list of tasks, for example, is not just a series of keys. It is a list where each list item is a task mapping.

```yaml
tasks:
  - name: Install tree
    ansible.builtin.package:
      name: tree
      state: present

  - name: Start the example service
    ansible.builtin.service:
      name: example
      state: started
```

The playbook stays valid because each task starts at the same indentation level and each module parameter nests beneath that task. This same principle applies throughout YAML. Lists and mappings are not special Ansible constructs. They are basic YAML structures that Ansible interprets later.

Consistent indentation also makes complex values readable. A nested mapping can hold structured module arguments cleanly.

```yaml
user0:
  name: Joe
  details:
    age: 27
    department: IT
```

Using correct names helps here. In modern YAML terms, this is a mapping that contains another mapping. Calling it a hash or a dictionary is reasonable. Calling it a hashed array is imprecise and can confuse the structure.
## Editing YAML safely
Whitespace errors cause more problems than syntax choices. Tabs and spaces are not interchangeable in YAML. A file that looks aligned may still fail if it contains tab characters. Tools such as `cat -vet` expose hidden characters and make tab problems obvious.

A workable editor setup should do four things:
- auto-indent new lines
- insert spaces instead of tabs
- use a two-space tab width
- show syntax and indentation clearly

For `nano`, a minimal `.nanorc` can enforce that behaviour.

```text
set autoindent
set tabsize 2
set tabstospaces
```

For `vim`, a `.vimrc` can apply YAML-specific settings.

```vim
syntax on
set background=dark
autocmd FileType yaml setlocal ai et ts=2 sw=2
```

A capable IDE simplifies the same work. Visual Studio Code, or any editor with solid YAML support, highlights indentation, recognises YAML files automatically, and makes larger playbooks easier to maintain.

YAML block scalars also matter when content spans several lines. The `|` style preserves line breaks. The `>` style folds line breaks into spaces except where YAML preserves paragraph boundaries. That distinction matters when a task writes configuration files or web content.

Literal block style with preserved newlines:

```yaml
content: |
  line one
  line two
```

Folded block style with wrapped text:

```yaml
content: >
  This text wraps across
  multiple lines in the YAML
  file but becomes one line
  when parsed.
```

Using the correct style avoids one of the transcript's main errors. `|` is not the folded style. `>` is the folded style.
## Validating playbooks before deployment
A playbook should pass both style checks and syntax checks before it changes any host.

`ansible-lint` checks style, common mistakes, and some risky patterns. `ansible-playbook --syntax-check` verifies that the playbook parses as Ansible expects. Check mode adds another safety layer by showing intended changes without applying them.

```bash
ansible-lint myplaybook.yaml
ansible-playbook --syntax-check myplaybook.yaml
ansible-playbook -C -v myplaybook.yaml
```

Verbose output also helps when testing or debugging. Ansible increases detail with repeated `v` flags.

```bash
ansible-playbook myplaybook.yaml -v
ansible-playbook myplaybook.yaml -vv
ansible-playbook myplaybook.yaml -vvv
```

A single `-v` usually shows more about task execution and changed state. Higher levels expose deeper argument details and connection information. That becomes useful when a playbook relies on variables, privilege escalation, or platform-specific behaviour.

Validation is strongest when it becomes routine rather than occasional. A practical sequence is:
- write or edit the playbook
- lint it
- run a syntax check
- run in check mode
- inspect the output
- apply the playbook

This workflow supports confidence without slowing development excessively. It also encourages smaller, incremental changes rather than large, risky revisions.

This workflow separates three concerns:
- style and best practice
- syntactic validity
- predicted runtime changes

Check mode is especially useful when a task alters services, authentication, storage, or system files. It does not guarantee a flawless run, but it reduces avoidable surprises.
## Facts, variables, and debugging
Ansible gathers facts by default at the start of a play. Those facts expose information such as operating system family, distribution, network addresses, architecture, and hostnames. When a play does not need that information, disabling fact gathering saves time.

```yaml
- name: Install tree without facts
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Ensure tree is installed
    ansible.builtin.package:
      name: tree
      state: present
```

When a play needs operating system awareness, gathered facts become valuable inputs for logic, debugging, and templates.

```yaml
- name: Show the operating system family
  hosts: all
  gather_facts: true
  tasks:
  - name: Print progress
    ansible.builtin.debug:
      msg: "OS family is {{ ansible_os_family }}"
```

`ansible_os_family` distinguishes broad platform families such as RedHat and Debian. `ansible_distribution` reveals the specific distribution such as Red Hat, CentOS, or Ubuntu.

Facts support two important practices:
- routing different hosts through shared logic
- making playbook output easier to interpret during testing

Variables extend the same idea beyond facts. Playbooks can define variables in the play, the inventory, group variable files, host variable files, or on the command line. Good variable design removes hard-coded values and keeps one playbook usable across multiple environments.
## Scripts versus playbooks
Shell scripts can automate administration, but they carry more of the logic burden. A script that sets a timezone and installs a package may work on one host family, then fail on another because package managers, service names, and file paths differ.

A purely imperative shell approach often needs extra work:
- write distribution detection logic
- copy scripts to remote hosts
- mark scripts executable
- execute them remotely
- handle errors and repeat runs safely

Ansible reduces that burden by moving common logic into modules. Instead of writing separate code for `apt`, `yum`, or `dnf`, a playbook can call the package module and describe the desired end state.

A shell script can still call Ansible ad hoc commands and gain some of that idempotent behaviour.

```bash
cd ~
ansible all -b -m ansible.builtin.package -a "name=tree state=present"
ansible all -b -m ansible.builtin.file -a "path=/etc/localtime src=/usr/share/zoneinfo/Europe/London state=link force=true"
```

This hybrid model is still less structured than a proper playbook, but it is better than imperative commands that rerun unnecessarily and duplicate package-manager logic.

The main distinction is simple. Scripts describe how to do the job. Ansible usually describes the state that must exist.

A simple shell script illustrates the contrast. This example sets the timezone and installs a package on a Red Hat-based host.

```bash
#!/bin/bash
sudo timedatectl set-timezone Europe/London
sudo yum install -y tree
```

It looks clear, but the script assumes a Red Hat-style package manager and a systemd-based system with `timedatectl`. Porting it across distributions means adding more detection logic and more branches.

A shell version that handles both Ubuntu and Red Hat families grows quickly.

```bash
#!/bin/bash
sudo timedatectl set-timezone Europe/London

if [ -f /usr/bin/apt ]
then
  sudo apt update
  sudo apt install -y tree
else
  sudo yum install -y tree
fi
```

Even this remains narrow. It still assumes only two Linux families. It still runs imperative commands rather than describing a final state. It still needs copying and remote execution when it targets other hosts.

Ansible solves the same problem more directly.

```yaml
- name: Set state rather than shell steps
  hosts: all
  become: true
  tasks:
  - name: Ensure tree is installed
    ansible.builtin.package:
      name: tree
      state: present
```

This is why playbooks scale better than ad hoc shell logic. They move the burden of package-manager selection and many repeat-run checks into the module implementation. The administrator then focuses on state, not branching mechanics.

That same distinction matters when work must run across several remote hosts. A plain script usually needs `scp` or some other copy step, then a separate remote execution step. Ansible already has transport, privilege escalation, inventory, and reporting built in.
## Provisioning with Vagrant
Vagrant can use Ansible as a provisioner. That pairing fits local lab environments well because Vagrant creates the machines and Ansible configures them after boot.

A minimal Vagrant provisioner block looks like this.

```ruby
vm.vm.provision "ansible" do |ansible|
  ansible.playbook = "deploy.yaml"
end
```

A single provisioning playbook can create local accounts, grant sudo access, and edit SSH daemon settings across several virtual machines. In practice, this makes lab environments more consistent and easier to recreate.
## Running native commands safely
Native commands remain useful when no suitable Ansible module exists. Four modules matter most in this space:
- `ansible.builtin.command`
- `ansible.builtin.shell`
- `ansible.builtin.raw`
- `ansible.builtin.script`
### `command` for direct commands
`ansible.builtin.command` runs a command on the managed host without invoking a shell there. That makes it the safer default because shell metacharacters, pipelines, and redirection are not interpreted on the remote side.

```yaml
- name: Read the hosts file
  ansible.builtin.command:
    cmd: cat /etc/hosts
```

This behaviour also explains the security difference between `command` and `shell`. If a task should not interpret shell syntax, `command` is the preferred choice.
### `shell` when shell features are required
`ansible.builtin.shell` runs through `/bin/sh` on the managed host. Use it only when the task needs shell behaviour such as redirection, pipes, wildcards, or command substitution.

```yaml
- name: Use shell redirection
  ansible.builtin.shell: >
    echo "hello" > /tmp/example.txt
```

Because a shell interprets the command string, `shell` carries more injection risk than `command`. That does not make it wrong. It simply means the playbook should use it deliberately and keep variable handling tight.
### `raw` for bootstrapping or non-Python targets
Most Ansible modules depend on Python on the managed host. `raw` does not. It sends commands directly over the transport, typically SSH. That makes it useful for first contact with minimal systems or appliances that do not yet have Python installed.

```yaml
- name: Bootstrap Python
  ansible.builtin.raw: yum install -y python3
```

After Python is available, standard modules become practical again.
### `script` for local scripts
`script` copies a local script to the managed host and runs it. This is convenient for small grouped commands that already exist as a script on the controller.

```yaml
- name: Run a local script remotely
  ansible.builtin.script: /home/vagrant/myscript
```

This still lacks the built-in state awareness of many native modules, so repeated runs need careful thought.
## Idempotency and controlled execution
Standard modules usually check whether the desired state already exists. Native commands often do not. A task that calls `fallocate`, `losetup`, or another command may rerun every time unless the playbook adds a guard.

The transcript referred to `creates` as a meta-parameter. In practice, `creates` and `removes` are command arguments supported by modules such as `command` and `shell`. They tell Ansible when a command can be skipped.

```yaml
- name: Create a disk file once
  ansible.builtin.command:
    cmd: fallocate -l 1G /root/disk0
  args:
    creates: /root/disk0
```

That single guard restores safe repeatability. The command runs only when `/root/disk0` does not already exist.

The same pattern applies to other native steps.

```yaml
- name: Create a loop device once
  ansible.builtin.command:
    cmd: losetup /dev/loop100 /root/disk0
  args:
    creates: /dev/loop100
```

Once a native command has created the artefact that the task cares about, a regular Ansible module can often take over. In the storage example, a filesystem module formats the device, and a mount module mounts it. Those later tasks return to declarative state management.

```yaml
- name: Format the loop device
  filesystem:
    fstype: xfs
    dev: /dev/loop100
```

This produces a reliable pattern:
- use native commands only where modules do not cover the task
- add `creates` or `removes` when the command itself is not idempotent
- switch back to native Ansible modules as soon as possible
## Packages, files, and services
Most day-to-day Linux configuration work revolves around three actions:
- install or remove software
- manage files and directories
- manage services

Ansible handles these cleanly with the package, copy, file, and service modules.
### Package management
The package module hides most package-manager differences. It works well when the software package name matches across platforms or when variables abstract the difference.

```yaml
- name: Install Apache
  ansible.builtin.package:
    name: "{{ apache_pkg }}"
    state: present
```
### File management
The copy module can write content directly, copy a local file, or copy a directory tree.

Direct content works for short files:

```yaml
- name: Write a simple index page
  ansible.builtin.copy:
    dest: /var/www/html/index.html
    content: |
      <h1>Simple web server</h1>
      <p>Managed by Ansible</p>
```

A source file works better for longer or richer content:

```yaml
- name: Copy a prepared page
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html
```

A source directory works when the site includes multiple pages, scripts, or assets.

```yaml
- name: Copy a web directory
  ansible.builtin.copy:
    src: web/
    dest: /var/www/html/
```
### Service management
The service module starts, stops, restarts, enables, or disables services while remaining mostly agnostic about the underlying init system.

```yaml
- name: Start and enable Apache
  ansible.builtin.service:
    name: "{{ apache_svc }}"
    state: started
    enabled: true
```

Together, these modules cover most routine system administration work.
## Handling distribution differences with group variables
Ansible is distribution-agnostic only up to a point. Package managers and service managers vary, but so do package names, service names, and some file paths. Variables fill that gap.

For Apache, Red Hat-based hosts use `httpd` while Ubuntu uses `apache2`. Group variable files keep one playbook clean across both families.

`group_vars/RedHat`:

```yaml
apache_pkg: httpd
apache_svc: httpd
```

`group_vars/Ubuntu`:

```yaml
apache_pkg: apache2
apache_svc: apache2
```

The playbook then stays generic.

```yaml
- name: Manage Apache across Linux distributions
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Install Apache
    ansible.builtin.package:
      name: "{{ apache_pkg }}"
      state: present

  - name: Start and enable Apache
    ansible.builtin.service:
      name: "{{ apache_svc }}"
      state: started
      enabled: true

  - name: Copy web content
    ansible.builtin.copy:
      src: web/
      dest: /var/www/html/
```

The same approach works for chrony, where service names and configuration paths differ across host groups.

Package, file, and service tasks also show how idempotent playbooks build in layers. Installing Apache alone is not useful. A useful service needs software, content, and an active service state. The same playbook can build each layer in order.

```yaml
- name: Deploy a simple Apache site
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Install Apache
    ansible.builtin.package:
      name: "{{ apache_pkg }}"
      state: present

  - name: Write the landing page
    ansible.builtin.copy:
      dest: /var/www/html/index.html
      content: |
        <html>
        <body>
        <h1>Managed by Ansible</h1>
        </body>
        </html>

  - name: Ensure Apache is enabled and running
    ansible.builtin.service:
      name: "{{ apache_svc }}"
      state: started
      enabled: true
```

The ordering is deliberate. The package comes first so the document root and service definitions exist. The file comes next so the web content is ready before traffic reaches the service. The service starts last so the site comes online in a usable state.

The copy module supports several useful content strategies.

- `content` for short inline files
- `src` for prepared local files
- a source directory for small trees of related files

Choosing between them is a matter of scale and maintainability. Short demonstrations suit inline content. Reusable pages and configuration files usually belong in standalone files. Directory copies work well for small static web roots.

Chrony adds another layer of service discipline. A time service needs package installation, a platform-specific service name, a configuration file path that may differ between distributions, and a restart only when the configuration changes. That combination makes handlers particularly valuable.

A more complete chrony example looks like this.

```yaml
- name: Standardise chrony
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Install chrony
    ansible.builtin.package:
      name: chrony
      state: present

  - name: Copy the configuration
    ansible.builtin.copy:
      src: chrony.conf
      dest: "{{ chrony_conf }}"
    notify: restart_chrony

  - name: Enable and start the service
    ansible.builtin.service:
      name: "{{ chrony_svc }}"
      state: started
      enabled: true

  handlers:
  - name: restart_chrony
    ansible.builtin.service:
      name: "{{ chrony_svc }}"
      state: restarted
```

This keeps the service stable during routine runs. Only a real configuration change triggers the restart.

The chrony example also reinforces a useful editing rule. Deploy the effective configuration, not the clutter. Comments and blank lines are valuable in vendor files, but a cleaned operational baseline is often easier to track under version control and easier to compare during review.

`group_vars/RedHat` can hold:

```yaml
chrony_conf: /etc/chrony.conf
chrony_svc: chronyd
```

`group_vars/Ubuntu` can hold:

```yaml
chrony_conf: /etc/chrony/chrony.conf
chrony_svc: chrony
```

This technique avoids repeated conditional logic inside every task and keeps platform-specific values close to the inventory.
## Handlers and configuration changes
A normal task runs whenever the play reaches it. A handler runs only when another task notifies it. This suits service restarts because a service usually needs a restart only after a relevant file changes.

Chrony is a clear example.

```yaml
- name: Manage the chrony time service
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Install chrony
    ansible.builtin.package:
      name: chrony
      state: present

  - name: Ensure the service is enabled and running
    ansible.builtin.service:
      name: "{{ chrony_svc }}"
      state: started
      enabled: true

  - name: Deploy the configuration file
    ansible.builtin.copy:
      src: chrony.conf
      dest: "{{ chrony_conf }}"
    notify: restart_chrony

  handlers:
  - name: restart_chrony
    ansible.builtin.service:
      name: "{{ chrony_svc }}"
      state: restarted
```

This pattern improves efficiency and limits unnecessary restarts.

The chrony example also shows another useful habit. A noisy vendor configuration can be reduced to an effective corporate baseline by removing comments and blank lines before deployment. A shorter configuration is easier to audit and easier to version.
## Managing users
The user module handles account creation, modification, and removal. A static user task is straightforward.

```yaml
- name: Create a managed account
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Ensure the ansible account exists
    ansible.builtin.user:
      name: ansible
      state: present
      shell: /bin/bash
```

That task is readable but inflexible. Variables make the account name configurable while keeping a sensible default.

```yaml
- name: Create a managed account
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Ensure the account exists
    ansible.builtin.user:
      name: "{{ user_account | default('ansible') }}"
      state: present
      shell: /bin/bash
```

A caller can then pass the username at runtime.

```bash
ansible-playbook user.yaml -e "user_account=mary"
```
### Creating and removing accounts with one playbook
A playbook can support both creation and deletion by combining variables with `when`.

```yaml
- name: Manage a local account
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Create the account
    ansible.builtin.user:
      name: "{{ user_account | default('ansible') }}"
      state: present
      shell: /bin/bash
    when: user_create == 'yes'

  - name: Remove the account
    ansible.builtin.user:
      name: "{{ user_account | default('ansible') }}"
      state: absent
      remove: true
    when: user_create == 'no'
```

This keeps one playbook responsible for the full lifecycle of the account. `remove: true` ensures Ansible deletes the home directory as well as the account entry.
### Passwords
Linux accounts expect a hashed password value, not clear text. Ansible can generate that hash inline.

```yaml
- name: Create an account with an initial password
  ansible.builtin.user:
    name: "{{ user_account | default('ansible') }}"
    state: present
    shell: /bin/bash
    password: "{{ 'Password1' | password_hash('sha512') }}"
    update_password: on_create
```

`update_password: on_create` matters. Because a hash is one-way, Ansible cannot compare the original clear text string with the current password on the host. Without that setting, a playbook may reset the password more often than intended. With it, Ansible sets the password when the account is first created and leaves later user-managed changes alone.
### SSH key pairs on the controller
The user module can generate an SSH key pair for a local account on the controller.

```yaml
- name: Generate a key pair for the controller account
  hosts: RHEL
  gather_facts: false
  tasks:
  - name: Ensure the controller account has an ECDSA key pair
    ansible.builtin.user:
      name: vagrant
      generate_ssh_key: true
      ssh_key_type: ecdsa
      ssh_key_file: .ssh/id_ecdsa
```

This example uses an ECDSA key pair to avoid clashing with an existing RSA key pair in the lab and to demonstrate a second identity.
### Authorised keys on managed hosts
The `authorized_key` module places the controller's public key into the managed account's `authorized_keys` file.

```yaml
- name: Allow key-based access for the managed account
  ansible.builtin.authorized_key:
    user: "{{ user_account | default('ansible') }}"
    state: present
    manage_dir: true
    key: "{{ lookup('file', '/home/vagrant/.ssh/id_ecdsa.pub') }}"
```

This removes the need for manual `ssh-copy-id` and integrates remote login setup into the same playbook that creates the user.
### Grouping related tasks with blocks
Blocks group tasks so a shared condition can apply once instead of being repeated for every task. This improves readability when account creation, key deployment, and access changes belong together.

```yaml
- name: Manage an automation account
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Create account and access
    block:
    - name: Create the account
      ansible.builtin.user:
        name: ansible
        state: present
        shell: /bin/bash

    - name: Add the public key
      ansible.builtin.authorized_key:
        user: ansible
        state: present
        manage_dir: true
        key: "{{ lookup('file', '/home/vagrant/.ssh/id_ecdsa.pub') }}"

    - name: Create a sudoers file
      ansible.builtin.copy:
        dest: /etc/sudoers.d/ansible
        content: "ansible ALL=(root) NOPASSWD: ALL"
    when: user_create == 'yes'

  - name: Remove the account
    ansible.builtin.user:
      name: ansible
      state: absent
      remove: true
    when: user_create == 'no'
```

This final pattern creates a proper automation account. It can log in with a key and elevate privileges without a password. That makes it suitable for Ansible itself in a lab or tightly controlled environment.

Variables on the command line make user management much more flexible. A single playbook can create different accounts without edits.

```bash
ansible-playbook user.yaml -e "user_account=mary user_create=yes"
ansible-playbook user.yaml -e "user_account=mary user_create=no"
```

That simple mechanism turns one file into a general lifecycle tool. The same logic can default to `ansible` when no account name is passed, which makes the playbook suitable both for a common automation user and for one-off lab accounts.

Verbose output becomes useful here because account names may not appear clearly in short task output when variables drive the task. Higher verbosity levels expose the arguments that the user module actually receives. That makes it easier to confirm whether a playbook is managing `mary`, `ansible`, or another account.

An end-to-end approach can split controller preparation from remote account deployment. The controller play generates a key pair locally. The second play creates the remote account, installs the public key, and grants sudo access.

```yaml
- name: Prepare the controller identity
  hosts: RHEL
  gather_facts: false
  tasks:
  - name: Ensure the controller account has an ECDSA key pair
    ansible.builtin.user:
      name: vagrant
      generate_ssh_key: true
      ssh_key_type: ecdsa
      ssh_key_file: .ssh/id_ecdsa

- name: Create the automation account
  hosts: all
  become: true
  gather_facts: false
  tasks:
  - name: Create account and access
    block:
    - name: Create the account
      ansible.builtin.user:
        name: ansible
        state: present
        shell: /bin/bash
        password: "{{ 'Password1' | password_hash('sha512') }}"
        update_password: on_create

    - name: Add the public key
      ansible.builtin.authorized_key:
        user: ansible
        state: present
        manage_dir: true
        key: "{{ lookup('file', '/home/vagrant/.ssh/id_ecdsa.pub') }}"

    - name: Create the sudoers entry
      ansible.builtin.copy:
        dest: /etc/sudoers.d/ansible
        content: "ansible ALL=(root) NOPASSWD: ALL"
    when: user_create == 'yes'

  - name: Remove the account
    ansible.builtin.user:
      name: ansible
      state: absent
      remove: true
    when: user_create == 'no'
```

This model demonstrates several Ansible techniques at once:
- multiple plays in one playbook
- user creation and removal
- initial password hashing
- SSH key generation
- authorised key deployment
- conditional blocks
- sudo access through a managed file

It also reflects a real operational pattern. A useful automation account needs more than an entry in `/etc/passwd`. It needs a predictable shell, controlled access, and a reliable authentication method.

Managing the SSH key path explicitly also helps. The playbook knows exactly which public key it copies and which private key the controller must use later.

A corresponding SSH test from the controller looks like this.

```bash
ssh -i /home/vagrant/.ssh/id_ecdsa ansible@192.168.0.13
```

Once the login works, the remote account can also verify its sudo rights.

```bash
sudo -l
```

In a lab or controlled automation environment, that sequence confirms that the account can authenticate without a password and can elevate privileges without interactive input. That is the access model Ansible usually needs for non-interactive administration.

The examples also show why blocks improve readability. Without a block, the same `when` clause would repeat across account creation, key deployment, and sudoers management. With a block, the playbook groups those related tasks under one condition and makes the account lifecycle easier to follow.

A final detail matters here. Password creation and SSH access solve different problems. A hashed password supports console or password-based login where allowed. An authorised key supports SSH key-based access. A robust playbook can manage both, but it should do so deliberately rather than accidentally.
## Reusing tasks and playbooks
As automation grows, long files become hard to read and harder to maintain. Ansible supports smaller units of reuse through task inclusion and playbook imports.
### `include_tasks` for dynamic task inclusion
`include_tasks` reads a task file dynamically at runtime. It can use variables that earlier tasks defined.

```yaml
- name: Run a backup task file
  ansible.builtin.include_tasks: backup.yaml
```

This suits task files that depend on runtime context.
### `import_tasks` for static task inclusion
`import_tasks` is static. Ansible imports the task list before execution begins.

```yaml
- name: Import a task file
  ansible.builtin.import_tasks: backup.yaml
```

This can be clearer when the task list is fixed and does not depend on dynamically produced variables.

The transcript used singular forms such as `include_task` and `import_task`. The correct module names are `include_tasks` and `import_tasks`.
### `import_playbook` for top-level composition
Separate playbooks can also be combined in a controller playbook.

```yaml
- import_playbook: archive.yaml
- import_playbook: vdo.yaml
```

This keeps target selection inside each imported playbook while allowing a single top-level command to run broader workflows across the estate.

Task reuse becomes more valuable as projects grow. A backup task, a cron task, and a VDO task can each live in their own YAML file and be assembled into playbooks as needed. That keeps each file focused on one unit of work.

A small backup task file might contain only this:

```yaml
- name: Back up the configuration directory
  ansible.builtin.archive:
    path: /etc
    dest: "/tmp/etc-{{ ansible_hostname }}.tgz"
```

A scheduling task file can stay equally small:

```yaml
- name: Schedule the backup job
  ansible.builtin.cron:
    name: "Weekly /etc backup"
    weekday: "5"
    minute: "0"
    hour: "2"
    user: root
    job: "tar -czf /tmp/etc-{{ ansible_hostname }}.tgz /etc"
    cron_file: etc_backup
```

A playbook can then assemble these pieces.

```yaml
- name: Back up and schedule backups
  hosts: all
  become: true
  gather_facts: true
  tasks:
  - ansible.builtin.include_tasks: backup.yaml
  - ansible.builtin.include_tasks: schedule.yaml
```

This style has two advantages. It improves readability by keeping each unit small, and it encourages reuse across other playbooks. The same backup task can appear in an operational playbook, a lab playbook, or a broader controller playbook.

A controller playbook can sit one level above that and compose full playbooks.

```yaml
- import_playbook: archive.yaml
- import_playbook: vdo.yaml
```

That makes it possible to run one command from the controller while still keeping storage, backup, user, and service automation in separate, understandable files.
## Backups and scheduled jobs
The archive module can create a compressed archive on the managed host. This example backs up `/etc` and names the file with the host's own hostname.

```yaml
- name: Back up the configuration directory
  ansible.builtin.archive:
    path: /etc
    dest: "/tmp/etc-{{ ansible_hostname }}.tgz"
```

A task file containing only that task can be included in other playbooks.

Scheduling uses the cron module. A cron task can create a dedicated file under `cron.d` and keep the backup job under configuration management.

```yaml
- name: Schedule a weekly backup
  ansible.builtin.cron:
    name: "Weekly /etc backup"
    weekday: "5"
    minute: "0"
    hour: "2"
    user: root
    job: "tar -czf /tmp/etc-{{ ansible_hostname }}.tgz /etc"
    cron_file: etc_backup
```

This turns a manual administrative habit into a managed policy.
## VDO workflow
The VDO section demonstrates a larger, storage-oriented workflow on Red Hat-based hosts. The steps follow a clear order:
- install VDO packages
- ensure the VDO service is running
- create the VDO device on a backing disk
- create a filesystem on the mapped VDO device
- create a mount point
- mount the filesystem persistently

A condensed version looks like this.

```yaml
- name: Manage VDO storage
  hosts: RedHat
  become: true
  gather_facts: false
  tasks:
  - name: Install VDO packages
    ansible.builtin.package:
      name:
      - vdo
      - kmod-kvdo
      state: present

  - name: Ensure the VDO service is running
    ansible.builtin.service:
      name: vdo
      state: started
      enabled: true

  - name: Create the VDO device
    vdo:
      name: vdo1
      device: /dev/sdb
      logicalsize: 20G
      state: present

  - name: Create the filesystem
    filesystem:
      fstype: xfs
      dev: /dev/mapper/vdo1

  - name: Create the mount point
    ansible.builtin.file:
      path: /vdo1
      state: directory

  - name: Mount the filesystem
    ansible.builtin.mount:
      path: /vdo1
      src: /dev/mapper/vdo1
      fstype: xfs
      opts: defaults,x-systemd.requires=vdo.service
      state: mounted
```

The example illustrates two broader lessons. First, some workflows naturally mix specialised modules with standard ones. Second, large operations become easier to follow when each task does one thing and the task names describe intent clearly.
## Practical operating rules
A disciplined playbook style makes Ansible easier to trust and easier to maintain.

- Use spaces, not tabs, in YAML.
- Keep task names descriptive.
- Prefer built-in modules over native commands.
- Prefer `command` over `shell` unless shell features are necessary.
- Add `creates` or `removes` when native commands would otherwise rerun pointlessly.
- Use group variables to absorb platform differences.
- Use handlers for service restarts triggered by file changes.
- Use defaults and command-line variables to avoid hard-coded values.
- Keep task files small and compose larger workflows with includes and imports.
- Run lint, syntax checks, and check mode before risky changes.
## A concise workflow for reliable playbooks
A solid Ansible workflow looks like this:
- write clear YAML with consistent two-space indentation
- organise platform-specific values in inventory and group variables
- express state with modules such as `package`, `copy`, `file`, `service`, `user`, `authorized_key`, and `mount`
- drop to `command`, `shell`, `raw`, or `script` only where necessary
- make non-idempotent commands safe with `creates` or `removes`
- trigger restarts with handlers
- validate with `ansible-lint`, syntax checks, and check mode
- split growing automation into reusable task files and imported playbooks

This produces playbooks that stay readable, repeatable, and portable across mixed Linux estates.
## Reading execution output
Ansible output is part of the operating model, not just decoration. A clean run usually answers three questions immediately:
- which hosts the play targeted
- which tasks changed state
- which tasks were skipped because the desired state already existed

This matters when debugging idempotency. If a playbook installs Apache once and later reports no changes, that is usually correct behaviour. If a native command task reports changes every time, that is a sign that the playbook may need `creates`, `removes`, or a more suitable module.

Skip messages also matter. A user-management playbook that says it skipped the create block and ran the delete task has likely interpreted `user_create` correctly. A handler that never appears in the output probably was not notified, which usually means the watched file did not change.

A practical review after each run should check four things:
- changed tasks match the intended scope
- skipped tasks make sense
- handlers ran only when necessary
- repeated runs settle into zero or near-zero change

That pattern is one of the clearest signs that the playbook describes state cleanly.