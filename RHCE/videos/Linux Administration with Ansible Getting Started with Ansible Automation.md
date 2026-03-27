# Linux Administration with Ansible: Getting Started with Ansible Automation

> [!NOTE]
> These notes introduce Ansible as a lightweight, SSH-based framework for automating Linux administration across mixed distributions, using a small lab to explain controller setup, inventory and variable management, SSH bootstrapping, core commands and modules, and the shift from ad-hoc tasks to idempotent, repeatable playbooks.

Ansible provides a lightweight way to automate Linux administration from a controller node. It relies on SSH, Python on the managed host, and a human-readable model that separates configuration, inventory, and execution. A practical starter workflow spans a small multi-distribution lab, Ansible installation, the main command-line tools, inventory design, and ad-hoc automation across Red Hat Enterprise Linux, CentOS Stream, and Ubuntu.

The material assumes basic Linux administration skills and routine command-line work. An administrator does not need a large estate to benefit from Ansible. The value appears as soon as repeated work begins to spread across multiple hosts, especially when those hosts do not all use the same package manager, repository layout, service names, or configuration file locations. Ansible reduces that variation to a smaller set of modules, variables, and inventory data.
## Building a repeatable lab
A simple lab can run on a host operating system with VirtualBox as the hypervisor and Vagrant as the machine manager. A three-node layout is enough to demonstrate cross-platform administration:
- Red Hat Enterprise Linux 8 at 192.168.33.11
- CentOS Stream 8 at 192.168.33.12
- Ubuntu 20.04 at 192.168.33.13

Each guest can use 1 GB of RAM and a private network so that the nodes can reach one another directly. The private network matters because Ansible depends on predictable connectivity between the controller and the managed nodes.

A typical working directory keeps the Vagrant definition separate from unrelated projects:

```bash
mkdir -p ~/vagrant/ansible
cd ~/vagrant/ansible
```

The `Vagrantfile` defines the three boxes, their hostnames, their RAM allocation, and their private IP addresses. Once the file exists, Vagrant can create the full lab from that single definition.

```bash
vagrant up
vagrant status
vagrant ssh rhel8
vagrant halt
```

This approach has two benefits. First, it makes the lab reproducible. Secondly, it keeps the focus on Ansible rather than on guest provisioning. The controller can live on RHEL, CentOS Stream, or Ubuntu, but a Red Hat based controller keeps the workflow close to an enterprise environment.

RHEL requires an active subscription before it can use Red Hat repositories. A free Red Hat Developer subscription is sufficient for a personal lab. Registration is straightforward:

```bash
sudo subscription-manager register --auto-attach \
  --username <red-hat-username> \
  --password <red-hat-password>

sudo subscription-manager status
```

If the subscription is active, RHEL can install software from Red Hat content sources. If no subscription is available, CentOS Stream and Ubuntu still provide enough variation to practise most introductory tasks.
## Why Ansible matters
Manual administration works while the estate is small. An administrator can log in to each host, run the required commands, edit configuration files, and restart services. That model fails once the same change must be repeated across many nodes. Repetition introduces two problems:
- time cost
- inconsistency

Shell scripts improve consistency, but they still force the administrator to encode every platform difference by hand. A package install is a good example. Ubuntu uses APT. RHEL 8 and CentOS Stream use YUM or DNF. Even when the package manager difference is hidden, the package name may still differ. The Vim editor illustrates the problem. Ubuntu installs `vim`. Red Hat based systems often install `vim-enhanced`.

A shell solution can inspect `/etc/os-release` and branch on the operating system family:

```bash
#!/bin/bash
id=$(grep '^ID=' /etc/os-release | cut -d= -f2)

if [ "$id" = "ubuntu" ]
then
  sudo apt-get install -y vim
else
  sudo dnf install -y vim-enhanced
fi
```

This script works, but it does not scale elegantly. Every new package, service, file path, and operating system adds more branching. Ansible improves the model by expressing the intended state through modules and variables. The controller chooses the right transport and often the right package backend. The inventory and variable layers handle the remaining differences.
## Installing Ansible
Ansible needs to be installed only on the controller. Managed nodes do not need the full Ansible package. They need SSH access and a suitable Python interpreter.
### RHEL 8
On RHEL 8, the controller can use Red Hat repositories. The first step is to inspect the available Ansible repositories under the active subscription.

