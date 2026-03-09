# Part I: Introduction to Ansible
## Understanding Configuration Management
## Ansible project foundation
Ansible relies on two core project files: an inventory source and an `ansible.cfg` file. The inventory defines managed hosts and groups. A project can store a single static inventory file or point to a directory of inventory sources. Static files can sit beside dynamic sources, provided executable scripts have execute permission. Modern Ansible prefers inventory plugins over legacy scripts, but external scripts still need to answer the `--list` and `--host <hostname>` arguments when used as inventory sources.

The project configuration file stores default behaviour for routine administration. A typical configuration sets the remote user, the default inventory path, SSH host key checking, and privilege escalation. The main escalation settings are `become`, `become_method`, `become_user`, and `become_ask_pass`. These values establish how Ansible reaches a target host and how it gains administrative access when required. More specific settings override broader ones, so playbook settings override `ansible.cfg`.

```ini
[defaults]
remote_user = ansible
inventory = inventory
host_key_checking = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```

This layout supports predictable project behaviour. It also removes the need to pass the inventory file on every command line. A project can then confirm the active inventory with `ansible-inventory --list` or inspect group structure with `ansible-inventory --graph`.
## Ad hoc commands
Ad hoc commands let administrators run one-off tasks against managed hosts without writing a playbook. They suit quick setup work, validation after a change, and discovery checks. They do not replace playbooks for larger repeatable workflows, but they provide a fast operational tool.

An ad hoc command follows a simple pattern:

```bash
ansible <host-pattern> -m <module> -a "<arguments>"
```

In `ansible all -m user -a "name=lisa"`, `all` selects the target hosts, `user` selects the module, and the argument string defines the desired state. Ansible then compares that desired state with the host's current state and applies change only when needed. That idempotent design helps modules return `SUCCESS` when nothing changes and `CHANGED` when a host required modification.

Modules that express state clearly, such as `user`, `copy`, `package`, and `service`, usually behave predictably. Arbitrary command execution through `command`, `shell`, or `raw` can still run successfully without proving that the target now matches a declared state.
## Core modules
The most useful ad hoc work relies on a small set of modules:
- `command` runs a command directly on the target without a shell. It is safer and more predictable than shell execution, but shell metacharacters such as `|`, `>`, `<`, and `&` do not work.
- `shell` runs a command through the remote shell. It supports pipes, redirects, and shell expansion, but it increases quoting risk and reduces predictability.
- `raw` sends a low-level command over the remote connection without requiring Python on the target. It is useful for bootstrapping new hosts and for devices that do not provide a normal Python environment.
- `copy` transfers a file or writes literal content to a remote path. It works well for small configuration files, banners, or message-of-the-day content.
- `package` manages software through the target's native package manager and suits mixed Linux estates. On Red Hat-family systems, `dnf` provides a more specific package-management interface.
- `service` manages service state across several init systems. On hosts that use systemd, `systemd_service` offers more precise control when systemd-specific features matter.
- `ping` checks manageability rather than network reachability. It verifies that Ansible can connect and run a basic module on the host. It is not ICMP ping.

`command` fails for piped package queries such as `rpm -qa | grep httpd` because no shell interprets the pipe. `shell` handles that syntax, while `package` or `dnf` usually provides a cleaner and more reliable way to inspect or install packages. Service management follows the same principle. `state=started` starts a service now, while `enabled=true` ensures it also starts after reboot.
## Module discovery and documentation
`ansible-doc -l` lists installed modules, and command-line filtering helps narrow the result set. Searching for a platform or feature, such as `ansible-doc -l | grep vmware`, quickly exposes relevant modules.

Detailed module help comes from `ansible-doc <module>`. The output usually includes the module name, a short description, maintainership, parameters, related modules, examples, and return values. That structure helps administrators judge whether a module fits the task, which arguments are mandatory, and what output to expect.

For playbook work, `ansible-doc -s <module>` often provides the most efficient view. It shows a concise, playbook-style parameter structure with short descriptions. That format helps when a task needs the right arguments quickly without reading the full manual page.

Running the script directly with `--list` exposes the JSON inventory, and piping that output through a formatter makes group membership easier to inspect. When a project uses several inventory files, placing them in one directory lets Ansible compose a broader inventory without repeating long command lines.
## Practical operating pattern
A sound operating pattern starts with inventory and configuration, verifies reachability with `ping`, then uses the most specific stateful module available. Arbitrary shell commands remain useful for edge cases, but they should not be the first choice when a purpose-built module exists.

This approach keeps automation readable, reduces avoidable change, and improves confidence in results. Ansible works best when project settings are explicit, inventory sources are organised, and module choice reflects the target state rather than the operator's preferred shell syntax.