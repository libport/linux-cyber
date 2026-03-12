# Ad Hoc commands
Ad hoc commands apply a single task to managed hosts without a playbook. They suit one-off setup, quick validation after a change, and short discovery tasks. A typical command specifies the Ansible executable, a host pattern, a module, and module arguments.

```bash
ansible all -m user -a "name=lisa"
```

Run ad hoc commands as the Ansible user from a project directory that contains `inventory` and `ansible.cfg`. Ansible compares the requested state with the current state and applies only the changes needed to reach the target state. In a user-management task, `SUCCESS` means the host already matched the request. `CHANGED` means Ansible modified the host.

Many Ansible modules are idempotent, but not every ad hoc command is. Resource modules such as `user`, `copy`, `yum`, and `service` usually converge on the same state across repeated runs. Generic execution modules such as `command`, `shell`, and `raw` do not guarantee that behaviour and can report changes even when they only inspected state.
## Core modules
Ansible depends on modules:
- `command` runs a command directly and does not process shell features such as pipes or redirects
- `shell` runs through a shell and supports pipes, redirects, and other shell syntax
- `raw` sends a command over SSH without requiring Python on the managed host, which helps when bootstrapping minimal systems
- `copy` writes files or simple text content to a destination path
- `yum` manages packages on RHEL-family systems
- `service` manages service state and boot-time enablement
- `ping` tests whether a host is manageable through Ansible, not whether it answers ICMP echo

Use the most specific module available. That choice improves readability, supports idempotent behaviour where possible, and makes results easier to audit. Package installation belongs in `yum` rather than `shell`, and a service that must survive reboot needs both `state=started` and `enabled=yes`.
## Module discovery and documentation
`ansible-doc` is the main on-system reference for modules. It supports three common workflows.

- `ansible-doc -l` lists available modules
- `ansible-doc modulename` shows full documentation
- `ansible-doc -s modulename` shows a compact, playbook-oriented synopsis

The full entry usually includes the module purpose, maintainer, options, related modules, examples, and return values. The compact synopsis is often more useful during authoring because it shows required parameters and expected structure without extra detail. Searching the module list with `grep` narrows the field quickly when several modules could fit the same task.

Online documentation complements `ansible-doc`. It mirrors the core reference material and helps locate modules by task area when the exact module name is unknown.
## Shell scripts for repeatable ad hoc work
Short shell scripts can package repeatable ad hoc tasks into a single runnable file. This is useful when the same setup must be applied again to fresh hosts. A script should be plain text, begin with a shebang, and contain explicit Ansible commands.

```bash
#!/bin/bash
ansible all -m yum -a "name=httpd state=latest"
ansible all -m service -a "name=httpd state=started enabled=yes"
```

Make the script executable with `chmod +x setup.sh`. Run it from the current directory with `./setup.sh`, or place it in a directory on `PATH`. The `.sh` suffix is optional, but it helps identify the file as a script.

Shell scripts are practical for small, repeatable operations. Playbooks remain the better tool for larger automation because they organise state clearly, scale more cleanly, and express intent more precisely.