```bash
sudo subscription-manager repos --list | grep -i ansible
```

The relevant Ansible repository can then be enabled. The exact repository identifier varies by subscription and repository set, so the command below uses a placeholder.

```bash
sudo subscription-manager repos --enable <ansible-repository-id>
```

A clean RHEL workflow also avoids accidental package selection from EPEL when the intention is to use Red Hat content. The `config-manager` command comes from `dnf-plugins-core`.

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --set-disabled epel epel-modular
sudo dnf install -y ansible
```

RHEL 8 still accepts `yum` syntax because YUM v4 uses DNF underneath. Either command works, though DNF is clearer in new documentation.
### CentOS Stream
CentOS Stream can install Ansible from EPEL.

```bash
sudo dnf install -y epel-release
sudo dnf install -y ansible
```

CentOS Stream is not a rebuild of RHEL in the way that classic CentOS Linux was. It tracks just ahead of the current RHEL release, so it is useful for learning Red Hat style administration without being identical to RHEL.
### Ubuntu 20.04
Ubuntu 20.04 already ships Ansible 2.9.6 in the focal package stream. Some administrators still add the Ansible PPA when they want packaging that moves faster than the base distribution.

```bash
sudo apt update
sudo add-apt-repository -y ppa:ansible/ansible
sudo apt update
sudo apt install -y ansible
```

Once Ansible is installed, the controller can confirm the executable path, file type, and version.

```bash
which ansible
file "$(which ansible)"
ansible --version
```

The version output also reveals the active configuration file, module search paths, executable location, and Python version. That command becomes one of the quickest ways to confirm which configuration file Ansible is actually using.
## Understanding the core tools
Ansible exposes a small group of command-line tools that matter immediately.
### `ansible`
The `ansible` command runs ad-hoc tasks against a host or group. It is useful for quick checks, one-off changes, and experimentation with modules.

```bash
ansible localhost -m ping
ansible localhost -m package -a "name=zsh state=present" -b
ansible localhost -m package -a "name=zsh state=absent" -b
```

The `-m` flag selects a module. The `-a` flag passes module arguments. The `-b` flag enables privilege escalation with `become`.

For `localhost`, Ansible uses an implicit host entry and a local connection. That makes it possible to validate the controller before any real inventory exists.
### `ansible-playbook`
The `ansible-playbook` command runs structured YAML playbooks. Playbooks provide the repeatability that ad-hoc commands lack. A playbook keeps the tasks, their order, and their variables in a file that can be reused safely.

A minimal playbook that pings localhost and prints the operating system family looks like this:

```yaml
---
- name: Simple play
  hosts: localhost
  connection: local
  tasks:
    - name: Ping localhost
      ping:
    - name: Show OS family
      debug:
        msg: "{{ ansible_os_family }}"
```

The playbook runs with:

```bash
ansible-playbook my.yml
```

By default, Ansible gathers facts before it runs the explicit tasks. That is why a playbook often shows an extra `Gathering Facts` step before the named tasks. Facts become variables such as `ansible_os_family`, `ansible_distribution`, and interface data.
### `ansible-doc`
`ansible-doc` is essential. It lists available modules and shows argument details and examples for each module.

```bash
ansible-doc -l
ansible-doc ping
ansible-doc package
ansible-doc user
```

Ansible ships with thousands of modules and plugins, so direct familiarity with every module is unrealistic. Effective use depends on recognising the likely module category and then checking the built-in documentation.
### `ansible-config`
`ansible-config` shows the active configuration and the default values.

```bash
ansible-config view
ansible-config list
ansible-config dump
ansible-config dump --only-changed
```

`dump --only-changed` is especially useful after editing `ansible.cfg` because it shows only the values that differ from the defaults.
### `ansible-inventory`
`ansible-inventory` shows how Ansible sees the host inventory after it merges all files and variables.

```bash
ansible-inventory --host localhost
ansible-inventory --list
ansible-inventory --list -y
```

JSON is the default output format for `--list`. YAML can be easier to read during manual review.
## Facts, variables, and Jinja
Ansible facts let the controller adapt to different systems without hard-coding every branch. Facts are available after the `setup` module runs, either explicitly or through playbook fact gathering.

```bash
ansible localhost -m setup -a "filter=ansible_os_family"
ansible localhost -m setup -a "filter=ansible_distribution"
```

The operating system family is more useful than the exact distribution in many tasks because it groups related platforms. RHEL and CentOS Stream share the `RedHat` family. Ubuntu belongs to the `Debian` family.

Variables become most useful when Ansible can choose the backend automatically but still needs help with names or paths. The `package` module can choose APT or DNF, but it cannot guess that one family uses `vim` and another uses `vim-enhanced`. Jinja expressions fill that gap.

A practical pattern is to define a variable such as `vim_editor` and set its value per group. The command or playbook can then refer to the variable rather than to the platform-specific package name.
## Managing `ansible.cfg`
Ansible reads configuration in a clear precedence order. The highest priority is the `ANSIBLE_CONFIG` environment variable. Below that comes `./ansible.cfg` in the current working directory, then `~/.ansible.cfg`, and finally `/etc/ansible/ansible.cfg`.

A quick check confirms the active file:

```bash
ansible --version
```

A practical user-level configuration might look like this:

```ini
[defaults]
inventory = ~/inventory
remote_user = tux
private_key_file = ~/.ssh/id_rsa

