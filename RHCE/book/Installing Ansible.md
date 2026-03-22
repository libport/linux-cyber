# Installing Ansible
## Ansible lab design
An Ansible environment uses one control node and at least two managed nodes. The control node runs Ansible. Managed nodes do not need an agent. Linux hosts usually need SSH access and a compatible Python interpreter. A small lab works well with three virtual machines, fixed hostnames, fixed IP addresses, and local name resolution through DNS or `/etc/hosts`. Separating the control node from the managed hosts reduces accidental changes during early practice. In production, teams often automate the control node as well.
## Controller requirements
The control node needs Python, an SSH client, and access to a supported package source. Current Ansible packaging separates the minimal runtime, ansible-core, from the larger community bundle, ansible. The chosen package should match the lab goal. A certification lab may need a fixed version, while a general environment should use a maintained release on a supported platform.

RHEL remains a valid base for a lab. Red Hat still provides no-cost developer access for individual use. CentOS Linux 8 does not remain a sound choice because it reached end of life in December 2021 and no longer receives updates. A current community alternative is CentOS Stream or another supported Enterprise Linux derivative.

On RHEL-like systems, dnf is the standard package manager. A maintained installation usually starts with distribution packages such as ansible-core, or with a user-scoped Python installation through pip or pipx. Changing the system-wide python command is unnecessary in most cases. The safer approach is to call Python explicitly, such as python3 -m pip.
## Managed host preparation
Managed Linux hosts need SSH access, a compatible Python interpreter, and a user account that allows interactive shell access. They do not need the Ansible package itself. Network devices and Windows hosts follow different connection models, so a Linux-only lab can ignore those details. Basic connectivity checks should confirm name resolution, SSH reachability, and firewall rules that allow SSH.
## Dedicated automation account
A dedicated ansible account keeps automation predictable. That account needs two capabilities on each managed node:
- key-based SSH login from the control node
- privilege escalation for tasks that need root access

SSH keys remove repeated password prompts and make batch automation practical. A lab can use a key without a passphrase for speed. A production environment usually protects the private key with a passphrase or uses a managed secret store.

Privilege escalation normally uses `sudo`. Drop-in files under `/etc/sudoers.d` are cleaner than editing `/etc/sudoers` directly. Password-less `sudo` is convenient in a lab, but a production system should apply tighter controls and restrict escalation to the commands or roles that need it.
## Minimal setup sequence
- Build three virtual machines with stable hostnames and addresses.
- Create the ansible user on the control node and managed nodes.
- Confirm SSH service availability and Python compatibility on each managed host.
- Install ansible-core or ansible on the control node from a supported source.
- Generate an SSH key pair on the control node and copy the public key to each managed host.
- Configure sudo access for the ansible user on the managed hosts.
- Test the setup with direct SSH access and a simple Ansible command.
## Outcome
A straightforward Ansible lab keeps the controller simple, keeps managed hosts agentless, and relies on SSH, Python compatibility, and controlled privilege escalation. The core design still holds, but the platform and packaging choices must reflect current supported releases.