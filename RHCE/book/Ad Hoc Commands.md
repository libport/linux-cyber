# Ad Hoc Commands
Red Hat Enterprise Linux 10 provides `ansible-core` 2.16. A RHEL 10 control node can manage RHEL 9 and RHEL 10 hosts without installing Ansible on those managed hosts.
## Purpose and syntax
An ad hoc command uses the `ansible` command-line tool to run one task across one host or a host pattern. It suits quick checks, one-off changes, service recovery, and information gathering. It does not preserve the task as reusable automation, so a playbook remains the better choice for repeated or multi-step work.

The general form is:

```shell
ansible PATTERN -i INVENTORY -m MODULE -a 'ARGUMENTS'
```

The operator selects hosts through an inventory and supplies valid connection credentials. An `ansible.cfg` file can provide defaults, but Ansible does not require a particular project directory or login name. Tasks that need administrative rights use `-b` for privilege escalation and, when required, `-K` to request the escalation password.

Host patterns can select one host, a group, several groups, or the complete inventory. Operators should confirm the pattern before a change and can add `--limit` to narrow an existing selection. The `-f` option controls parallel forks when the default concurrency does not suit the environment.

These commands verify access, create an account, inspect it, and remove it:

```shell
ansible all -i inventory -m ansible.builtin.ping
ansible all -i inventory -b -m ansible.builtin.user -a 'name=lisa state=present'
ansible all -i inventory -m ansible.builtin.command -a 'id lisa'
ansible all -i inventory -b -m ansible.builtin.user -a 'name=lisa state=absent remove=true'
```

State-oriented modules usually report `CHANGED` only when they alter a host. A second run often reports success without a change. This behaviour depends on the module and its arguments. Ansible commands are not universally idempotent, especially when they invoke arbitrary commands.

Ansible reports a result for each selected host. `UNREACHABLE` indicates a connection or authentication failure. `FAILED` indicates that Ansible reached the host but could not complete the task. Partial success can therefore leave hosts in different states. Once an operator corrects the cause, a state-oriented module can usually run again and change only the outstanding hosts.

`--check` requests a dry run from modules that support check mode:

```shell
ansible webservers -i inventory -b --check -m ansible.builtin.dnf -a 'name=httpd state=present'
```

Check mode estimates changes and cannot replace validation on a suitable test system. Modules that lack check-mode support may skip the operation or provide limited information.
## Choosing modules
Ansible distributes modules and other plugins in collections. The `ansible.builtin` collection accompanies `ansible-core`, while installed collections add support for other platforms and products. Fully qualified collection names identify the intended module and avoid name collisions.

| Module | Appropriate use |
|---|---|
| `ansible.builtin.command` | Runs a program without a shell |
| `ansible.builtin.shell` | Runs a command through `/bin/sh` |
| `ansible.builtin.raw` | Sends a command through the connection without the module subsystem |
| `ansible.builtin.copy` | Transfers a file or writes fixed content |
| `ansible.builtin.dnf` | Manages RHEL 10 packages |
| `ansible.builtin.systemd_service` | Manages systemd units |
| `ansible.builtin.ping` | Tests login and a usable remote Python interpreter |

`command` is the default module for the `ansible` utility. It passes arguments directly to a program, so pipes, redirection, wildcards, and other shell syntax do not operate. This check requires no shell:

```shell
ansible webservers -i inventory -m ansible.builtin.command -a 'rpm -q httpd'
```

`shell` supports pipes and redirection because it invokes a remote shell. Shell parsing also increases injection risk, particularly when data comes from variables or users. `command` or a purpose-built module should take priority whenever either can perform the task safely.

`raw` bypasses the module subsystem and does not require Python on the managed host. It can bootstrap an unusually minimal host or address a device that lacks Python. It offers limited change reporting and no check-mode support, so normal administration should use a dedicated module.

The generic `ansible.builtin.package` and `ansible.builtin.service` modules can support mixed operating systems. On RHEL 10, `ansible.builtin.dnf` and `ansible.builtin.systemd_service` expose the platform's native capabilities more directly. Purpose-built modules also describe desired state more clearly than equivalent shell commands.
## RHEL 10 administration
RHEL 10 uses DNF for package management and systemd for services. The following commands install Apache HTTP Server, start it, and enable it at boot:

```shell
ansible webservers -i inventory -b -m ansible.builtin.dnf -a 'name=httpd state=present'
ansible webservers -i inventory -b -m ansible.builtin.systemd_service -a 'name=httpd state=started enabled=true'
```

`state=present` installs an available package without forcing every run to upgrade it. Administrators should use `state=latest` only when the task intentionally applies the newest available version.

The `copy` module can set a short login banner and its file attributes:

```shell
ansible all -i inventory -b -m ansible.builtin.copy -a '{"content":"Authorised systems only\n","dest":"/etc/motd","owner":"root","group":"root","mode":"0644"}'
```

The `content` argument replaces the destination with fixed text. The `src` argument instead transfers a file from the control node. Templates, managed blocks, or line-oriented modules suit content that requires variables or selective editing.

`ansible.builtin.ping` returns `pong` after Ansible logs in and runs Python. It does not send an ICMP echo request. Windows and network devices require their platform-specific ping modules.
## Finding documentation
Installed documentation reflects the available collections and versions:

```shell
ansible-doc -t module -l
ansible-doc ansible.builtin.dnf
ansible-doc -s ansible.builtin.systemd_service
ansible-galaxy collection list
```

Module documentation describes parameters, requirements, examples, return values, and check-mode support. Administrators should extend Ansible through a custom collection or contribute upstream instead of editing installed module files, which package updates can replace.
## Sequencing commands
A Bash script can sequence a small set of ad hoc commands:

```bash
#!/usr/bin/env bash
set -euo pipefail

inventory=${1:?Provide an inventory path}

ansible webservers -i "$inventory" -b -m ansible.builtin.dnf \
  -a 'name=httpd state=present'
ansible webservers -i "$inventory" -b -m ansible.builtin.systemd_service \
  -a 'name=httpd state=started enabled=true'
```

After `chmod +x configure-httpd`, the operator can run `./configure-httpd inventory`. A playbook provides clearer review, reuse, variables, handlers, and error control when automation grows beyond a short operational sequence.

The shebang selects Bash, and the executable bit permits direct invocation. A `.sh` extension remains optional. `set -euo pipefail` stops the wrapper after common command, variable, or pipeline errors, but operators must still review Ansible's per-host results.