[privilege_escalation]
become = True
```

This file does three things:
- points Ansible at the chosen inventory
- sets a dedicated remote login account
- enables privilege escalation by default

A project-specific `./ansible.cfg` can override the user-level configuration. That is helpful when one project needs a different inventory, key, or default remote user.

An administrator can also enforce a configuration path through the shell environment:

```bash
export ANSIBLE_CONFIG=/etc/ansible/ansible.cfg
```

A read-only exported variable makes that setting harder to override in a shell session:

```bash
declare -xr ANSIBLE_CONFIG=/etc/ansible/ansible.cfg
```

This is useful in managed lab environments and in tightly controlled shared systems. It is less useful where individual projects need their own local configuration files.
## Building an inventory
The inventory is the list of hosts and groups that Ansible targets. INI format is easy to read and easy to maintain, though JSON and YAML are also valid.

A minimal flat inventory can begin with raw host entries:

```ini
192.168.33.11
192.168.33.12
192.168.33.13
```

Ansible automatically creates two useful groups:
- `all` for every host in the inventory
- `ungrouped` for hosts that do not belong to any explicit group

A more practical inventory assigns meaningful groups:

```ini
[rhel]
192.168.33.11

[stream]
192.168.33.12

[ubuntu]
192.168.33.13

[redhat:children]
rhel
stream
```

This layout supports several targeting strategies:
- `rhel` for the RHEL node
- `stream` for the CentOS Stream node
- `ubuntu` for the Ubuntu node
- `redhat` for both Red Hat family systems
- `all` for the full estate

Host and group membership can be inspected directly:

```bash
ansible-inventory --host 192.168.33.11
ansible-inventory --list -y
```

Patterns such as `www[001:006]` are also valid when hosts follow a strict naming scheme. That can simplify large inventories.
## Host variables and group variables
Inventory variables allow host-specific and group-specific customisation without cluttering the main inventory file. Ansible loads these from `host_vars` and `group_vars` relative to the inventory or playbook directory.

Create the variable directories first:

```bash
mkdir -p host_vars group_vars
```

A host variable can override the connection method for a specific host. In a lab, the RHEL controller can be managed locally even when it appears in the inventory by IP address.

```bash
cat > host_vars/192.168.33.11.yml <<'EOF'
ansible_connection: local
EOF
```

This lets the controller address `192.168.33.11` without SSH while still treating it as a real inventory member.

Group variables solve platform-specific naming issues. The Vim example can be expressed cleanly as YAML.

```bash
cat > group_vars/redhat.yml <<'EOF'
vim_editor: vim-enhanced
EOF

cat > group_vars/ubuntu.yml <<'EOF'
vim_editor: vim
EOF
```

Ansible merges the correct variable based on group membership. The resulting command stays simple:

```bash
ansible all -m package -a "name={{ vim_editor }} state=present" -b
```

The same pattern works for service names, configuration paths, repository packages, firewall tools, and other cross-platform differences. For example, a time service variable can capture whether a system expects one service name or another. A configuration path variable can point to `/etc/chrony.conf` on one family and a different path on another, though chrony commonly uses `/etc/chrony.conf` on both modern Ubuntu and RHEL based systems.
## Dynamic inventory and host discovery
Static inventories are easiest to start with, but automation can also discover hosts dynamically. One simple lab method scans for SSH listeners and extracts the IP addresses from the scan output.

```bash
sudo dnf install -y nmap

sudo nmap -Pn -p 22 -oG - 192.168.33.0/24 | awk '/22\/open/ {print $2}'
```

This command does the following:
- skips the ICMP host discovery phase with `-Pn`
- probes only TCP port 22
- prints greppable output with `-oG -`
- extracts the second field from lines where port 22 is open

The output can be redirected into a file and used as the basis of a generated inventory. In production, dynamic inventory usually integrates with cloud APIs, virtualisation platforms, or CMDB tooling rather than with a port scan, but the lab example shows the principle clearly.
## Preparing SSH access
The controller needs passwordless SSH access to the managed hosts. Vagrant already creates SSH keys for its own access model, so the lab can reuse those keys during the first stage of setup.

On the host operating system, Vagrant can show the SSH settings for each guest:

```bash
vagrant ssh-config stream
vagrant ssh-config ubuntu
```

The `IdentityFile` values point to the private keys that Vagrant uses. A controller running inside the RHEL guest needs copies of the keys for the Stream and Ubuntu guests. A Vagrant SCP plugin makes this convenient.

```bash
vagrant plugin install vagrant-scp
```

A typical workflow copies the Vagrant-generated keys into the controller guest under descriptive names such as `stream.key` and `ubuntu.key`. Once the keys are present, the controller can verify direct SSH access:

```bash
ssh -i ~/stream.key vagrant@192.168.33.12
ssh -i ~/ubuntu.key vagrant@192.168.33.13
```

This first access also records the remote host keys in `known_hosts`, which prevents later prompts during Ansible runs.
## Creating a dedicated Ansible account
Using a dedicated remote account is cleaner than using Vagrant for long-term administration. The account can be created on the managed nodes with ad-hoc commands, using the temporary Vagrant access only for bootstrapping.

Create the account on the Red Hat family host:

```bash
ansible stream -u vagrant --private-key ~/stream.key \
  -m user -a "name=tux state=present" -b
```

Create the same account on Ubuntu:

```bash
ansible ubuntu -u vagrant --private-key ~/ubuntu.key \
  -m user -a "name=tux state=present" -b
```

The new account needs passwordless sudo access if the controller is expected to install packages, manage services, and modify system files without interactive prompts. A standard sudoers drop-in works well.

```bash
printf 'tux ALL=(ALL) NOPASSWD: ALL\n' > tux
visudo -cf tux
```

After validation, copy the file to each managed host:

```bash
ansible stream -u vagrant --private-key ~/stream.key \
  -m copy -a "src=tux dest=/etc/sudoers.d/tux mode=0440" -b

ansible ubuntu -u vagrant --private-key ~/ubuntu.key \
  -m copy -a "src=tux dest=/etc/sudoers.d/tux mode=0440" -b
```

This stage finishes the account bootstrap. The controller now needs its own SSH keypair for the permanent `tux` login.
## Deploying the controller's SSH key
Generate a controller-side keypair if one does not already exist:

```bash
ssh-keygen -t rsa -b 4096 -N '' -f ~/.ssh/id_rsa
```

The controller can then push the public key to the `tux` account with the `authorized_key` module. The Jinja `lookup("file", ...)` call reads the local public key file.

```bash
ansible stream -u vagrant --private-key ~/stream.key \
  -m authorized_key \
  -a "user=tux state=present key='{{ lookup(\"file\", \"~/.ssh/id_rsa.pub\") }}'" \
  -b

ansible ubuntu -u vagrant --private-key ~/ubuntu.key \
  -m authorized_key \
  -a "user=tux state=present key='{{ lookup(\"file\", \"~/.ssh/id_rsa.pub\") }}'" \
  -b
```

Once the key is in place, the controller no longer needs the temporary per-host Vagrant keys for routine Ansible work. The permanent access path becomes:
- remote user `tux`
- controller private key `~/.ssh/id_rsa`
- privilege escalation through sudo

That model aligns cleanly with the earlier `ansible.cfg` example.
## Verifying the completed environment
After the inventory, remote user, and private key are configured, the controller can test the entire estate with a single command.

```bash
ansible all -m ping
```

A successful result means that:
- the inventory resolves the correct hosts
- SSH authentication works
- Python is available on the managed nodes
- `become` succeeds where needed

At that point, ad-hoc commands become genuinely useful. The `package` module can install a package shared across all distributions without caring about the package manager backend.

```bash
ansible all -m package -a "name=tree state=present" -b
```

Running the same command twice shows Ansible's idempotent behaviour. The first run changes hosts that need the package. The second run reports success without changes because the estate already matches the requested state.

The same principle applies to removal:

```bash
ansible all -m package -a "name=tree state=absent" -b
```

When package names differ, the variable layer restores portability:

```bash
ansible all -m package -a "name={{ vim_editor }} state=present" -b
```

This is the practical centre of beginner-level Ansible administration. Modules describe the task. Inventory defines the target. Variables absorb platform differences. SSH and privilege escalation make the execution model consistent.
## A sensible beginner workflow
A useful working method for new Ansible administrators follows a predictable sequence.
### 1. Build the lab
Use Vagrant and VirtualBox to produce a small mixed environment quickly. Confirm that each guest has the expected IP address and that the private network works.
### 2. Register RHEL if required
RHEL needs repository access before it can serve as a controller or managed node. A valid subscription removes package installation friction later.
### 3. Install Ansible only on the controller
The controller holds the Ansible package. Managed nodes need SSH and Python, not the full Ansible installation.
### 4. Validate the controller locally
Use `ansible localhost -m ping`, `ansible-doc`, `ansible-config`, and `ansible --version` before attempting remote work.
### 5. Create a clean configuration
Set the inventory path, remote user, private key file, and privilege escalation defaults in `ansible.cfg`. Keep the configuration in `~/.ansible.cfg` or in the project directory.
### 6. Build the inventory
Start with explicit IP addresses and small groups. Add child groups where they simplify targeting.
### 7. Introduce host and group variables
Use variables for real differences, especially package names, service names, file paths, and connection settings.
### 8. Bootstrap SSH and the remote user
Use temporary access only long enough to create the permanent Ansible user, install sudoers policy, and deploy the controller's public key.
### 9. Favour modules over shell commands
Modules know how to express state. They reduce error handling, improve readability, and remain idempotent.
### 10. Move repeated commands into playbooks
Ad-hoc commands are excellent for learning and for one-off changes. Repeated operational tasks belong in playbooks because the file becomes the repeatable source of truth.
## Practical limits of ad-hoc commands
Ad-hoc commands are intentionally narrow. They suit quick checks such as:
- connectivity
- package presence
- service state
- user creation
- file copy
- one-off fact gathering

They become awkward when the workflow needs ordering, conditional execution, loops, templates, handlers, roles, or version-controlled history. Playbooks take over at that point. A new administrator should still spend time with ad-hoc commands because they expose the building blocks clearly. Every playbook task still uses the same module logic underneath.
## What a beginner should remember
Ansible becomes easier once a few ideas are fixed firmly.

- The controller is the only machine that needs the Ansible package.
- SSH is the transport for most Linux administration.
- Python on the managed node is required for the common module model.
- `ansible` runs ad-hoc commands.
- `ansible-playbook` runs YAML playbooks.
- `ansible-doc` explains modules and their arguments.
- `ansible-config` shows configuration and precedence.
- `ansible-inventory` shows the processed inventory.
- Facts describe the remote system.
- Jinja expressions insert variables into commands and playbooks.
- Modules describe state rather than shell procedure.
- Group variables and host variables are the cleanest place to absorb platform differences.

A small mixed lab is enough to practise all of these fundamentals. Once the controller can reach the estate reliably, the next logical step is to replace repetitive ad-hoc work with structured playbooks, roles, and templated configuration.
## Working locally before remote execution
The implicit `localhost` entry is more useful than it first appears. It provides a safe place to prove that the controller can run modules, gather facts, and resolve configuration before any remote authentication is involved. That reduces the number of moving parts during early troubleshooting.

A sensible first sequence on the controller is:

```bash
ansible localhost -m ping
ansible localhost -m setup -a "filter=ansible_os_family"
ansible-doc package
ansible-config dump --only-changed
```

If those commands succeed, the controller is already capable of reading its configuration, loading modules, calling Python, and printing facts. Failure at this stage usually points to a bad installation, a broken Python environment, or an unexpected configuration file rather than to an SSH issue.

Local execution also demonstrates an important Ansible habit. The administrator asks for a state, not for a procedure. For package management, that difference matters. A shell workflow says "run `apt install` or `dnf install`". Ansible says "make sure this package is present". The module handles the implementation detail.
## Reading module behaviour properly
Each module has its own expected arguments, defaults, and return values. The built-in documentation is therefore part of normal administration rather than an optional reference.

The `ping` module is a good example because its name can mislead. It does not send an ICMP echo request like the system `ping` command. It checks whether Ansible can run a simple Python-based module on the target and receive a normal response. That makes it a connectivity test, an authentication test, and a Python test at the same time.

The `package` module demonstrates Ansible's portability layer. It abstracts the package manager so that one task can work across APT, YUM, and DNF. It does not remove the need for variables when package names differ, but it removes the need to write separate tasks for different package tools.

The `user` module is equally important in early setup. It can create a user, assign groups, set shells, define system users, and manage home directory behaviour. The basic bootstrap example uses only `name=tux state=present`, but the module supports much richer account policy.

The `copy` module handles straightforward file delivery from the controller to the managed node. It is ideal for sudoers drop-ins, static configuration snippets, and small policy files. The `authorized_key` module is then the clean way to manage SSH public keys in a user's `authorized_keys` file.

This pattern is worth remembering because it reflects how a real Ansible build-out often starts:
- `ping` to verify access
- `user` to create the administration account
- `copy` to apply sudo policy
- `authorized_key` to install permanent SSH trust
- `package` to enforce baseline software
## Playbooks and YAML discipline
A playbook is a YAML file that contains one or more plays. Each play targets hosts and runs tasks. YAML uses indentation to express structure, so spacing is not cosmetic. A misplaced indent changes the meaning of the file or makes it invalid.

A minimal playbook usually contains these elements:
- a list of plays marked by `-`
- a `name` for readability
- a `hosts` target
- optional connection or privilege settings
- a `tasks` list
- one or more module calls under each task

That structure makes playbooks more reliable than repeated ad-hoc commands. The file preserves the order of operations. It also preserves intent. A reader can inspect the playbook later and see exactly what state the administrator wanted.

A slightly fuller example shows facts, a package install, and variable use in one place:

```yaml
---
- name: Baseline editor install
  hosts: all
  become: true
  tasks:
    - name: Show the platform family
      debug:
        msg: "{{ ansible_os_family }}"

    - name: Ensure the preferred editor is installed
      package:
        name: "{{ vim_editor }}"
        state: present
```

This playbook illustrates several core behaviours at once. The task names read clearly. `become: true` applies privilege escalation to the play. `debug` exposes a fact. `package` remains portable because the inventory supplies the correct package name per group.

The value of playbooks increases with every repeated task. A one-line package check might remain ad-hoc. A baseline build, security hardening step, or common configuration policy belongs in a playbook.
## Configuration precedence in practice
Configuration precedence matters because Ansible often appears to "ignore" a setting when it is actually reading a different file. The precedence model solves that mystery.

An administrator can test precedence quickly with empty files. A blank `~/.ansible.cfg` is enough to override `/etc/ansible/ansible.cfg`. A blank `./ansible.cfg` in the current working directory is enough to override `~/.ansible.cfg`. The file does not need meaningful content to prove which path Ansible prefers.

A practical test sequence looks like this:

```bash
ansible --version
touch ~/.ansible.cfg
ansible --version
mkdir -p ~/ansible/test
cd ~/ansible/test
ansible --version
touch ansible.cfg
ansible --version
```

Each `ansible --version` call shows the configuration file currently in use. That makes it easy to confirm whether Ansible is reading the expected path.

The same technique helps when a project behaves differently from another project on the same controller. If one directory contains its own `ansible.cfg`, that file may override the user's global defaults entirely. The correct response is not guesswork. It is to check `ansible --version` and then inspect the active file with `ansible-config view`.

A project-specific file is often the right choice. It allows one repository to carry its own inventory path, SSH key, and output settings without affecting every other Ansible project. The user-level file remains useful for defaults that make sense everywhere, such as a preferred editor or general output behaviour.
## Inventory inspection and grouping strategy
Inventories become easier to manage when the grouping model mirrors the way systems are administered. A poor inventory groups hosts by arbitrary labels. A good inventory groups them by operational meaning.

In the small lab, the following group types make sense:
- platform groups such as `rhel`, `stream`, and `ubuntu`
- family groups such as `redhat`
- global groups such as `all`

That structure supports both narrow targeting and broad targeting. An administrator can address a single platform, an operating system family, or the full estate without rewriting commands.

The group hierarchy also influences variable inheritance. A variable placed in `group_vars/redhat.yml` applies to both `rhel` and `stream` because both groups are children of `redhat`. That makes family-wide defaults easy to express. A narrower variable in `group_vars/rhel.yml` can still override the family value if RHEL needs a special case later.

`ansible-inventory` is therefore more than a display tool. It is a diagnostic tool for inheritance and merge behaviour. Two commands are especially useful:

```bash
ansible-inventory --host 192.168.33.11
ansible-inventory --list -y
```

The `--host` form shows the merged variables for a single host. The `--list -y` form shows the overall inventory tree and group structure in readable YAML. When a variable appears to be missing, these commands usually show whether it never loaded, loaded under the wrong name, or was overridden by a higher-precedence value.
## Using variables for real platform differences
Variables matter most when the controller needs to keep a single task portable across systems that differ in a meaningful way. Package names are the simplest example, but they are not the only one.

Common variable categories include:
- package names
- service names
- configuration file paths
- repository enablement packages
- firewall tools
- file ownership and group policy

A time synchronisation example illustrates the pattern well. One platform might use a service called `chronyd`. Another might use a different name or package dependency. Rather than branching repeatedly in each task, the inventory can define variables such as `time_pkg`, `time_service`, and `time_conf`. The playbook or ad-hoc command then refers only to those variables.

The result is not just shorter code. It is cleaner operational intent. The task says "manage the time service" rather than "run this platform-specific command". Inventory becomes the place where the platform difference lives.

This separation also makes audits easier. A reviewer can inspect the inventory variables and immediately see what differs between platforms, instead of scanning many tasks for repeated conditional logic.
## Managing privilege escalation sensibly
Most real administration requires elevated privileges. Package installation, service control, file placement in `/etc`, and user creation all need root-level access. Ansible handles this through `become`.

At the ad-hoc command line, `-b` is enough in the common case:

```bash
ansible all -m package -a "name=tree state=present" -b
```

In playbooks, `become: true` can be set at the play level or task level. A default `become = True` in `ansible.cfg` can also reduce repetition, though that convenience should be used with intent. Some administrators prefer explicit privilege escalation in playbooks so that the file makes the privilege boundary obvious.

A dedicated remote account such as `tux` is usually preferable to logging in directly as `root`. It creates an auditable boundary between connection identity and privilege. The account authenticates over SSH, then escalates through sudo only for tasks that need it.

That is why the bootstrap sequence matters so much. It establishes:
- a named administration account
- explicit sudo policy
- a controller-controlled SSH key
- predictable privilege escalation

Once those pieces are in place, the rest of the Ansible workflow becomes far more stable.
## Idempotence and repeatability
One of Ansible's defining behaviours is idempotence. If the requested state already exists, Ansible does not apply the change again. It reports success without change.

Package tasks show this behaviour clearly. The first run installs the missing package. The second run confirms that the package is already present. The same logic applies to user accounts, copied files, and authorised keys.

This changes the way the administrator thinks about operations. The goal is not to write a script that "does the install". The goal is to declare the end state safely enough that repeating the declaration causes no harm.

That is also why modules are better than raw shell commands for most configuration work. A shell command may or may not be safe to rerun. A well-designed module is usually built around repeatable state checks.
## Moving from ad-hoc work to structured automation
Ad-hoc commands are the right starting point because they expose the essentials without hiding anything behind a larger framework. A beginner sees the host pattern, the module name, the arguments, and the privilege flag directly.

Over time, repeated ad-hoc commands should migrate into playbooks. The trigger is easy to recognise. If the same command starts to appear in shell history repeatedly, it probably belongs in a playbook. The same applies when a task depends on ordering or when success requires multiple coordinated steps.

A common progression looks like this:
- verify connectivity with ad-hoc `ping`
- create users and install packages with ad-hoc commands
- move the repeated baseline into a playbook
- add variables and templates
- break larger playbooks into roles

This progression matters because it keeps early learning concrete while still moving towards maintainable automation. It also respects how most administrators actually learn Ansible. They start with one command that saves time. They then generalise that command into a repeatable file.